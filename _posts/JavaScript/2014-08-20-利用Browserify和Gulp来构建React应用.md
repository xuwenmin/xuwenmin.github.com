---
layout: post
category : JavaScript
tagline: ""
tags : [Browserify,Gulp,React]
---
{% include JB/setup %}


JS的世界发展的非常快,有很多好用的框架,工具层出不穷,今天就来说说怎么结合`browserify`,`gulp`,`react`来构建前端应用.

---

下面我们先一一简单介绍下`browserify`,`gulp`,`react`

### Browserify

`browserify`是一个以`commonjs`规范来定义模块的打包工具.

`browserify`是一个开发工具,它允许我们以`nodejs`的代码风格来定义我们的模块,使用`module.exports`来导出模块功能,使用`require`来请求某个模块,跟`amd`规范不一样的是,我们不需要在模块外层包装一个`define`,可以完全跟`nodejs`后端模块通用,保持单个功能模块的重用性.

下面以一个简单的使用例子来说明下`browserify`的用法,假定下面两文件为兄弟关系

* module.js, 定义一个模块

```js
module.exports = {
    name: 'feenan'
};
```

* app.js, 来使用上面的模块

```js
var mod = require('./module');
console.log(mod.name);
```

* 使用,运行`bowserify app.js>bundle.js`

* 最后建立一个静态页面,只需引用上面的`bundle.js`文件就可以.

### Gulp

`gulp`是一个流构建工具,不同于`grunt`的是,前者是以代码优于配置的,而且不像`grunt`那样需要中间文件目录来方便后续任务的调用,`gulp`利用流和管道思想,极大的提高了构建效率,下面以一个简单的`Gulpfile.js`来说明它的用法

* 下面的任务是用来预处理`.styl`样式文件为`.css`文件 

```js

// 加载gulp模块
var gulp = require('gulp');
// 加载stylus gulp插件
var stylus = require('gulp-stylus');
// 定义要处理的文件路径信息.
var paths = {
  css: ['src/css/**/*.styl']
};
// 开始定义任务
gulp.task('css', function() {
  return gulp.src(paths.css)
    .pipe(stylus())
    .pipe(gulp.dest('./src/css'));
});
// 定义默认任务,方便使用 gulp命令直接调用
gulp.task('default', ['css']);

```

* 运行上面命令直接用 `gulp`就可以

### React

`react`是`facebook`发起创建的一个`UI`框架,通常被认为是`MVC`框架里的`V`,不过我喜欢把它理解为`MVVM`里的`VM`,因为它支持自动更新视图,利用内部的`state`属性,跟`angularjs`不一样的是,`react`更关注视图的渲染,所以它不是一个框架,只是一个`ui`库,可以非常方便的嵌入到第三方`MVV*`框架中去.

然后`react`使用了一个叫`JSX`的语法库来支持html元素的编写,下面放上一个简单的例子让大家对`react`有一个大概的了解

```js

/** @jsx React.DOM */
var HelloMessage = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});

React.renderComponent(<HelloMessage name="John" />, mountNode);

```

上面的代码应该放在`<script type="text/jsx"></script>`里,这里的`type=text/jsx`当里面的代码没有预处理的话是需要加上的,然后`/** @jsx React.DOM */`这个注释也是必填项,格式也得按照这个来写

下面以一个完整的例子来讲解三者之前的运用

### 完整的例子

主要分以下几步:

* 先以`jsx`和`js`规格来定义`commonjs`规范的模块
* 编写`Gulpfile.js`来预处理`jsx`文件,然后利用`browserify`来打包文件,最后压缩

> 一个普通的cjs规范的模块,data.js

```js

module.exports = {
	name: 'hello tina!',
	getName: function(){
		var r = Math.floor(Math.random() * 40 + 10);
		return 'hello ' + r;
	}
}

```

> 一个JSX格式的cjs规范的模块,module.jax,它依赖`react`,以及上面定义的普通模块

```js

/** @jsx React.DOM */
var React = require('react');  // Browserify!
var data = require('./data');
var HelloMessage = React.createClass({  // Create a component, HelloMessage.
	  render: function() {
	    return <div>Hello {data.getName()}</div>;  // Display a property.
	  }
});

module.exports = {
	invoke: function(){
		React.renderComponent(  // Render HelloMessage component at #name.
		  <HelloMessage name="feenan" />,
		  document.querySelector('body'));
	}
}

```

> 一个启动应用的app.js文件

```js

var app = require('./module.jsx');
app.invoke();

```

> 然后我们来编写我们`Gulpfile.js`

```js

/* gulpfile.js */
 
// Load some modules which are installed through NPM.
var gulp = require('gulp');
var browserify = require('browserify');  // Bundles JS.
var del = require('del');  // Deletes files.
var reactify = require('reactify');  // Transforms React JSX to JS.
var source = require('vinyl-source-stream');
var streamify = require('gulp-streamify');
var uglify = require('gulp-uglify');
 
// Define some paths.
var paths = {
  app_js: ['./src/js/app.js']
};
 
// An example of a dependency task, it will be run before the css/js tasks.
// Dependency tasks should call the callback to tell the parent task that
// they're done.
gulp.task('clean', function(done) {
  del(['build'], done);
});
 
 
// Our JS task. It will Browserify our code and compile React JSX files.
gulp.task('js', ['clean'], function() {
  // Browserify/bundle the JS.
  browserify(paths.app_js)
    .transform(reactify)
    .bundle()
    .pipe(source('bundle.js'))
    // .pipe(streamify(uglify()))
    .pipe(gulp.dest('./src/'));
});
 
// Rerun tasks whenever a file changes.
gulp.task('watch', function() {
  gulp.watch(paths.js, ['js']);
  gulp.watch(paths.app_js, ['js']);
});
 
// The default task (called when we run `gulp` from cli)
gulp.task('default', ['watch', 'js']);

```

上面的脚本实现了自动监听文件改变,然后自动预处理`jsx`文件,`browserify`打包文件,`uglify`压缩文件

关于例子里的`npm`模块依赖都可以从文件里找出来,然后使用命令`npm install [module_name]`安装就可以

### 总结

`react`是一个不错的`UI`库,可以方便的与`backbone`结合从而解决它里面视图功能的薄弱性,结合`browserify`,可以实现前后端`js`代码的重用性,`gulp`相比`grunt`来说性能与配置代码量就能得到很大的提升,有兴趣的话大家可以多用用它们.

相关参考资料:

* <a href="http://facebook.github.io/react/index.html" target="_blank">React</a>

* <a href="http://browserify.org/" target="_blank">Browserify</a>

* <a href="http://gulpjs.com/" target="_blank">Gulp</a>
