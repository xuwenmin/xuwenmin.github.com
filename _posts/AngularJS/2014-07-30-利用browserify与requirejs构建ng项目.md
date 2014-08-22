---
layout: post
category : AngularJS
tagline: ""
tags : [ng,源码]
---
{% include JB/setup %}

随着业务的增长,利用`ng`来构建项目时候,文件数量就会显著的上升,从而上线部署的时候就要考虑压缩合并问题,随着前端工程化的发展,现在已经有很多种第三方工具来实现开发与部署的便捷性,`browserify`与`requirejs`就是其中的两个比较好多的工具.

---

* browserify以`commonjs`模块开发规范来约束前端模块开发,最后上线时提供命令行生成合并文件,详情请 <a href="https://www.npmjs.org/package/browserify" target="_blank">点击这里</a>
* requirejs以`amd`模块开发规范来约束前端模块开发,最后上线的时候提供`r.js`命令行工具来生成合并压缩文件,详情请 <a href="http://requirejs.org/" target="_blank">点击这里</a>

### 一个普通的ng例子

下面先提供一个原生的ng例子,只显示核心的代码

```js

// 核心文件 app.js

(function () {

	  var app = angular.module('myApp', ['ngRoute']);

	  // 下面这个需要依赖angular-route模块
	  app.config(['$routeProvider', function ($routeProvider) {
		    $routeProvider.when('/home',
		    {
		      templateUrl: 'partials/home.html',
		      controller: 'HomeController'
		    })
		    .otherwise(
		    {
		      redirectTo: '/home'
		    })
	  }]);

	  app.controller('HomeController', ['$scope', 'usersService', function($scope, usersService){
		    $scope.title = 'Home';
		    $scope.users = usersService.getData();
	  }]);

	  app.factory('usersService', function () {
		    var service = {
		      getData: function(){
		        return [{name: 'feenan', info: 'fe'}, {name: 'tina', info: 'financy'}];
		      }
		    }
		    return service;
	  });

}());

```

### browserify 构建ng

下面以`browserify`的模块开发规范来重构上面的例子,项目结构大概这样:

* app
	* lib  这个下面存放第三方js库,例如jquery,angularjs
	* partials
		* home.html
	* controllers
		* homeController.js
	* services
		* usersService.js
	* app.js
* index.html

> homeController.js 

```js

module.exports = function ($scope, usersService) {
   
    $scope.title = 'Home';
    $scope.users = usersService.getData();

};
```

> usersService.js

```js
module.exports = function () {
    var service = {
      getData: function(){
        return [{name: 'feenan', info: 'fe'}, {name: 'tina', info: 'financy'}];
      }
    }
    return service;
}
```

> app.js

```js
// 加载所有的依赖
window.jQuery = require('./lib/jquery/jquery.min');
require('./lib/angular/angular.min');
require('./lib/angular-route/angular-route.min');

// 获取控制器与服务的依赖
var homeController = require('./controllers/homeController');
var usersService = require('./services/usersService');

// module up
var app = angular.module('app', [ 'ngRoute']);

// routes and such
app.config(['$routeProvider', function($routeProvider) {
  $routeProvider
    .when('/home',
    {
      templateUrl: 'partials/home.html',
      controller: 'HomeController'
    })
    .otherwise(
    {
      redirectTo: '/home'
    });
}]);

// create factories
app.factory('usersService', usersService);

// create controllers
app.controller('HomeController', ['$scope', 'usersService', homeController]);

```

然后我们用`browserify`来生成最终`index.html`引用的文件名,先`cd`到`app`目录

> browserify app.js -o main.js

最后我们来看看`index.html`代码

```html

<!doctype html>
<html lang="en" ng-app="myApp" >
<head>
    <meta charset="utf-8">
</head>
<body>
	<div ng-view></div>

	<script src="app/main.js"></script>
</body>
</html>

```

可以看到最终的页面里只需要引用一个js文件就可以了,这就是神奇的地方,因为`browserify`编译文件时候已经加载了一套自己的模块处理方案,有兴趣的话可以自己写个简单的项目分析下编译后的js文件



### requirejs 构建ng

下面我们来用`requirejs`来重构上面的例子,大概的目录结构如下:

* app
	* partials
		* home.html
    * controllers
		* index.js  // 包括所有的 控制器的入口
		* module.js // 创建控制器模块
		* homeController.js
	services
		* index.js  // 同控制器
		* modules.js 
		* userService.js
	* app.js  // 创建ng主模块的地方
	* main.js // requirejs配置文件地方
	* routes.js // 配置路由的地方

下面我们一一来创建这些文件,静态页文件除外

先说说`requirejs`配置文件

> main.js 

```js

require.config({
  paths: {
    'jquery': './lib/jquery/jquery.min',
    'angular': './lib/angular/angular.min',
	'angular-route': './lib/angular/angular-route.min',
    'app': 'app'
  },
  shim: {
  	'angular-route': {
        deps: ['angular']
    },
    'app': {
        deps: ['jquery', 'angular', 'angular-route']
    }
  }
});

define(['./routes'], function () {

  // 启动ng
  angular.bootstrap(document, ['app']);

});

```

上面的`define`的时候依赖了`routes`文件

> routes.js

```js
define([
  './app'
], function (app) {
  // 通过返回app主模块来定义配置
  return app.config(['$routeProvider', function ($routeProvider) {
    $routeProvider
      .when('/home',
        {
          templateUrl: '/app/partials/home.html',
          controller: 'homeController'
        })
      .otherwise(
        {
          redirectTo: '/home'
        });
    
  }]);
});

```

上面的路由文件依赖`app.js`

> app.js

```js

define([
  './controllers/index',
  './services/index'
], function (controllers, index) {
  
  // 因为主模块依赖控制器与服务模块
  // 上面的两个index 依赖分别为控制器与服务模块的两个入口
  return angular.module('app', [
    'ngRoute',
    'app.controllers',
    'app.services'
  ]);
});

```

下面先看看控制器的入口文件

> controllers/index.js

```js

define([
  './homeController'
], function () {
    // 此处为空,这个文件主要是控制当控制器比较多时可以多加载完毕
});

```

再来看看控制器代码

> controllers/homeController.js

```js

define([
  './module'
], function (module) {

  module.controller('homeController', ['$scope', 'userService',
    function ($scope, userService) {
      $scope.title = 'Home';
      $scope.users = userService.getData();
    };
  );

});

```

下面来看看控制器主模块

> controllers/module.js

```js

define(function () {

  return angular.module('app.controllers', []);

});

```

上面定义了控制器模块,以后新增的控制器都由它来定义

然后`service`的代码跟控制器思路差不多,这里就不一一定义了.

最后总结下这些依赖关系

* “main.js” requires “routes.js”
	* “routes.js” requires “app.js”
		* “app.js” requires “controllers/index.js”
			* “controllers/index.js” requires all controllers
				* all controllers require “module.js”
					* “module.js” creates the “app.controllers” module


从上面的依赖关系可以看到,当所有的控制器与服务都加载完毕,然后创建ng主模块,加载路由配置信息,最后调用启动方法.

关于`requirejs`的代码优化可以用它提供的`r.js`来合并压缩上面定义的文件最后合成一个文件来运行.


### 总结

通过上面的比较可以看出,`requirejs`比较适合大型ng项目开发,`browserify`在小型项目上要更适合,因为它比较轻量,而且`commonjs`规范可以保证相同的功能在前后端通用.

本文参考了此篇文章: <a href="http://developer.telerik.com/featured/requiring-vs-browerifying-angular/" target="_blank">Requiring vs Browserifying Angular</a>






