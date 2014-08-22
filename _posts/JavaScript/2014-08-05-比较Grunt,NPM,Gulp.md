---
layout: post
category : JavaScript
tagline: ""
tags : [grunt,npm,gulp]
---
{% include JB/setup %}

随着前端工程化的趋势,产生了越来越多的构建工具,而其中比较优秀的就是`grunt`,`npm`,`gulp`,今天我来说说这三者间的区别以及他们的优缺点.

---

相信一般前端开发者选择构建工具的时候,更多的是看个人习惯以及团队的情况.相信这三个构建工具总有一个会适合你的,我们先来看看`grunt`.

### Grunt

`grunt`是目前社区最成熟,插件支持最多的一个构建工具,不过它的配置项之多也常常被人诟病.下面就一个简单的例子来说说它的用法.

`grunt`运行之前需要全局安装命令行工具,本地安装`grunt`插件

> npm install -g grunt-cli

```js
// 包装函数
module.exports = function(grunt) {

  // 任务配置
  grunt.initConfig({
      concat: {
          foo: {
              files:{
                  'dist/main.js': 'source/*.js'
              }
          }
      }
  });

  // 加载编译JSDOC文档插件
  grunt.loadNpmTasks('grunt-contrib-concat');

  // 测试合并任务
  grunt.registerTask('concatdemo', ['concat:foo']);

};
```

上面就是一个合并文件的例子,假如任务内的操作比较多的话,配置文件就非常多,`grunt`适合对`nodejs`不是非常熟悉的情况下使用,而且自定义插件也非常方便,这个可以看我之前写的文章.

`grunt`在处理多个子功能的时候会频繁的调用`io`来读取文件,而且不支持任务模块的重用,也就是说不能添加任务的依赖.不过入门非常容易,这些只是我自己的一些看法,也许最适合你的才是最好的,下面我们再来看看`npm`.

### NPM

也许有些人会说`npm`不是包管理工具吗?怎么又成了构建工具了,其实它是可以当成构建工具来用的,奥秘就在`package.json`文件里的`scripts`属性上.这里是可以定义当前模块的一些构建功能的,比如当你开发一个有点规模的模块的时候,开发与发布引用的文件其实是不一样的,一般发布的时候都会提供压缩版或者一些测试用例,下面以一个简单的例子来说明

```js
  {
    "scripts": {
	    "test": "phantomjs test/vendor/runner.js test/index.html?noglobals=true",
	    "build": "uglifyjs underscore.js -c \"evaluate=false\" --comments \"/    .*/\" -m --source-map underscore-min.map -o underscore-min.js",
	    "doc": "docco underscore.js"
  	},
  	"devDependencies": {
	    "docco": "0.6.x",
	    "phantomjs": "1.9.0-1",
	    "uglify-js": "2.4.x"
  	},
  }
```

上面的这个配置片段是`underscore`类库的配置,可以看出上面的构建属性有`test`,`build`,`doc`,这三个分别代表三个任务,运行命令如下

> npm run 

`npm run`后跟任务名就可以,任务内容支持`bash`脚本,也支持`npm`模块本身提供的命令行命令,像上面的`uglifyjs`本身就提供有命令行压缩命令,运行`npm run`命令之前要保证`devDependencies`里的依赖模块已经安装到本地,没有的话可以使用`npm install`命令安装.

最后要说的是,任务的内容了可以是自定义模块,比如可以像这样的

```js
	"scripts": {
	    "demo": "./demo.js"
  	}
```

> demo.js

```js
#!/usr/bin/env node

console.log(__dirname);
console.log(process.cwd());

```

注意运行命令之前需要确保系统有执行`demo.js`的权限,可以使用

> chmod 777 demo.js

打开访问权限,然后我们运行

> npm run demo

就会看到输出当前运行根目录内容,而且这里的`js`文件可以写很多自定义的东西.

最后要说明下,`npm`最适用于类linux系统,因为这些系统对命令支持非常友好,`windows`的话得安装模拟`*inux`的命令行工具

`npm`一般用在个人项目里,对于团队项目则不适用.最后说下`gulp`.


### Gulp

`gulp`跟`grunt`一样支持跨平台,不同于`grunt`需要`Gruntfile.js`,`gulp`需要`Gulpfile.js`,最核心的不同之处在于,`gulp`是以流为核心的,而`grunt`是以配置加上文件`io`为核心的.

`gulp`是流为核心然后通过管道来输入输出各个子功能的内容以方便后续操作,这样可以提高`io`访问效率,自定义插件方面要比`grunt`要求要高点,下面以一个简单的例子看看它的用法.

运行`gulp`的系统要求

> touch Gulpfile.js

> npm install -g gulp

> npm install gulp


```js

var gulp    = require('gulp'),
	uglify  = require('gulp-uglify'),
	size 	= require('gulp-size');

gulp.task('jsmin', function(){
	return gulp
		.src('./app.js')
		.pipe(size())
		.pipe(uglify())
		.pipe(size())
		.pipe(gulp.dest('dist/main'));
});

```

`gulp`是以`src`为开始,里面传递任务需要的源文件,文件格式跟`grunt`相同,后面都是以`pipe`来传输前一步的输出内容,最后可以输出到一个目标文件内通过`dest`方法.

`gulp-size`是一个统计管道里面内容的大小的,上面是用它来显示出压缩前后的大小用来对比用的.

`gulp`跟`grunt`一样也支持任务里包含多个子任务,像这样的

```js
	gulp.task('build', ['jshint', 'jsmin']);
	// 以 gulp build 命令来运行

```

不过跟`grunt`不一样的是,上面的多个子运任务是异步执行的,而`grunt`是同步执行的.

`gulp`也支持像模块里的依赖注入,而且运行自己的任务之前它会保证依赖都运行完毕,像下面这样的

```js
gulp.task('test', ['jsmin'], function(){
	return gulp
		.src('dist/main/*.js')
		.pipe(gulp.dest('build'));
});
```
上面的代码功能是当压缩完js之后,把压缩之后的文件内容重新copy到一个新的地方.

也许`gulp`唯一的缺点就是社区插件没`grunt`丰富,但是随着越来越多的人进入`gulp`,相信这个也不是问题.

### 总结

看了上面三个构建工具的分析,大家都喜欢用哪一个呢,个人推荐用`gulp`,因为它代码量少而且效率比`grunt`要高,不过还是那句老话,`适合自己的才是最好的`.


