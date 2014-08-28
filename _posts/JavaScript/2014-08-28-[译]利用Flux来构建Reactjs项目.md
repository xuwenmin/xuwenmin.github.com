---
layout: post
category : JavaScript
tagline: ""
tags : [react,flux]
---
{% include JB/setup %}

<a href="http://christianalfoni.github.io/javascript/2014/08/20/react-js-and-flux.html" target="_blank">原文地址</a>

首先我不想浪费大家的时间在`flux`的细节上,关于它的详情,可以点击<a href="http://facebook.github.io/flux/" target="_blank">Facebook flux site</a>,我想告诉大家的是,利用`flux`与`react`的结合可以更好的处理业务交互以及视图同步.

### 一切源于状态

在同一个页面上,状态可以理解为它上面的一个`checkbox`,随着`checkbox`显中或者不显中,状态都会不一样,不用去关心状态改变时要做什么,而是去关心状态改变跟UI交互的方式.


### 一个小故事

我现在在构建一个大型的电话交换机的web项目,它起始于我在<a href="http://www.marcello.no/" target="_blank">Marcello</a>的工作,那时候像`backbone`与`angularjs`都没出现,为了实现在浏览器上面显示每个电话交换机用户的调用,系统是以事件为基础,意为着没有保存每次调用的状态,但是可以从每个调用的服务端状态改变那里得到独立的事件.

长话短说,依赖这些事件来改变dom,每个事件类型对应一个dom里的操作,目前来看这似乎不是一个坏主意,但是每个调用事件有20-30个状态,比如播号,挂断等,这些慢慢让项目扩展与维护失去控制,而且问题也越来越多.

后来我重构了整个项目,那时候`backbone`还不错,所以就用它来代替之前的单一事件框架,每次调用就传递完整的状态,从服务端传递的调用状态对应到集合中的模型,然后传递到视图中去,每当集合改变的时候,就会重新渲染视图来同步状态,这时已经没什么问题了.

关于这个故事有两点需要注意下:

* 渲染之前拥有完整的状态要比在数据位上构建状态要容易的多

* 渲染是一个不错的概念,虽然还有些问题,不过非常容易来处理应用程序逻辑

下面来看看不同框架对状态改变跟UI的交互方式,其中的输写规范统一是`cjs`(commonjs)规范,后续可以利用`browserify`来部署


### Backbone

* main.js 应用核心JS

```js

var UserModel = require('./UserModel.js');
var CheckboxView = require('./CheckboxView.js');
new CheckboxView({model: new UserModel()}).render().$el.appendTo('body');

```

* UserModel.js 模型js,存储状态信息

```js

var model = Backbone.Model.extend({
  defaults: {
    notify: false
  }
});

```

* CheckboxView.js 视图信息,模型的交互对象

```js

var View = Backbone.View.extend({
  events: {
    'click input': 'updateUser',
    template: handlebars.compile('<input type="checkbox" {#if notify}checked{/if}/> Notify'),
    initialize: function () {
      this.listenTo(this.model, 'change', this.render);
    },
    render: function () {
      this.$el.html(this.template(this.model.toJSON()));
      return this;
    },
    updateUser: function () {
      this.model.set('notify', this.$el.find('input').is(':checked'));
    }
  }
});

```

上面的脚本传递一个模型给定义好的视图,视图利用`handlebars`来解析模板,初始化的时候添加对模型的监听,功能就是当模型改变的时候可以渲染视图,所以当触发视图里的dom事件时,利用监听关系也可以同步视图,从而达到模型状态的改变与UI视图的同步.

### 优点

`backbone`让视图更新变的非常容易,任何UI的交互都可以更新模型,而模型更新又可以影响视图,从而脱离利用原生`dom`操作来更新UI,一切是以模型来做为主角

### 缺点

`backbone`渲染的时候都是更新整个视图,这并不是一个好的功能,因为`dom`渲染更多的节点是需要花费很多时间的,而且整个视图渲染有可能影响交互,比如当在输入框里输入东西的时候.

这里也有一个扩展方面的问题,给视图更多修改模型的能力并不是一个好的主意,有可能改变某个状态会影响到应用程序其它部分.有时候需要销毁绑定在模型上的多个监听,其中每个监听用来处理不同的应用程序状态,这个也会潜在的存在管理问题.

### Angularjs

* main.js 应用核心JS

```js

angular.module('myApp', [])
.factory('UserService', function () {
  var user = {};
  return {
    getUser: function () {
      return user;
    }
  }
})
.controller('MyCtrl', function ($scope, UserService) {
  $scope.user = UserService.getUser();
});

```

* index.html 应用程序首页中的一部分

```html

<body ng-app="myApp">
  <div ng-controller="MyCtrl">
    <input type="checkbox" ng-model="user.notify"/>
  </div>
</body>

```
上面的代码运用了`ng`里的双向绑定功能,这是一个强大的功能,通过注入`UserService`解决了模型的功能,仅仅在视图里`bind`一个状态就可以实现双向同步,看起来代码非常漂亮.

### 优点

`ng`里的双向绑定极大了减少了代码量,而且能够提高开发效率,个人非常喜欢它里面的原型继承.对于模型思想的实现弱化了,不仅可以包含基本的状态属性,而且也可以添加各种不同类型的行为.

### 缺点

即使我们写再少的代码也会碰到跟`backbone`相同的问题,而且更糟.在大型应用程序中,多个组件想了解属性的改变,虽然双向绑定非常强大,但是仍然没有设置属性的地方,只能在内部改变模型,你将只能通过事件或者手动监听属性来了解属性的改变,这就是跟`backbone`相同的问题,应用程序的多个部分想了解某个属性的状态改变.

`ng`的双向绑定依赖于`脏值检测`,这个贯穿整个`ng`上下文中,而且很难辨认出它是否处于`脏值检测`,而且这个过程也会影响应用程序的性能,当手动的调用`$apply`的时候也有可能产生异常.

### Flux

`flux`并不是一个框架,只是一个利用`reactjs`来作为自己控制器与视图的一个架构,这里没有`mvc, mv*`,不用去关心这是`mv`什么,只需要了解它是一个容易理解而且扩展性极好的架构就行了,就算要跟`mv`系列比较的话,那么它算是`mv*`,关于构建`flux`的时候,有三个概念需要了解下:

* Dispatcher(调度)

* Stores(存储)

* Components(组件)

虽然这三个概念没有跟上面的例子进行关联,但是它们之间有相似的地方,模型相当于`flux`里的`Stores`,在`backbone`与`angularjs`中,视图改变依赖于模型,不过在`flux`里不是这样的

在`flux`视图中是不会更新`Stores`的,而是发送一个动作,然后`Dispatcher`通知`Stores`接收这个动作,所以重点是`Stores`持有所有应用程序的状态,当一个动作接收到时,就可以产生一个服务端请求等.所以视图是利用`Dispatcher`来传递动作即而来更新`Stores`的.

当`Stores`更新它的状态之后,持有改变监听事件的视图就会重新渲染,这里的原理跟`backbone`一样,只是跟它的区别在于,一个组件不会更新它所有的视图内容,只会更新一部分.这是靠`reactjs`里的`virtual DOM`来实现的,这实在是不错.

所以整个流程是这样的, `DISPATCHER -> STORES -> COMPONENTS`,如果一个组件想改变状态,则必须要给`DISPATCHER`发送一个动作,而在典型的`MVC`中,通常是这样的,`MODEL <-> CONTROLLER <-> VIEW`,状态改变是双向的,所以想想`model`,`controller`,`view`都是双向的,相互作用的,这是造成问题的关键所在.在`flux`中,不用关心你的应用程序有多复杂,流程都是这样的,这才是它让项目变的简单的地方.

我们来看看它的代码例子

在下面的例子中会用到一个`reactjs`扩展,它包含自己的`dispatcher`,`store`,更多关于它的详情,请点击<a href="https://github.com/christianalfoni/flux-react" target='_blank'>flux-react</a>

* main.js  应用主要js文件

```js

/** @jsx React.DOM */
var React = require('flux-react');
var Checkbox = require('./Checkbox.js');
React.renderComponent(<Checkbox/>, document.body);

```

* UserStore.js 存储js,类似于模型

```js

var React = require('flux-react');
var user = {
  notify: false
};
module.exports = React.createStore({
  getNotify: function () {
    return user.notify;
  },
  
  // dispatch runs whenever a new action is received
  // from the dispatcher
  dispatch: function (payload) {
    switch (payload.type) {
      case 'CHANGE_NOTIFY':
        user.notify = payload.notify;
        this.flush(); // Give notice that changes has been done
        break;
    }
  }
});

```

* Checkbox.js 组件js,关联store与dispatcher

```js

/** @jsx React.DOM */
var React = require('flux-react');
var UserStore = require('./UserStore.js');
module.exports = React.createClass({

  // What stores the component is dependant of
  stores: [UserStore], 
  getInitialState: function () {
    return {
      notify: UserStore.getNotify() 
    };
  },
  
  // A react-flux callback that triggers when any
  // of its dependant stores updates (flushes)
  storesDidUpdate: function () {
    this.setState({
      notify: UserStore.getNotify()
    });
  },
  notify: function () {
    React.dispatch({
      type: 'CHANGE_NOTIFY',
      notify: this.refs.checkbox.getDOMNode().checked
    });
  },
  render: function () {
    return (
      <input ref="checkbox" type="checkbox" checked={this.state.notify} onChange={this.notify}/>
    )
  }
});

```

嗯,看起来代码有点多,试着这样想想,`flux`上面的例子里,假如需求改变了,有可能我们要移除很多代码,但是在`flux`的例子里,应用程序添加任何东西,都不需要移除任何代码,如果你需要一个新的`store`,只要添加它然后给依赖它的组件就可以,如果需要更多的视图,只需要在创建它并添加到使用它的组件中去,而且也不会影响当前环境内的`其它视图与模型`.

`reactjs`与`flux`目前还比较新,还需要在大型应用程序里去检验,不过我相信这个概念非常吸引人.希望这篇文章能够帮助你开始利用`reactjs`和`flux`来构建应用程序.最后感谢大家的宝贵意见!


### 参考资料

* <a href="https://github.com/christianalfoni/flux-react" target="_blank">官方github</a>

* <a href="http://facebook.github.io/flux/docs/todo-list.html#content" target="_blank">官方Todo例子</a>

* <a href="https://github.com/christianalfoni/flux-react-boilerplate" target="_blank">一个flux-react脚手架项目</a>



