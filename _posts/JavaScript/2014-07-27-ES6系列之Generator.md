---
layout: post
category : JavaScript
tagline: ""
tags : [ES6,Generator]
---
{% include JB/setup %}


`Ecmascript 6`简称`es6`,是`javascript`下一代标准,还处在开发阶段,估计2014年底发布,有关更多浏览器对`es6`的支持情况,<a href="http://kangax.github.io/compat-table/es6/" target="_blank">点击这里</a>

今天说说`es6`里新增的`Generators`.

下面是Generator系列的相关文章链接

* <a href="http://www.ifeenan.com/JavaScript/2014-07-27-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator.md" target="_blank">Generator基础篇</a>
* <a href="http://www.ifeenan.com/JavaScript/2014-07-28-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator.md" target="_blank">深入Generator之异常处理与相互调用</a>
* <a href="http://www.ifeenan.com/JavaScript/2014-08-04-ES6%E7%B3%BB%E5%88%97%E4%B9%8B%E6%B7%B1%E5%85%A5Generator2.md" target="_blank">深入Generator之异步方法处理</a>
* <a href="http://www.ifeenan.com/JavaScript/2014-08-15-ES6%E7%B3%BB%E5%88%97%E4%B9%8BGenerator%E5%B9%B6%E5%8F%91%E8%B0%83%E7%94%A8.md" target="_blank">深入Generator之协程处理</a>

---

### Generator

`generator`简单的说就是提供了一种控制函数内部执行状态的功能,以往的普通函数只要开始执行则就不能中止,而`generator`函数不一样,提供了一种可以中止函数的手段,下面说说它的一般用法.

* function*  这是定义`generator`函数的语法
* yield  定义在函数内部实现不同的状态

看下面一个简单的例子

```js

function* build(){
	yield 'step1';
	yield 'step2';
	yield 'step3';
	return 'done';
}

var generator = build();
generator.next();  // => {value: 'step1', done: false}
generator.next();  // => {value: 'step2', done: false}
generator.next();  // => {value: 'step3', done: false}
generator.next();  // => {value: 'done', done: true}

```

首先我们执行`generator`函数来生成一个待遍历的函数,然后不断调用`next`方法来执行,它将返回一个对象包含`value, done`两个属性,前者是函数体内`yield`后面的值,后者代表函数遍历是否完成.

这里重点说下`yield`关键字

每次调用`next`方法时函数体内都会停留在碰到的下一个`yield`关键字那,并把关键字后面的内容作为返回值,下一次再调用`next`方法重复上面的操作直到最后没有遇到`yield`为止.假如函数体内不包含任何一个`yield`，则生成的遍历函数只是一个延迟执行的函数而以,像下面这样的

```js

function* delay(){
	return 'hello feenan';
}

var iterator = delay();

iterator.next(); // => {value: 'hello feenan', done: true}
iterator.next(); // => {value: undefined, done: true}

```

另外`yield`表达式还可以返回值用来在下一个`next`里调用,返回值只能通过`next`函数来传递,不过第一次调用`next`时传递参数无效，因为在它之前不存在`yield`,看下面的例子

```js

function * build(){
	yield 'step1';
	var step = yield 'step2';
	if(step == 3){
		yield 'step3';
	}else{
		yield 'step4';
	}
	yield 'step5';
	return '执行完毕!';
}

var generator = build();
console.log(generator.next());  // => {value: 'step1', done: false}
console.log(generator.next());  // => {value: 'step2', done: false}
console.log(generator.next(3));  // => {value: 'step3', done: false}
console.log(generator.next());  // => {value: 'step5', done: false}
console.log(generator.next());  // => {value: '执行完毕', done: true}

```

注意: `next`传递的参数值是传递给上一次`yield`表达式,所以上面例子里第三次调用`next`传递参数其实是把值传递给第二个`yield`表达式

### generator 将会用在什么地方呢?

上面说了这么多`generator`的用法,那么它可以用在什么地方呢

* 假如一个动作可以分成多步进行，并且想控制每步的时间的时候可以用`generator`函数

### 总结

今天只是简单的介绍`generator`的用法,更多的问题其实都没有说

* 假如`yield`后面是异步函数怎么办?
* 怎么处理`yield`的异常情况

这个以后会另起一篇文章你介绍,请保持关注.

本文参考这篇文章

<a href="http://davidwalsh.name/es6-generators" target="_blank">The Basics Of ES6 Generators</a>



