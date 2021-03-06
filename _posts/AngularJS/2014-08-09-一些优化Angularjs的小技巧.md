---
layout: post
category : AngularJS
tagline: ""
tags : [ng,性能优化]
---
{% include JB/setup %}

关于优化`ng`的手段网上已经有很多了,核心都是从`$$watchers`这个作用域内部属性说起的,今天我来说点别的,本质还是不变的,因为这是`ng`的硬伤,不过我相信只要运用合适的手法,这些问题还是可以避免的.

本文参考老外的一篇blog, <a href="http://www.binpress.com/tutorial/speeding-up-angular-js-with-simple-optimizations/135" target="_blank">---></a>.

---

### ng简介

`angularjs`简称`ng`,是`google`出品的`mvvm`框架,此在提高前端项目开发效率(实践中来看确实很能提高开发效率),以控制器,指令,服务来围绕整个项目,内部以独有的`DI`来解决三层之前的调用问题.更多的详情信息可以参考我之前写的`ng`源码分析.

### ng的硬伤

说到硬伤就要先说下它的简单的数据绑定原理

`ng`里每个页面上定义的`model`其实都会在当前作用域下添加一个监听器,内部容器就是`$$wachers`数组,只要页面任何一个`model`发生变化了,就会触发作用域内部`$digest`方法,它会依次查找当前作用域树里的所有`model`,是保证页面上的模型能得到数据同步,所以这个是非常消耗程序时间的,官方的说法就是当页面上出现`2000`个监听器时,页面性能就会明显下降.所以要提高`ng`的性能,就要从这方面入手了.

### tip1: 一次绑定

其实这个网上别人已经说过了,这里说下新的用法,`ng`的`1.3.0+`的版本已经内置提供了一个语法来支持模型只绑定一次的情况,看下面的例子

* old code

```html

hello {{name}}

```

* new code

```html

hello {{ ::name}}

```

可以看到新的语法就是在`model`前面加上`::`,相信这个语法要比网上用的第三方插件要方便的多了.

### tip2: $scope.$digest vs $scope.$apply

相信很多人对`$apply`方法不陌生,它一般用在,当不在`ng`的环境里执行代码的时候,为了保证页面的数据同步,在代码执行完成之后调用`$apply`方法会触发内部`$digest`来检查作用域里所有的模型,其实在它的内部是这样调用的,下面只写出一些代码片段

```js

...
...
$rootScope.$digest
...
...

```

所有它其实是调用`$rootScope`根作用域下的`$digest`,那么不同作用域下的`$digest`有什么区别呢?其实最重要的区别就在于

> `$digest` 只深度查找调用方下面所有的模型

所以`$scope`跟`$rootScope`相比,要节省掉很多查找模型的时间.

不过想要保证页面上所有模型数据的同步,还是得调用`$rootScope`的,所以在写代码之前最好想想哪些数据是要同步变化的.

### tip3: 尽可能少调用 ng-repeat

`ng-repeat`默认会创建很多监听器,所以在数据量很大的时候,这个非常消耗页面性能,我觉的只有在当需要经常更新数据列表的时候才需要用`ng-repeat`,要不然放那么多的监听器在那里也是浪费,这时候可以用`ng`自带的`$interpolate`服务来解析一个代码片段,类似于一个静态模板引擎,它的内部主要依赖`ng`核心解析服务`$parse`,然后把这些填充数据之后的代码片段直接赋给一个一次性的模型性就可以.

### tip4: 尽量在指令里写原生语法

虽然`ng`提供了很多的指令比如`ng-show`,`ng-hide`,其实它们作用就是根据模型的`true,false`来显示或隐藏一个代码片段,虽然能够很快速的实现业务要求,但是这些指令还是默认会添加监听器,假如这些代码片段存在于一个大的指令里面时,更好的方法是在指令`link`里编写`.show(), .hide()`这些类似的`jq`方法来控制比较好,这样可以节省监听器的数量,类似的还有自带的事件指令,这些其实都可以在外围指令里通过使用`addEventListener`来绑定事件,反正在写代码之前,最好想想怎么样来使监听器的数量最少,这样才能全面的提高页面性能.


### tip5: 页面内尽量少用filters

当在页面内的模型后面增加`filter`时,这个会造成当前模型在`$digest`里运行两次,造成不必要的性能浪费.第一次在`$$watchers`检测任务改变时;第二次发生在模型值修改时,所以尽量少用内联时的过滤器语法,像下面这样的非常影响页面性能

```html

{{ filter_expression | filter : expression : comparator }}

```

推荐使用`$filter`服务来调用某个过滤器函数在后台,这样能明显的提高页面性能,代码如下

```js

$filter('filter')(array, expression, comparator);

```

### 总结

上面都是些提高`ng`项目性能的一些小技巧,虽然`ng`很强大,但是不规范的代码也会破坏它的性能,所以在写代码之前最好构思下哪些地方是不需要监听器的.



