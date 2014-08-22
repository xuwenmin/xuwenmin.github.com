## 说说 BrowserSync

相信做过前端开发的人都知道,当你改变前端项目里的文件的时候,为了查看更改之后的效果需要手动的刷新浏览器,其实这个很浪费时间,那么这时候你可以选择用用`browser-sync`.

---

`browser-sync`是一个浏览器同步工具,可以帮你解决在项目开发时,文件保存之后,浏览器同步刷新.

本篇文章主要讲讲在下面几种情况下如何使用`browser-sync`

* 命令行
* Grunt   
* Gulp    
* npm run
* nodemon

### 安装

> npm install -g browser-sync

### 命令行中使用

标准命令如下

> browser-sync start [options]

下面说说常用的选项

>  --files 

后面带上相应监听的文件列表,格式例子如下:

* --files "js/*.js, css/*.css"
* --files "js/base.js, css/base.css"

> --server

表示启动一个内置的web服务器

> --proxy

这个一般用在有本地的`php`,`nginx`部署的web服务器下,例子如下:

* --proxy "localhost:3000"
* --proxy "www.cloud.cn"

最后来一个监听本地已有`www.c.cn`虚拟机的浏览器同步命令

> browser-sync start --proxy "www.c.cn" --files "js/*.js,css/*.css"

### Grunt中使用

想要在`Grunt`里使用则要添加它的插件,官方插件名称为`grunt-browser-sync`,

> npm install grunt-browser-sync --save-dev

运行上面命令之后,就可以在`Gruntfile.js`里增加配置文件了,下面是它的配置格式

```js

browserSync: {
  dev: {
    bsFiles: {
      src: 'public/**/*.{js,css}'
    },
    options: {
      proxy: 'localhost:3000'
    }
  }
}

```
首先定义一个`dev`目标,然后里面有两固定参数`bsFiles`和`options`,前者定义监听的文件列表,这里的文件格式模式比较灵活;后者里面可以定义`proxy`的值.

然后加载插件任务模块

```js

	grunt.loadNpmTasks('grunt-browser-sync');
	grunt.registerTask('default', ["browserSync"]);

```

这个结合`watch`,`compass`模块还可以实现修改的时候合并`sass`到`css`,然后同步浏览器,更多的例子点击这里,<a href="http://www.browsersync.io/docs/grunt/" target="_blank">传送门</a>

### Glup中使用

跟`grunt`一样,先安装`glup`插件`browser-sync`,然后在`Glupfile.js`里配置任务

```js

var gulp = require('gulp');
var browserSync = require('browser-sync');

gulp.task('browser-sync', function () {
  browserSync({
    proxy: 'localhost:3000',
    files: ['public/**/*.{js,css}']
  });
});

```

选项里一般配置`proxy`或者`files`即可,关于`glup`的详情例子可以看这里, <a href="http://www.browsersync.io/docs/gulp/" target="_blank">传送门</a>

### NPM中使用

这里使用`npm run`命令运行,相关的配置放在`package.json`的`script`属性里,命令如下:

>  browser-sync start --proxy "www.c.cn" --files "js/*.js,css/*.css"

### nodemon中使用

`nodemon`是保证使用`nodejs`作为服务器的时候,代码有变动时可以自动的重启相关的`nodejs`服务,这里可以结合使用`browser-sync`来保证静态资源的同步化

先定义一个`app.js`.

```js
var express = require('express');
var browserSync = require('browser-sync');
var app = express();
var port = process.env.PORT || 3000;

app.listen(port, listening);

function listening () {
  browserSync({
    proxy: 'localhost:' + port,
    files: ['public/**/*.{js,css}']
  });
}

```

然后运行下面的命令

> nodemon app.js

这样即可以保证服务器代码的变动自动重启,也能保证静态资源的浏览器同步化.

### 总结

远离低效率,就用`browser-sync`!!!

相关参考资料

* <a href="http://www.browsersync.io/">BrowserSync 官网</a>














