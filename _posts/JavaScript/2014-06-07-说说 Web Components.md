---
layout: post
category : JavaScript
tagline: ""
tags : [webcomponents]
---
{% include JB/setup %}


### Web Components 的现状

到目前为止,`w3c`定义的`web components`已经包含了

* `Templates`, 提供一个包含html,css,js的代码片段,类似于一般模板引擎里的视图
* `Custom Elements`, 提供一个自定义html元素的接口，支持对现有html元素的扩展
* `Imports`, 提供对模板或者自定义元素这些资源的加载支持
* `Shadow DOM`, 提供一个对外隐藏的web片段,而且支持独立的样式,不会破坏文档内的样式


web 组件的技术规范的制定就是为了保证前端组件的规范化，独立性，可重用性.先来说说`Templates`功能

---

### Templates

`Templates`提供了类似于模板引擎里的视图功能,默认浏览器是不会渲染此标签的内容的,只有当你真正用的时候才会去`render`,下面给一个简单的使用`Templates`的例子

```html

<template id="foo">
	<h2>Hello, world!</h2>
</template>

```

跟普通的html标签一样，定义一个id来标识这段代码,下面我们来使用这个模板

```js

var template = document.querySelector('#foo');
document.querySelector('body').appendChild(template.content.cloneNode(true));

```

上面的代码就是往body里插入模板里的内容,这里要注意下,想要获取模板内的内容,得使用模板元素的`content`属性,内容的类型是`documentFragment`，一般在同时插入多个html内容的时候，用`documentFragment`是最有效率的, 这里还要注意下，假如直接把`content`的内容插入到目标元素内之后，模板的内容就会丢失，所以为了保证别的目标元素也想插入这个模板的话，可以使用`Node`的`cloneNode`方法，这是所有html节点都有的方法,参数传递`true`的话会深度clone.

### Custom Elements

自定义元素api接口提供了一个可以自定义html元素的方法，支持新增方法以及对现有的元素进行扩展

`w3c`提供了`document.registerElement`方法来创建一个新元素,先来一个简单的实例，然后再分析它的实现

```js
var p = Object.create(HTMLElement.prototype);
p.hello = function () {
    this.appendChild("<h1>hello feenan</h1>");
}
var HelloFeenan = document.registerElement('hello-feenan', {prototype: p});
```
上面的代码创建一个名叫`hello-feenan`的元素,自定义的元素必须以`-`来连接,`Object.create`传递一个原型，将会创建继承这个原型的对象,然后对这个原型进行方法扩展，这里加了一个`hello`方法,定义完这个元素之后，怎么使用呢,其实它跟系统自带的元素一样，可以直接在html页里使用也可以动态的添加，这里使用`document.createElement`方法来添加，然后动态的添加`body`元素中

```js
var el = document.createElement('hello-feenan');

document.querySelector('body').appendChild(el);

// 调用hello方法来添加子元素
document.querySelector('hello-feenan').hello();

```

上面的代码就是应用自定义元素的好例子,其实这里的`hello`方法的内容可以在自定义元素的初始化事件回调里执行,说到这里,就得提提`customer element` 的支持的事件了

* createdCallback 自定义元素创建实例的时候触发
* attachedCallback 当元素实例被添加到`document`文档流中时触发
* detachedCallback 当从`document`文档流中删除自定义的元素实例时触发
* attributeChangedCallback 当元素实例的属性修改的时候触发

这四个事件的回调函数必须定义在自定义元素的原型对象上面，比如上面的`p`,可以通过`createdCallback`事件来实现上面的`hello`方法的内容

```js
p.createdCallback = function(){
	this.appendChild("<h1>hello feenan</h1>");
}
```
下面是一个完整的带自定义元素实现的例子

```html
<html>
	<head>
		<title>customer elements</title>
		<meta charset="utf-8">
	</head>
	<body>
		<script>
			
			var helloFeenan = Object.create(HTMLElement.prototype);

			helloFeenan.hello = function(){
				this.appendChild("<h1>hello feenan</h1>");
			}

			// 当元素被创建的时候触发此回调函数
			helloFeenan.createdCallback = function(){
				this.appendChild("<h1>hello feenan</h1>");
			}
			document.registerElement('hello-feenan', { prototype: helloFeenan});

			var el = document.createElement('hello-feenan');

			document.querySelector('body').appendChild(el);


		</script>
	</body>
</html>
```

### Imports

`imports`特性提供了以`link`方式来导入一段html文本的功能,相当于是`template`的进化版,方便做成模板文件

`w3c`规定`link`的`rel`属性传入`import`值,然后设置`src`为一个`html`文件,像下面这样的

```html

<link rel="import" href="import.html">

```

那么在引入页面的`dom`里怎么获取导入的文件内容呢,`w3c`给`rel=import`的元素提供了`import`属性来获取文件内容,本质上是`#document`类型,可以像下面这样来获取内容

```js

var link = $('link[rel=import]')[0];

var link_doc = link.import;

```


### Shadow DOM

`shadow dom`根据字面意思就知道这是隐藏在`dom`节点里的元素,提供了对`css,js,html`的封装从而形成一个独立组件的功能,这是一个非常不错的特性,结合`import`可以实现一个可重用的Web组件功能替代现在的一些ui组件.

下面我们来看看怎么使用`shadow dom`

`w3c`提供了`createShadowRoot`方法,这属于`element`的方法,在元素上面调用这个方法将在它下面创建`shadow dom`子元素,可以像下面这样来使用

```js

var shadowroot = $('.content')[0].createShadowRoot();
var elm = document.createElement('h1');
elm.textConent = 'hello feenan';
shadowroot.appendChild(elm);

```
### 一个完整的例子

目前`chrome 36` 已对`template, import, shadow dom`提供了完整的支持，所以下面提供一个结合三者的例子

* link元素需要引入的import.html文件

```html

<style>
	.content{
		margin: 20px;padding: 15px;
		width: 200px;height: 200px;
		border: 1px solid #ccc;border-radius: 5px;
		box-shadow: 0 0 10px red;
		overflow: auto;
	}
</style>
<div class="content">
	<h2>this is a test for ShadowRoot</h2>
</div>


```

上面的文件,`chrome`在用`link`引入之后,样式文件会放在`link.import`的`document`里的`head`元素内,普通`html`元素会放入`body`中，知道了内容的位置，方便下面用的时候查找

* index.html 运行例子的主html文件

```html

<!doctype html>
<html>
<head>
	<title>web components</title>
	<meta charset='utf8'>
	<link rel="import" href="import.html">
	<style>
		.content{
			margin: 20px;padding: 15px;
			width: 300px;height: 400px;
			border: 1px solid #ccc;border-radius: 5px;
			box-shadow: 0 0 10px #009933 inset;
			overflow: auto;
		}
		.btn{
			width: 120px;line-height: 30px;
			font-size: 16px;
			text-align: center;
			padding: 10px 15px;
			border-radius: 5px;
			border: 1px solid #ccc;
			box-shadow: 0 0 5px #ddd;
		}
		.btn:focus{
			outline: none;
			box-shadow: 0 0 10px green;
		}
	</style>
</head>
<body>
	<div class="content">
	</div>
	<p class="tmpl"></p>
	<p>
		<button class="btn add">添加</button>
	</p>
	<template id="tmpl1">
		<h1>hello feenan!</h1>
	</template>
	<script src="/lib/jquery/dist/jquery.min.js"></script>
	<script>
		$('.add').click(function(){

			var link = $('link[rel=import]')[0];

			// 获取引入文件的模板内容,返回的其实是#document类型
			var link_doc = link.import;
			
			// 创建shadow dom 元素,本质是一个documetn片段元素
			var shadowroot = $('.content')[0].createShadowRoot();

			// 创建样式
			var style = function(){
				var css = document.createElement('style');
				// 通过styleSheets获取模板内的样式内容
				css.textContent = link_doc.styleSheets[0].rules[0].cssText;
				return css;
			}

			// 创建内容
			var body = function(){
				return $(link_doc.body.innerHTML)[0];
			}

			shadowroot.appendChild(style());
			shadowroot.appendChild(body());
			$('.content').append($(link_doc).find('body').html());
			
			// 使用template来获取模板文件
			$('.content').append($('#tmpl1')[0].content.cloneNode(true));

		});
	</script>
</body>
</html>

```

* 以上的例子建议在`chrome 36`里运行


### 总结

非常高兴`chrome`已对`template,import,customer element,shadow dom `提供了完整的支持,更多关于各个浏览器对`web components`支持请<a href="http://jonrimmer.github.io/are-we-componentized-yet/" target="_blank">点击这里</a>

关于`web components`相关的网站，可以参考下面的地址,大部分都要翻墙浏览,因为基本上都是`google`维护的

<a href="http://webcomponents.org/" target="_blank">webcomponents 官网</a>

<a href="http://www.polymer-project.org/" target="_blank">Polymer 框架</a>, google出品的开发web组件的框架,兼容大部分浏览器

<a href="http://w3c.github.io/webcomponents/" target="_blank">w3c web 组件规范</a>





