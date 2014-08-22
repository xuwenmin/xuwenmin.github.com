## [转载]浅说qwrap的Helper机制（一）

qwrap的核心机制，是以Helper->Wrap->Retouch->Apps为主线的，其中最最基础的就是Helper->Wrap->Retouch机制。

要理解这个机制，只要理解这个机制本质上所做的事情 —— 函数变换。

函数变换，通俗来说，可以理解为将一个函数变为另一个函数，执行函数变换的函数是一个高阶的函数，或者说函数变换算子。

函数变换可以是很灵活的，能做很多事情。理论上讲，任何参数中包含function，而返回值为function的函数，就是一个某种变换的算子。

---

<!--more-->

例如：

```js
function comb(op, f1, f2){
    return function(){
        return op(f1.apply(this, arguments), f2.apply(this, arguments);
    }
};
 
function a(x){
    return x;
};
function b(x){
    return x*x;
};
 
var square_x_add_x = comb(function(x,y){return x+y}, a, b);
alert(square_x_add_x(10));  //110;
```

上面这个例子里，算子comb对f1、f2两个函数进行一个组合变换，变换的结果为对用相同参数依次调用这两个函数得到的返回值进行op操作。

在QWrap的核心中，实际上使用到的算子只有非常有限的三种： mul、rwrap和methodize

mul实际上做的事情只是将一个函数fn的第一个参数变换为一个list，具体是，当一个函数被mul变换之后，可以传递给这个函数一个list，mul所做的事情是对这个list的第一个或每一个元素执行对应的fn操作，然后将结果或结果list返回。

```js
mul: function(func, recursive, getFirst){
        function flat_arr(array, findFirst){
            var ret = [];
            for(var i = 0, len = array.length; i < len; i++){
                var o = array[i];
                if(o instanceof Array){
                    ret = ret.concat(flat_arr(o,findFirst));
                }else{
                    ret = ret.concat([o]);
                }
                if(findFirst && ret.length) return ret;
            }
            return ret;
        }
       
        return function(){
            var list = arguments[0];
            if(!(list instanceof Array)){
                return func.apply(this, arguments); //不是数组，直接调用后返回
            }
            if(recursive){//如果是一个递归的数组，先解递归，然后继续
                list = flat_arr(list, getFirst);
            }
            var ret = [];
            var eachArgs = [].slice.call(arguments,0);
            for(var i = 0, len = list.length; i < len; i++){
                eachArgs[0]=list[i];
                var r = func.apply(this, eachArgs);
                if(getFirst) return r;
                ret.push(r);    
            }
            return ret;       
        }
    },
```

这个函数有几个细节，体现在额外参数上，但为了便于理解，我们先忽略细节，看最最基本的功能：

```js
function add(x,y){
    return x+y;
}
 
var addlist = mul(add);
 
var result = addlist([1,2,3,4], 1);
 
alert(result); //2,3,4,5
```

可以看到，一个函数经过mul变换之后，本来只能接受单个参数的，变成可以接受一个列表。

注意到后面还有两个参数，实际上一个参数是可以将数组扁平化，即[[1,2],[3,4]]等同于[1,2,3,4]，另一个参数是默认只返回列表的第一个成员的调用结果（用来实现jquery的get first/set all）

第二个算子是一个return wrap，这个概念可能稍微有点费解，什么是wrap呢？ wrap其实就是对象外面包裹着一层”壳”。

比如：

```js
function Wrap(core){
    this.core = core;
}
var oWrap = new Wrap(obj);
```

上面的oWrap就是obj的一个wrap。rwrap是这样一个算子：

```js
	/**
     * 函数包装变换
     * @method rwrap
     * @static
     * @param {func}
     * @return {Function}
     */
    rwrap: function(func,wrapper,idx){
        idx=idx|0;
        return function(){
            var ret = func.apply(this, arguments);
            if(idx>=0) ret=arguments[idx];
            return wrapper ? new wrapper(ret) : ret;
        }
    },
```

这个函数其实就做这样的一件事情，它的参数如果是个非负整数idx，那么就返回将第idx个参数包装后的对象，否则就返回原函数的返回值。

大家可能会想这个有什么作用呢，其实是这样的，比如 dom.show(element)这样的方法我们可能原本实现上返回的是boolean值，但是为了支持像JQuery的链式调用，我们在retouch的时候会利用rwrap改写这个返回值，让它返回element的Wrap（什么是retouch，后续的文章会介绍），以支持后续的链式调用。

第三个算子是一个叫做methodize的方法，它的作用是把一般的function给转化为method。在这里我解释一下什么是一般的function和method。在这里，我们约定一个函数如果内部没有出现任何this，那么它就是一个普通function，否则就是一个method。例如：

```js
function Foo(){
    this.x = 10;
}
 
ArrayH.forEach = function(arr,callback,pThis){
    for (var i =0,len=arr.length;i<len;i++){
        if (i in arr) callback.call(pThis,arr[i],i,arr);
    }
}
```

上面两个函数，尽管Foo是一个全局函数，但是它内部有用到this我们说它就是一个method，而ArrayH.forEach虽然是ArrayH的方法，但是它内部没有出现this，所以它就是一个普通的function。

methodize可以将一个普通的function转化为一个method，它的具体实现代码如下：

```js
	/**
     * 函数包装器 methodize，对函数进行methodize化，使其的第一个参数为this，或this[attr]。
     * @method methodize
     * @static
     * @param {function} func要方法化的函数
     * @optional {string} attr 属性
     * @optional {boolean} chain 串化，如果串化，返回this，否则返回原来的函数返回值
     * @return {function} 已方法化的函数
     */
    methodize: function(func,attr,chain){
        if(attr) return function(){
            var ret = func.apply(null,[this[attr]].concat([].slice.call(arguments)));
            return chain?this:ret;
        };
        return function(){
            var ret = func.apply(null,[this].concat([].slice.call(arguments)));
            return chain?this:ret;
        };
    },
```

这里可以看到，methodize其实是把this给追加到参数列表的前面，即令函数的第一个参数为this。

所以： Array.prototype.forEach = methodize(ArrayH.forEach);之后，就可以直接用[].forEach了。

同样这里有两个细节的参数，第二个参数比较有用，它表示不但可以将一个function给methodize到一个对象上，还可以将其methodize到一个对象的某个属性上，这样，我就可以把一个函数methodize给一个Wrap对象。第三个参数也是为链式调用准备的，它的作用是，如果传一个true进去，那么将用this代替原来函数的返回值。

有了这三个基本的算子，我们看看可以做什么。

首先，我们可以写一个纯静态的无侵入的方法，然后将这个方法经过变换，让它支持批量调用、Wrap和链式操作，并且methodize到某个对象、类的prototyp或者某个Wrap上去，那么我们就可以以纯粹无侵入的方式设计我们的内核，最后将它给赋予到某些外在的Wrap或者对象上去从而定制出类似于JQuery那样方便的API来，这个过程我们称之为retouch，将在下一篇文章中详细介绍。