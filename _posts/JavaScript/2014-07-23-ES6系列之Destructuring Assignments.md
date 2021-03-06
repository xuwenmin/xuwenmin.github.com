---
layout: post
category : JavaScript
tagline: ""
tags : [ES6,Destructuring Assignments]
---
{% include JB/setup %}


`Ecmascript 6`简称`es6`,是`javascript`下一代标准,还处在开发阶段,估计2014年底发布,有关更多浏览器对`es6`的支持情况,<a href="http://kangax.github.io/compat-table/es6/" target="_blank">点击这里</a>

今天说说`es6`里对赋值语句的改进,简称`解构赋值`.

---

### 解构赋值

> 所谓解构赋值其实就是按照模式匹配进行批量赋值

下面的是以往的赋值语句

```js

var a = 1;
var b = 2;
var c = 3;

```

以往的方式对于赋值多个变量的时候代码比较多而且不方便,那么`es6`里对它是怎么改进的呢?

通过以对象或者数组结构组装数据进行赋值,要保证赋值左右值类型相同,先来说说数组赋值.

### 数组赋值

先上一段简单的代码

```js

var [a, b, c] = [1, 2, 3]; // => 1 2 3

```

上面就是给变量`a, b, c`分别赋值`1,2,3`,是不是感觉代码立马简便不少呢.这只是赋值语句最简单的用法,下面一一列出别的赋值特点

* 左值支持默认值

```js

var [a = 1] = []; // => 1

```

* 左值支持`...`语法来装载剩余的右值，只能放在最后

```js

var [a, b, ...c] = [1, 2, 3, 4, 5, 6]; // => 1 2 [3, 4, 5, 6]

```

* 左值支持嵌套多层,支持嵌套数组与对象,这里提前说下对象赋值要保证`键值`相同

```js

var [a, [b], [[c]] ] = [1, [2], [[3]]]; // => 1 2 3

var [a, [ {b} ], [c] ] = [1, [ {b: 2} ], [3] ]; // => 1 2 3

```

下面说说利用对象作为左右赋值类型来进行快速赋值


### 对象赋值

先上一段简单的代码

```js

var { a, b } = { a: 'hello' , b: 'feenan'}; // => hello feenan

```

对象类型跟数组类型的区别在于对象是以`键值`来匹配的,而数组是以`位置`来匹配的,所以对象赋值对顺序没要求,但是数组需要以左值顺序来赋值的

下面列出一些关于对象赋值的一些特点,有些跟数组赋值相似

* 左值支持默认值

```js

var { a = 1 } = {}; // =>1

```

* 左值支持嵌套多层,支持嵌套数组与对象

```js

var { a: [b, {c}] } = {
	a: ['hello', { c: 'feenan'} ]
};

// => undefined hello feenan

```
上面的赋值只会给`b,c`赋值,`a`没有赋值,经测试发现只要左侧内的变量包含子变量的话,则本身是不会赋值的

说完了数组与对象类型的赋值语句,下面再说说其它用途

### 赋值语句用途

> 变量互换


```js

var [x, y] = [1, 4];

[x, y] = [y, x]; // x => 4 , y => 1

```

> 函数默认参数

```js

function abc({
	x = 1,
	y = 2
}){
	console.log(x, y);
}
abc({x:7}); // => 7 2
abc({y:7}); // => 1 7
abc({}); // => 1 2

```

> 导出指定模块的功能

```js

var {open, close} = require('fs');

```

> 利用for...of语法获取map键值对

```js

var map = new Map();
map.set('action', 'hello');
map.set('info', 'feenan');

for(let [key, value] of map){
	console.log(key, value); // => action hello & info feenan
}

```


### 运行

关于上面的例子可以通过安装`traceur`模块来进行测试

> npm install -g traceur

运行例子如下

> traceur demo.js

### 总结

个人比较喜欢此次`es6`的赋值语法糖,希望各大浏览器厂商赶紧提供支持吧 :)










