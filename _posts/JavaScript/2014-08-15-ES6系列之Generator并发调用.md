## ES6系列之深入Generator.Concurrent

今天来说说`generator`之间的并发调用,分析生成器之间相互协调处理的功能.类似于其实语言里的`协程`概念.

此篇文章参考老外的一篇文章: <a href="http://davidwalsh.name/concurrent-generators" target="_blank">传送门:)</a>

下面是Generator系列的相关文章链接

* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-07-27-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator.md" target="_blank">Generator基础篇</a>
* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-07-28-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator.md" target="_blank">深入Generator之异常处理与相互调用</a>
* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-08-04-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator2.md" target="_blank">深入Generator之异步方法处理</a>
* <a href="http://www.ifeenan.com/~posts/JavaScript/2014-08-15-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator%E5%B9%B6%E5%8F%91%E8%B0%83%E7%94%A8.md" target="_blank">深入Generator之协程处理</a>

---

### CSP

这里先说下`CSP(Communicating Sequential Processes)`,为什么说这个呢,因为它是协程的核心思想

`CSP`代表程序里的子程序之间能够有条不紊的同时进行,而且能够相互合作完成一件事件,这里有几个关键点:

* Communicating 通信
* Sequential 序列化
* Processes 处理

但是因为`js`本身是单线程的,那么怎么在`generator`里实现并发以及相互传递呢,这里只有模拟这种效果.

### 共享变量

我们可以利用多个`generator`引用同一个变量,从而解决通信问题,只要保证同时只有一个`generator`来调用它就行,不过这个在单线程里完全不是问题.

因为要保证多个`generator`看起来是'并发'执行的，所以需要一种机制来检查,怎么控制在一个`generator`里跳到另外一个`generator`里去执行,就像操作系统里的CPU时间片一样,多个任务轮流的共享cpu的时间,这里也可以通过共享变量来做为标识,假如`generator`内部碰到这个标识那么就自动的切换到另一个`generator`.

不过这里又有一个切换的问题,我这里是这样的切换机制,按顺序切换,到最后了就自动切会第一个,所以这里也需要一个保存所有`generator`实例的地方.

下面我们看一个简单的例子

### 一个简单的协程例子

先定义两个可以实现`并发`的两个`generator`

```js

function multBy20(v){
    return new Promise(function(resolve, reject){
        setTimeout(function(){
            resolve(v * 20); 
        }, 500);
    });
}

function addTo2(v){
    return new Promise(function(resolve, reject){
        setTimeout(function(){
            resolve(v + 2); 
        }, 200);
    });
}

function *foo(token) {
    var value = token.messages.pop(); // 2
    token.messages.push( yield multBy20( value ) );
    // 这是一个协作的标志,代表转移到其它的generator
    yield token;
    yield "meaning of life: " + token.messages[0];
}

function *bar(token) {
    var value = token.messages.pop(); // 40
    token.messages.push( yield addTo2( value ) );
    // 这是一个协作的标志,代表转移到其它的generator
    yield token;
}

```

上面代码定义了两个`generator`,两个函数的参数`token`最终会引用同一个对象,最后要实现的功能就是这样的

> `foo`先从共享变量弹出数字2,利用`multBy20`得到40,然后添加到共享变量的消息数组里去,碰到`token`标识,切换到`bar`里去处理消息数组,弹出40,利用`addTo2`得到结果42,添加到消息数组里去,碰到`token`标识,切换到`foo`里去完成收尾操作.

上面两个`generator`是交替执行的,哪个先完成就返回哪个的内容,运行`generator`的代码跟上一篇文章里的差不多,只是这里扩展了`generator`切换的逻辑.

```js

// 模拟其它语言里的协程概念
function runGenerator(){
    var its, ret, callback,last,cache = {}, cur = 0;
    var token = {};
    token.messages = [];
    token.messages.push(number);
    // 运行generator内部yield的函数
    function iterate(val){
        last = ret;
        ret = cache[cur].next(val);
        console.log('inner:', cur);
        if(!ret.done){
            if(ret.value == token){
                // 切换到另一个generator
                if(cur == (its.length - 1)){
                    cur--;
                }else{
                    cur++;
                    cache[cur] = its[cur](token);
                }
                iterate();
            }
            // 检查是否是promise对象
            if('then' in ret.value){
                ret.value.then(iterate);
            }else{
                setTimeout(function(){
                    iterate(ret.value);
                }, 0)
            }
        }else{
            setTimeout(function(){
                callback.call(null, last || ret);
            }, 0);
        }
    };
    // 包括操作的一个实例对象
    var instance = {
        val: function(fn){
            callback = fn;
        },
        run: function(){
            its = [].slice.call(arguments);
            cache[cur] = its[cur](token);
            iterate();
            return instance;
        }
    }
    return instance;
}

```

运行代码如下:

```js
// 开始运行
runGenerator(2).run(foo, bar).val(function(msg){
    console.log(msg);
});

```
运行效果图如下:

<img src="http://xuwenmin.github.io/blog/img/generator-coroutines.png" alt="">

现在我们来简单的分析上面的代码

* `token`是一个共享变量,作为消息通道的载体以及切换`generator`的标识
* `cache`里保存所有运行的`generator`实例
* `iterate`为核心的运行`generator`实例的方法,主要负责处理异步返回值的情况以及切换`generator`
* `instance`为内部返回实例,用来作为链式调用的上下文

### 总结

其实利用`generator`模拟的并发调用功能可以用来做很多别的有趣事情,已有很多框架利用它来构建基础功能,比较有名的就是`nodejs`里的<a href="http://koajs.com/" target="_blank">koajs</a>.

想了解`generator`并发例子的可以参考文章头部那个老外链接.


