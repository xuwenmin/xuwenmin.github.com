---
layout: post
category : JavaScript
tagline: ""
tags : [ES6,Generator]
---
{% include JB/setup %}

## ES6系列之深入Generator

在<a href="http://www.ifeenan.com/JavaScript/2014-07-27-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator/" target="_blank">上一篇文章</a>里简单的介绍了`generator`的用法,这篇文章主要说说上一篇遗留的一些问题

* 异常处理
* generator之间的调用

此篇文章参考老外的一篇文章: <a href="http://davidwalsh.name/es6-generators-dive" target="_blank">传送门:)</a>.

下面是Generator系列的相关文章链接

* <a href="http://www.ifeenan.com/JavaScript/2014-07-27-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator.md" target="_blank">Generator基础篇</a>
* <a href="http://www.ifeenan.com/JavaScript/2014-07-28-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator.md" target="_blank">深入Generator之异常处理与相互调用</a>
* <a href="http://www.ifeenan.com/JavaScript/2014-08-04-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator2.md" target="_blank">深入Generator之异步方法处理</a>
* <a href="http://www.ifeenan.com/JavaScript/2014-08-15-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator%E5%B9%B6%E5%8F%91%E8%B0%83%E7%94%A8.md" target="_blank">深入Generator之协程处理</a>


---


### 异常处理

下面先以一个简单的例子来说明怎么捕获`generator`里的异常.

```js

function * build(){
	try{
		throw new Error('from build!');
		yield 'haha';

	}catch(err){
		console.log('inside error: ' + err);
	}
}

var it = build();

it.next(); // => inside error: Error: from build!

```

上面的例子就是在`generator`里面增加`try...catch`语句来捕获函数内部的异常,然后`generator`本身也提供了一个方法用于向外抛出异常

> throw  此方法可以向外抛出异常,由外部的`try...catch`来捕获

```js

function * build(){
	try{
		yield 'haha';
	}catch(err){
		console.log('inside error: ' + err);
	}
}

var it = build();
try{
	it.throw('from it'); // => out error from it
}catch(err){
	console.log('out error: ' + err);
}

```
假如`generator`内部没有加上`try...catch`,然后内部有异常的话,则异常默认会向上抛出,像上面那样的话则可以捕获.

注意：上面`generator`里的异常捕获只支持同步方法调用

### generator 间调用

之前我们都是那一个`generator`来举例子,下面我们说说怎么在`generator`函数里调用别的,先上一个例子,然后我们说说它是怎么工作的吧

```js

function* build(){
	var a = yield 1;
	var b = yield 2;
	var c = yield* child();
	console.log(a, b, c);
}
function* child(){
	var x = yield 3;
	var y = yield 4;
	console.log(x, y);
	return 'child';
}

var it = new build();
it.next();
it.next('a');
it.next('b');
it.next('c');
it.next('d');
// => c d
// => a b child

```

`generator`提供了`yield*`的语法来支持调用别的`generator`函数

> yield* 后面跟上别的`generator`实例就可以遍历别的`generator`里的`yield`了

看到上面`child`里的`return`了没,这个跟普通调用`generator`不同的时,这个返回值默认会传递给`yield*`表达式,像上面那样然后给`c`本地变量赋值

普通`yield`表达式假如想有返回值的话,则只能依赖后续的`next`传递参数进来

上面的例子里只是写了一层调用,其实在`child`函数还可以调用别的`generator`函数,然后在里面产生的异常也会一层一层的传递到外面来的. 看下面的例子

```js

function* build(){
	var a = yield 1;
	var b = yield 2;
	try{
		var c = yield* child();
	}catch(err){
		console.log('build error: ' + err);
	}
	console.log(a, b, c, d); // =>此处会向上抛异常 d未定义
}
function* child(){
	var x = yield 3;
	var y = yield 4;
	console.log(x, y, c); // => 此处会向上抛异常 c未定义
	return 'child';
}

var it = new build();
it.next();
it.next('a');
it.next('b');
it.next('c');
try{
	it.next('d');
}catch(err){
	console.log('out error: ' + err);
}

// => c d
// => a b child

```

### 总结

上面主要说了下`generator`关于同步操作下的异常处理,以及`generator`互相调用的问题,下一篇文章主要讲讲`generator`里调用异步方法的情况.

