---
layout: post
category : JavaScript
tagline: ""
tags : [Backbone,React]
---
{% include JB/setup %}

曾经用`backbone`做了几个项目,对它里面的视图功能真是不敢恭维,基本上用它作为前端`mvc`框架的都是为了兼容性,不过随着`M$`对`xp`的放弃,以及`win7`未来的不支持,`IE8`已经成了不少公司兼容IE的底限了,所以这时候假如还用`Backbone`的话,确实有点对不起住自己,不过有些时候不是你想用什么就用什么的,所以这时候你就要想办法去提高`backbone`视图功能,这时候该是`react`出马的时候了,

### React概览

`react`首先它是一个`ui`框架,用来创建独立的前端UI框架用的,类似于`angularjs`里的`指令`以及`polymer`里的`web组件`,为什么说它能够提高`backbone`视图功能呢,答案就在于它的`独立性`以及`数据同步`功能,类似于`ng`里的数据双向绑定,`react`基本`state`也提供了类似的数据跟视图的同步功能.

`react`提供了一个叫做`JSX`语法的`UI`输写规范,相当于在`js`里直接写`html`,而且不用包装成字符串,相当于`js`与`模板语言`的结合体,所以这个在运行的时候是需要转换成真实可调用的`js`的,先看个例子

* 带JSX语法的代码

```js

/** @jsx React.DOM */
var component = React.createClass({
  render: function() {
    return <a href="http://venmo.com">Venmo</a>
  }
});

```

* 转换后的代码

```js

/** @jsx React.DOM */
var component = React.createClass({
  render: function() {
    return React.DOM.a( {href:"http://venmo.com"}, "Venmo")
  }
});

```

而且在前端构建里去整合这些转换工具也是非常方便的,目前已经有下面的npm模块:

* <a href="https://github.com/seiffert/require-jsx" target="_blank">require-jsx</a>,这是一个`RequireJS`插件用来加载`.jsx`后缀文件的.

* <a href="https://github.com/andreypopp/reactify" target="_blank">reactify</a>,这是一个用在`Browserify`构建时候的`jsx`转换模块.

* <a href="https://github.com/ericclemmons/grunt-react" target="_blank">grunt-react</a>,这是一个用在`grunt`构建时的一个`grunt`插件.

不过`JSX`语法是可选的,本身提供了`React.DOM`对象来支持各种`html`元素的构建,对`react`不熟悉的话,可以看这个<a href="http://facebook.github.io/react/docs/tutorial.html" target="_blank">教程</a>

### 渲染React组件到Backbone视图中

我们先创建一个简单的组件,功能就是单击里面的链接然后去触发定义的一个方法,代码如下:

```js

var MyWidget = React.createClass({
  handleClick: function() {
    alert('Hello!');
  },
  render: function() {
    return (
      <a href="#" onClick={this.handleClick}>Do something!</a>
    );
  }
});

```

然后我们添加一个`backbone`视图来渲染它.

```js

var MyView = Backbone.View.extend({
  el: 'body',
  template: '<div class="widget-container"></div>',
  render: function() {
    this.$el.html(this.template);
    React.renderComponent(new MyWidget(), this.$('.widget-container').get(0));
    return this;
  }
});

new MyView().render();

```

下面是`jsfiddle`上面运行的例子,可以点击里面的链接看看效果.

<iframe width="100%" height="250" src="http://jsfiddle.net/4vF2r/2/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


### React组件与Backbone通信

上面的例子只是单方面的调用,这里说下双方的交互,这才是实际项目中出现的场景.下面演示一个,在`react`组件里单击,然后在`backbone`视图里增加一个元素。

先定义一个`react`组件

```js

var MyWidget = React.createClass({
  render: function() {
    return (
      <a href="#" onClick={this.props.handleClick}>Do something!</a>
    );
  }
});

```

注意这里的`this.props`是用来获取元素实例里的属性用的，然后我们来定义一个`backbone`视图

```js

var MyView = Backbone.View.extend({
  el: 'body',
  template: '<div class="widget-container"></div>' +
            '<div class="outside-container"></div>',
  render: function() {
    this.$el.html(this.template);

    React.renderComponent(new MyWidget({
      handleClick: this.clickHandler.bind(this)
    }), this.$('.widget-container').get(0));

    return this;
  },
  clickHandler: function() {
    this.$(".outside-container").html("The link was clicked!");
  }
});

```

在上面的代码里通过`new`一个`react`组件,然后传递一个事件参数,利用`bind`方法把事件指向了视图实例中去,从而实现交互

来看看在`jsfiddle`上面的效果

<iframe width="100%" height="300" src="http://jsfiddle.net/MgUXK/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

通过上面的`bind`方法实现了`react`组件与`backbone`视图的交互,下面来看看`backbone`是怎么跟`react`交互的

### Backbone与React组件通信

上面已经了解`react`组件与`backbone`视图的通信,这里讲下怎么利用`backbone`中的`model`与组件的通信,当`model`改变的时候怎么同步修改组件呢,下面我先定义一个简单的`model`

```js

var ExampleModel = Backbone.Model.extend({
  defaults: {
    name: 'Backbone.View'
  }
});

```

然后我们定义两个视图,这里直接使用`react`组件来代替`backbone`里的视图,一个视图用来触发修改`model`，一个同步显示修改的`model`

```js
// 这个视图用来同步显示修改的model属性
var DisplayView = React.createClass({
  componentDidMount: function() {
    this.props.model.on('change', function() {
      this.forceUpdate();
    }.bind(this));
  },

  render: function() {
    return (
      <p>
        {this.props.model.get('name')}
      </p>
    );
  }
});

```

下面的视图组件是用来触发`model`的修改

```js

var ToggleView = React.createClass({
  handleClick: function() {
    this.props.model.set('name', 'React');
  },
  render: function() {
    return (
      <button onClick={this.handleClick}>
        model.set('name', 'React');
      </button>
    );
  }
});

```

最后我们来渲染这两组件,并传递`model`为这两件组件的属性

```js

var model = new ExampleModel();

React.renderComponent((
  <div>
    <DisplayView model={model} />
    <ToggleView model={model} />
  </div>
), document.body);

```
 
注意下这里的`this.forceUpdate`方法,它是保证重新`render`组件用的,`componentDidMount`方法是组件初始化的时候会执行,这里负责绑定监听`model`改变事件的地方.

下面是在`jsfiddle`上面运行的效果,可以点击看看

<iframe width="100%" height="300" src="http://jsfiddle.net/c76Un/3/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

###  总结

`Backbone`与`React`是一对极好的搭档,下面是一些其它关于这方面的blog,有兴趣的可以看看.

* <a href="http://joelburget.com/backbone-to-react/" target="_blank">Joel Burget discusses Khan Academy's move to React</a>

* Paul Seiffert (`require-jsx`的作者) 写了一篇关于<a href="http://blog.mayflower.de/3937-Backbone-React.html" target="_blank">replacing Backbone views with React</a>

`React`官方也提供了一个与`Backbone`结合的`Todo`例子,<a href="https://github.com/facebook/react/tree/master/examples/todomvc-backbone" target="_blank">an example Backbone+React TodoMVC app</a>

虽然`React`阵营不是很成熟,但是它是一个令人兴奋的库,而且还可以投入生死环境,非常适合用来构建重用的组件,与`Backbone`的结合更是能够增强视图方面的业务能力.







