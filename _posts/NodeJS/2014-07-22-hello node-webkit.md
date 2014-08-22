---
layout: post
category : NodeJS
tagline: ""
tags : [nodejs,node-webkit]
---
{% include JB/setup %}

`node-webkit`是开发桌面webapp的框架,基于`Chromium`与`nodejs`,利用`html,css,js`混合`nodejs`模块可以开发出漂亮的桌面应用.

更多详情可以点击这里,<a href="https://github.com/rogerwang/node-webkit/wiki" target="_blank">node-webkit</a>

---

下面我们以`node-webkit`框架来建立一个`mac`系统上面的app,构建工具利用<a href="http://codeb.it/nuwk/" target="_blank">Nuwk!</a>.

###  安装 node-webkit

<a href="https://github.com/rogerwang/node-webkit#downloads" target="_blank">点击这里</a>选择相应系统的`node-webkit`安装包,然后直接解压把`node-webkit.app`放在`应用程序`里

### 安装 numk!

<a href="http://codeb.it/nuwk/" target="_blank">点击这里</a>进去下载,然后把下载的解压包解压之后,将`nuwk!.app`放入`应用程序`里

### 创建一个应用

打开`nuwk!.app`,`mac`下的可以直接用`alfred`里输入`nuwk`即可,打开之后,图片如下

<img src="http://xuwenmin.github.io/blog/img/nuwk.png" alt="">

点击创建项目,然后输入项目名称,最后完成,点击修改默认是用`sublime`打开的,这里我输入`hello-feenan`,大概的文件结构如下

<img src="http://xuwenmin.github.io/blog/img/strcuture.png" alt="">

`App`目录为程序文件,包括js,css,html,nodejs模块

`Build`为`nuwk!`最后生成`app`的地方

`Resources`为`app`的静态资源,包括`app`的图标文件`nw.icns`以及一个必需的`Info.plist`文件,默认都会自动生成,基本上不用改

默认的`index.html`是这样的

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node.js <script>document.write(process.version)</script>.
  </body>
</html>
```

此处的代码根据`nodejs`里的`process`对象获取`nodejs`的版本信息,相当于告诉大家在`dom`里是可以操作`nodejs`原生对象的

### 配置 app 

配置`app`的相关属性在`package.json`文件里

* name 代表项目名称
* main 代表app起始页面默认为index.html
* version 代表app 版本
* window 它包含很多子属性,比如宽度，高度，是否有工具条，最大化，最小化等

想了解更多的配置app的参数信息可以点击这里,<a href="https://github.com/rogerwang/node-webkit/wiki/Manifest-format" target="_blank">package.json 配置</a>

### 增加一个浏览文件的功能

首先增加相关的第三方库,这里使用`npm`来安装,因为`dom`里可以直接使用`require`来加载模块

> npm install jquery --save

先修改默认的`index.html`文件如下

```html

<!DOCTYPE html>
<html>
<head>
    <title>打开文件</title>
    <meta charset="utf8">
    <link rel="stylesheet" href="css/app.css">
    <link rel="stylesheet" href="css/reset.css">
	<link rel="stylesheet" href="css/bootstrap.css">
</head>
<body>
	<div class="container">
		<h1>打开文件</h1>
		<p>
			<input id="readFile" type="file">
		</p>
		<div class="row">
			<textarea class="col-md-10 text" id="info"></textarea>
		</div>
		
	</div>
	<script src="js/app.js"></script>
</body>
</html>

```

相关的`css`文件后面会放在完整的代码包里，这里就不写了,默认我们会在`App`里建立`js`和`css`目录

读取文件我们使用`html5`本地文件api,相应的`app.js`代码如下

```js

var $ = require('jquery');

$('#readFile').change(function(){
	var path = $(this).val();
	var reader = new FileReader();
	reader.onload = function(e){
		$('#info').val(this.result);
	};
	try{
		reader.readAsText($(this)[0].files[0], 'utf-8');
	}catch(e){
		console.log(e);
	}	
});

```

编写完上面的代码之后,我们可以来看看怎么跑起来.

### 以浏览器方式运行 app

打开`nuwk!`app,选择刚才的新建的app,上面有三项，如图

<img src="http://xuwenmin.github.io/blog/img/nuwk-run.png" alt="">

点击上图的`Run project`,就会打开一个页面,看下图

<img src="http://xuwenmin.github.io/blog/img/nuwk-debug.png" alt="">

点击右侧的设置可以出现一个熟悉的chrome调试窗口,里面的功能跟chrome浏览器的开发者工具窗口差不多

### 以本地app方式运行

打开`nuwk!`app,选择刚才新建的app,看上图,选择`build project`,系统会在`build`目录里生成一个.app的文件,这个是可以直接打开的

### 完整版代码下载

<a href="http://pan.baidu.com/s/1qWHggwc" target="_blank">点击这里下载</a>,下载完之后进入`App`目录运行

>  npm install

安装需要的模块依赖,然后打开`nuwk!`选择这个项目直接`build project`就可以了,最后运行生成的`.app`文件

### 总结

`node-webkit`是一个非常不错的开发桌面app的框架,而且完美的支持`nodejs`,相信两者的结合可以创造出更多更好用的app.






