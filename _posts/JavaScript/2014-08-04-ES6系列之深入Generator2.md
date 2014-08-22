---
layout: post
category : JavaScript
tagline: ""
tags : [ES6,Generator,async]
---
{% include JB/setup %}


在<a href="http://www.ifeenan.com/~posts/JavaScript/2014-07-28-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator.md" target="_blank">上一篇文章</a>里深入的了解了`generator`的异常处理以及函数间的调用,这篇文章主要说说上一篇遗留的一些问题

* 异步方法的处理

此篇文章参考老外的一篇文章: <a href="http://davidwalsh.name/async-generators" target="_blank">传送门:)</a>

下面是Generator系列的相关文章链接

* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-07-27-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator.md" target="_blank">Generator基础篇</a>
* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-07-28-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator.md" target="_blank">深入Generator之异常处理与相互调用</a>
* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-08-04-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator2.md" target="_blank">深入Generator之异步方法处理</a>
* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-08-15-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator%E5%B9%B6%E5%8F%91%E8%B0%83%E7%94%A8.md" target="_blank">深入Generator之协程处理</a>


---

### 第一种方案

利用`generator`来调用异步主要是为了解决异常调用嵌套的问题,下面先模拟一个嵌套调用的异步过程.

```js

functin request(delay, callback){
	setTimeout(function(){
		callback({
			name: 'feenan'
		})
	}, +delay);
}

request(1000, function(msg){
	console.log(msg);
	request(2000, function(msg){
		console.log(msg);
	})
})

```

然后我们来看看怎么用`generator`来`同步`处理这里请求,首先我们来定义生成器函数.

```js

function* main(){
	var result = yield request(1000);
	console.log(result);
	var result1 = yield request(2000);
	console.log(result);
}

var it = main();

```
此时我们需要改进下`request`函数.

```js
functin request(delay){
	setTimeout(function(){
		it.next({
			name: 'feenan'
		})
	}, +delay);
}
```

注意上面的异步函数里增加了`it.next`,这样就能保证回调的值能传入`main`函数里去.因为`next`函数的参数是上一次`yield`表达式的值,第一次给`next`传参默认会忽略.

上面的方案虽然实现了异步方法的扁平化处理,但是有些缺点

* 异步方法深度依赖生成器实例,倒致不能重用
* 不能处理多个异步同时调用的情况

来看看第二种方案是怎么解决这种问题的?

### 第二种方案promise

虽然上面的方案可以实现异步调用的扁平化处理,但是仍然存在问题,下面我们来用`Promise`来实现一个更好的扁平化方案

`Promise`是`es6`提供的原生支持`promise规范`的类,提供有`then`,`all`,`catch`方法.我们先来改装下`request`方法

```js
function request(id, delay){
	return new Promise(function(resolve, reject){
		setTimeout(function(){
			if(id == '1') reject(new Error('new Error!'));
			resolve({
				name: 'feenan',
				id: id
			});
		}, +delay);
	});
}
```

`Promise`通过`new`一个实例,传入待执行的异步方法,内部利用`resolve`,`reject`来导出成功和失败的信息.可以看出这个方法跟`generator`是完全没关系的,已经解藕了,而且也可以重用在别的类库里,然后我们再来定义我们的生成器函数.

```js

function* main(){
	try{
		var result = yield request('1', 1000);
		if(result.name == 'feenan'){
			var result1 = yield request('2', 2000);
		}
		return 'done!';
	}catch(err){
		console.log(err);
	}
	
}

```

此处的`try...catch`是用来捕获里面异步方法的异常的,通过`throw`方法,这个等会儿会讲到.

也许大家会看到,上面的`main`方法的`yield`会返回一个`promise`对象,但是`result`应该会接收到异步回调结果的,这里我们应该怎么处理呢?对,我们还需要定义一个运行生成器函数的函数.

```js

function runGenerator(g){
	return new Promise(function(resolve, reject){
		var it = g(), ret;
		(function iterate(val){
			ret = it.next(val);
			if(!ret.done){
				// 检查是否是promise对象
				if('then' in ret.value){
					ret.value.then(iterate).catch(function(err){
						it.throw(err);
					});
				}else{
					setTimeout(function(){
						iterate(ret.value);
					}, 0)
				}
			}else{
				resolve(ret);
			}
		})();
	});
}

```

代码看起来有点复杂,我一一来解释.

首先函数本身返回一个`promise`对象用来处理生成器函数返回值,内部定义了一个`iterate`立即执行函数,`runGenerator`内部会先生成一个迭代器名为`it`,参数即是生成器函数,然后`iterate`会先调用`it.next`传递参数,默认第一次参数会忽略,然后检查`it.next`的返回值,这个返回值属性格式默认为`{value: , done: }`,`value`为`yield`后表达式的返回值,`done`代表迭代是否完毕,因为`yield`后面返回的是一个`promise`对象,所以检查`ret.value`是否是一个`promise`对象,是的话则调用`then`方法传递当前`iterate`本身,这就是运行函数的奥妙所在,因为此时会把异步回调的结果传递给下一次的`it.next`,这样`main`函数里的`result`就能接收到异步回调的结果了,然后就是不断重复上面的步骤直到执行完毕,利用`resolve`向外抛出生成器函数的返回值,我们来看看运行的样子

```js
runGenerator(main).then(msg){
	console.log(msg); // => done!
}
```

另外我们还可以利用`promise.all`方法达到异步同时调用多个方法的功能,此处只需要定义一个多个异步调用方法,`promise.all`会调用多个`promise`对象最终只返回一个`promise`对象

```js

// 包装多个异常请求成功之后返回一个新的promise
// 利用Promise.all类方法
function requestAll(){
	var promises = [1000, 2000, 3000].map(function(v, k){
		return new Promise(function(resolve, reject){
			setTimeout(function(){
				resolve({
					name: 'feenan',
					id: v
				})
			}, v);
		});
	});
	return Promise.all(promises);
}

```

然后我们修改`main`函数里的`request`即可

```js
// var result = yield request('1', 1000);
var result = yield requestAll(); 

```
此处需要注意下,上面的`result`是一个数组,里面每项为单个`promise`对象的返回值

可以看到利用`promise`+`generator`可以完美的原生实现异步调用扁平化,这对复杂业务逻辑的实现是相当好的.

下面我再说说`es7`里对于异步调用方法的更进一步的完美支持,而且代码量更少

### 第三种方案async

`es7`里提供了`async`语法来定义异步函数,配合`await`关键字轻易就能实现上面的功能

```js

function request(id, delay){
	return new Promise(function(resolve, reject){
		setTimeout(function(){
			if(id == '1') reject(new Error('new Error!'));
			resolve({
				name: 'feenan',
				id: id
			});
		}, +delay);
	});
}


async function main() {
    var result1 = await request('1', 1000);
    console.log(result1);
    var result2 = await request('2', 2000);
    console.log(result2);
    console.log( "The value you asked for: " + resp.value );
}

main();

```

`await`告诉引擎需要等后面的异步执行完成之后才能执行下面的语句,这是原生支持的,不需要像上面那样定义运行函数来一步一步执行.

`await`后面必须是一个`promise`对象实例,所以这里也的用上`promise`特性.

不过`async`语法成为标准之路还很遥远,庆幸的是`traceur`支持这个语法.


### 总结

最后我觉的还是`promise`和`generator`是完美解决异步调用的方案,以上所有实例都可以通过`traceur`命令来运行.







