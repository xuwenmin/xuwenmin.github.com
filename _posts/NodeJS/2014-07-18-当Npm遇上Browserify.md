---
layout: post
category : NodeJS
tagline: ""
tags : [nodejs,npm,browserify]
---
{% include JB/setup %}

`npm`是`nodejs`包管理工具,提供了模块的发布以及下载功能,与`bower`不同的是,模块文件保存在`npm`平台上面

`browserify`是`npm`在前端项目里延伸的神器,有了它之后,前后端可以共用一个`commonjs`规范的模块

关于各种包管理之前的比较,可以参考知乎上面的一个问题,<a href="http://www.zhihu.com/question/24414899" target="_blank">bower,spm,npm</a>

---


### 安装 npm & browserify

`npm`基本上安装`nodejs`的时候就会自动安装

安装`browserify`

> npm install -g browserify

因为这是命令行使用的,所以推荐使用`-g`参数

### 使用 browserify

* 首先定义一个`commonjs`规范的模块，功能比较简单,名为`app.js`

```js

var app = {
	get: function(){
		return 'feenan!';
	}
}

// 此处对外导出模块功能

module.exports = app;

```

然后我们来定义一个使用这个模块的文件,名为`main.js`

```js

// 此处引用模块跟`nodejs`里一样
var app = require('./app.js')

var p = document.createElement('p');
p.textContent = app.get(); // -> 'feenan'
document.body.appendChild(p);

```

到了这里也许大家就会怀疑,前端浏览器里根本就没有`require`,`module`的定义,上面的文件应该会报错吧,没错,假如直接在html文件里引用上面的js文件的话,肯定会报错,现在是`browserify`出马的时候了

执行一条非常简单的命令就OK了

> browserify main.js > bundle.js

上面的`bundle.js`可以自定义别的名字,最后我们只需要把这个新生成的文件引入到html文件内即可,不用再引入别的文件

```html
<!doctype html>
<html>
<head>
	<title>browserify demo</title>
	<meta charset="utf8">
</head>
<body>

	<script src="js/bundle.js"></script>
</body>
</html>

```

最后在`chrome`里打开这个页面将会输出`feenan`的字样,是不是感觉很棒,确实，上面的`app.js`文件也可以拿到`nodejs`里去引用，因为它遵守`commonjs`规范

### 自动执行命令 watchify

当每次修改文件,然后执行上面的命令,这个过程其实是比较浪费时间的,幸运的时,`browserify`社区提供了一个工具`watchify`,先来安装它吧

> npm install -g watchify

然后在命令行里使用它

> watchify main.js -o bundle.js

这样当你每次修改main.js之后,就会自动生成转后的入口文件,有兴趣想研究`browserify`原理的,可以直接看看生成之后的js文件

想了解更多的使用方法以及原理的可以点击下面的文件
* <a href="http://browserify.org/" target="_blank">browserify</a>
* <a href="https://www.npmjs.org/package/watchify" target="_blank">watchify</a>

### 总结

`npm`与`browserify`的结合使用,将会有效的利用模块重用,对提高工作效率有很大的帮助.最后说下`chrome 36`发布了,听说对`web 组件`规范又支持了不少.