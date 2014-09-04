---
layout: post
category : AngularJS
tagline: ""
tags : [ng,指令,compile,link]
---
{% include JB/setup %}

通常大家在使用`ng`中的指令的时候,用的链接函数最多的是`link`属性,下面这篇文章将告诉大家`complie`,`pre-link`,`post-link`的用法与区别.

<a href="http://www.jvandemo.com/the-nitty-gritty-of-compile-and-link-functions-inside-angularjs-directives/" target="_blank">原文地址</a>

`angularjs`里的指令非常神奇,允许你创建非常语义化以及高度重用的组件,可以理解为`web components`的先驱者.

网上已经有很多介绍怎么使用指令的文章以及相关书籍,相互比较的话,很少有介绍`compile`与`link`的区别,更别说`pre-link`与`post-link`了.

大部分教程只是简单的说下`compile`会在`ng`内部用到,而且建议大家只用`link`属性,大部分指令的例子里都是这样的

这是非常不幸的,因为正确的理解这些函数的区别会提高你对`ng`内部运行机理的理解,有助于你开发更好的自定义指令.

所以跟着我一起来看下面的内容一步步的去了解这些函数是什么以及它们应该在什么时候用到

* 本文假设你已经对指令有一定的了解了,如果没有的话强烈建议你看看这篇文章<a href="https://docs.angularjs.org/guide/directive" target="_blank">AngularJS developer guide section on directives</a>

### NG中是怎么样处理指令的

开始分析之前,先让我们看看`ng`中是怎么样处理指令的.

当浏览器渲染一个页面时,本质上是读`html`标识,然后建立`dom`节点,当`dom`树创建完毕之后广播一个事件给我们.

当你在页面中使用`script`标签加载`ng`应用程序代码时,`ng`监听上面的`dom`完成事件,查找带有`ng-app`属性的元素.

当找到这样的元素之后,`ng`开始处理`dom`以这个元素的起点,所以假如`ng-app`被添加到`html`元素上,则`ng`就会从`html`元素开始处理`dom`.

从这个起点开始,`ng`开始递归查找所有子元素里面,符合应用程序里定义好的指令规则.

`ng`怎样处理指令其实是依赖于它定义时的对象属性的,你可以定义一个`compile`或者一个`link`函数,或者用`pre-link`和`post-link`函数来代替`link`.

所以这些函数的区别呢?为什么要使用它?以及什么时候使用它呢?

带着这些问题跟着我一步一步来解答这些迷团吧

### 一段代码

为了解释这些函数的区别,下面我将使用一个简单易懂的例子

* 如果您有任何的问题,请不要犹豫赶紧在下面加上你的评论吧.

看看下面一段`html`标签代码

```html

	<level-one>
	    <level-two>
	        <level-three>
	            Hello {{name}}
	        </level-three>
	    </level-two>
	</level-one>

```

然后是一段`js`代码

```js

	var app = angular.module('plunker', []);

	function createDirective(name){
	  return function(){
	    return {
	      restrict: 'E',
	      compile: function(tElem, tAttrs){
	        console.log(name + ': compile');
	        return {
	          pre: function(scope, iElem, iAttrs){
	            console.log(name + ': pre link');
	          },
	          post: function(scope, iElem, iAttrs){
	            console.log(name + ': post link');
	          }
	        }
	      }
	    }
	  }
	}

	app.directive('levelOne', createDirective('levelOne'));
	app.directive('levelTwo', createDirective('levelTwo'));
	app.directive('levelThree', createDirective('levelThree'));

```

结果非常简单:让`ng`来处理三个嵌套指令,并且每个指令都有自己的`complile`,`pre-link`,`post-link`函数,每个函数都会在控制台里打印一行东西来标识自己.

这个例子能够让我们简单的了解到`ng`在处理指令时,内部的流程

### 代码输出

下面是一个在控制台输出结果的截图

<img src="http://www.jvandemo.com/content/images/2014/Aug/console-output-1.png" alt="result output">

如果想自己试一下这个例子的话,请点击<a href="http://plnkr.co/edit/5rbeFfKO7QUM2PGkK3Qy?p=preview" target="_blank">this plnkr</a>,然后在控制台查看结果.

### 分析代码

第一个要注意的是这些函数的调用顺序:

```js

	// COMPILE PHASE
	// levelOne:    compile function is called
	// levelTwo:    compile function is called
	// levelThree:  compile function is called

	// PRE-LINK PHASE
	// levelOne:    pre link function is called
	// levelTwo:    pre link function is called
	// levelThree:  pre link function is called

	// POST-LINK PHASE (Notice the reverse order)
	// levelThree:  post link function is called
	// levelTwo:    post link function is called
	// levelOne:    post link function is called

```

这个例子清晰的显示出了`ng`在`link`之前编译所有的指令,然后`link`要又分为了`pre-link`与`post-link`阶段.

注意下,`compile`与`pre-link`的执行顺序是依次执行的,但是`post-link`正好相反.

所以上面已经明确标识出了不同的阶段,但是`compile`与`pre-link`有什么区别呢,都是相同的执行顺序,为什么还要分成两个不同的函数呢?

### DOM

为了挖的更深一点,让我们简单的修改一下上面的代码,它也会在各个函数里打印参数列表中的`element`变量

```js

	var app = angular.module('plunker', []);

	function createDirective(name){ 
	  return function(){
	    return {
	      restrict: 'E',
	      compile: function(tElem, tAttrs){
	        console.log(name + ': compile => ' + tElem.html());
	        return {
	          pre: function(scope, iElem, iAttrs){
	            console.log(name + ': pre link => ' + iElem.html());
	          },
	          post: function(scope, iElem, iAttrs){
	            console.log(name + ': post link => ' + iElem.html());
	          }
	        }
	      }
	    }
	  }
	}

	app.directive('levelOne', createDirective('levelOne'));
	app.directive('levelTwo', createDirective('levelTwo'));
	app.directive('levelThree', createDirective('levelThree'));

```

注意下`console.log`里的输出,除了输出原始的`html`标记基本没别的改变.

这个应该能够加深我们对于这些函数上下文的理解.

再次运行代码看看

### 输出

下面是一个在控制台输出结果的截图

<img src="http://www.jvandemo.com/content/images/2014/Aug/console-output.png" alt="">

假如你还想自己运行看看效果,可以点击<a href="http://plnkr.co/edit/KtMs0H1pBsrOmXrFh9nf" target="_blank">this plnkr</a>,然后在控制台里查看输出结果.

### 观察

输出`dom`的结果可以暴露一些有趣的事情:`dom`内容在`compile`与`pre-link`两个函数中是不一样的

### 所以发生了什么呢?

#### Compile

我们已经知道当`ng`发现`dom`构建完成时就开始处理`dom`.

所以当`ng`在遍历`dom`的时候,碰到`level-one`元素,从它的定义那里了解到,要执行一些必要的函数

因为`compile`函数定义在`level-one`指令的指令对象里,所以它会被调用并传递一个`element`对象作为它的参数

如果你仔细观察,就会看到,浏览器创建这个`element`对象时,仍然是最原始的`html`标记

* 在`ng`中,原始`dom`通常用来标识`template element`,所以我在定义`compile`函数参数时就用到了`tElem`名字,这个变量指向的就是`template element`.

一旦运行`levelone`指令中的`compile`函数,`ng`就会递归深度遍历它的`dom`节点,然后在`level-two`与`level-three`上面重复这些操作.

#### Post-link

深入了解`pre-link`函数之前,让我们来看看`post-link`函数.

* 如果你在定义指令的时候只使用了一个`link`函数,那么`ng`会把这个函数当成`post-link`来处理,因此我们要先讨论这个函数

当`ng`遍历完所有的`dom`并运行完所有的`compile`函数之后,就反向调用相关联的`post-link`函数.

`dom`现在开始反向,并执行`post-link`函数,因此,在之前这种反向的调用看起来有点奇怪,其实这样做是非常有意义的.

<img src="http://www.jvandemo.com/content/images/2014/Aug/cycle.png" alt="">

当运行包含子指令的指令`post-link`时,反向的`post-link`规则可以保证它的子指令的`post-link`是已经运行过的.

所以,当运行`level-one`指令的`post-link`函数的时候,我们能够保证`level-two`和`level-three`的`post-link`其实都已经运行过了.

这就是为什么人们都认为`post-link`是最安全或者默认的写业务逻辑的地方.

但是为什么这里的`element`跟`compile`里的又不同呢?

一旦`ng`调用过指令的`compile`函数,就会创建一个`template element`的`element`实例对象,并且为它提供一个`scope`对象,这个`scope`有可能是新实例,也有可能是已经存在,可能是个子`scope`,也有可能是独立的`scope`,这些都得依赖指令定义对象里的`scope`属性值

所以当`linking`发生时,这个实例`element`以及`scope`对象已经是可用的了,并且被`ng`作为参数传递到`post-link`函数的参数列表中去.

* 我个人总是使用`iElem`名称来定义一个`link`函数的参数,并且它是指向`element`实例的

所以`post-link`(`pre-link`)函数的`element`参数对象是一个`element`实例而不是一个`template element`.

所以上面例子里的输出是不同的

#### Pre-link

当写了一个`post-link`函数,你可以保证在执行`post-link`函数的时候,它的所有子级指令的`post-link`函数是已经执行过的.

在大部分的情况下,它都可以做的更好,因此通常我们都会使用它来编写指令代码.

然而,`ng`为我们提供了一个附加的`hook`机制,那就是`pre-link`函数,它能够保证在执行所有子指令的`post-link`函数之前.运行一些别的代码.

这句话是值得反复推敲的

`pre-link`函数能够保证在`element`实例上以及它的所有子指令的`post-link`运行之前执行.

所以它使的`post-link`函数反向执行是相当有意义的,它自己是原始的顺序执行`pre-link`函数

这也意为着`pre-link`函数运行在它所有子指令的`pre-link`函数之前,所以完整的理由就是:

一个元素的`pre-link`函数能够保证是运行在它所有的子指令的`post-link`与`pre-link`运行之前执行的.见下图

<img src="http://www.jvandemo.com/content/images/2014/Aug/cycle-2.png" alt="">

#### 回顾

如果我们回头看看上面原始的输出,就能清楚的认出到底发生了什么:

```js

	// HERE THE ELEMENTS ARE STILL THE ORIGINAL TEMPLATE ELEMENTS

	// COMPILE PHASE
	// levelOne:    compile function is called on original DOM
	// levelTwo:    compile function is called on original DOM
	// levelThree:  compile function is called on original DOM

	// AS OF HERE, THE ELEMENTS HAVE BEEN INSTANTIATED AND
	// ARE BOUND TO A SCOPE
	// (E.G. NG-REPEAT WOULD HAVE MULTIPLE INSTANCES)

	// PRE-LINK PHASE
	// levelOne:    pre link function is called on element instance
	// levelTwo:    pre link function is called on element instance
	// levelThree:  pre link function is called on element instance

	// POST-LINK PHASE (Notice the reverse order)
	// levelThree:  post link function is called on element instance
	// levelTwo:    post link function is called on element instance
	// levelOne:    post link function is called on element instance

```

### 概要

回顾上面的分析我们可以描述一下这些函数的区别以及使用情况:

#### Compile 函数

使用`compile`函数可以改变原始的`dom`(`template element`),在`ng`创建原始`dom`实例以及创建`scope`实例之前.

可以应用于当需要生成多个`element`实例,只有一个`template element`的情况,`ng-repeat`就是一个最好的例子,它就在是`compile`函数阶段改变原始的`dom`生成多个原始`dom`节点,然后每个又生成`element`实例.因为`compile`只会运行一次,所以当你需要生成多个`element`实例的时候是可以提高性能的.

`template element`以及相关的属性是做为参数传递给`compile`函数的,不过这时候`scope`是不能用的:

下面是函数样子

```js

	/**
	* Compile function
	* 
	* @param tElem - template element
	* @param tAttrs - attributes of the template element
	*/
	function(tElem, tAttrs){

	    // ...

	};

```

#### Pre-link 函数

使用`pre-link`函数可以运行一些业务代码在`ng`执行完`compile`函数之后,但是在它所有子指令的`post-link`函数将要执行之前.

`scope`对象以及`element`实例将会做为参数传递给`pre-link`函数:

下面是函数样子

```js

	/**
	* Pre-link function
	* 
	* @param scope - scope associated with this istance
	* @param iElem - instance element
	* @param iAttrs - attributes of the instance element
	*/
	function(scope, iElem, iAttrs){

	    // ...

	};

```

#### Post-link 函数

使用`post-link`函数来执行业务逻辑,在这个阶段,它已经知道它所有的子指令已经编译完成并且`pre-link`以及`post-link`函数已经执行完成.

这就是被认为是最安全以及默认的编写业务逻辑代码的原因.

`scope`实例以及`element`实例做为参数传递给`post-link`函数:

下面是函数样子

```js

	/**
	* Post-link function
	* 
	* @param scope - scope associated with this istance
	* @param iElem - instance element
	* @param iAttrs - attributes of the instance element
	*/
	function(scope, iElem, iAttrs){

	    // ...

	};

```

### 总结

现在你应该对`compile`,`pre-link`,`post-link`这此函数之间的区别有了清晰的认识了吧.

如果还没有的话,并且你是一个认真的`ng`开发者,那么我强烈建议你重新把这篇文章读一读直到你了解为止

理解这些概念非常重要,能够帮助你理解`ng`原生指令的工作原理,也能帮你优化你自己的自定义指令.

如果还有问题的话,欢迎大家在下面评论里加上你的问题

以后还会接着分析关于指令里的其它两个问题:

* 指令使用`transclusion`属性是怎么工作的?

* 指令的`controller`函数是怎么关联的

最后,如果发现本文哪里有不对的,请及时给我发评论

谢谢!















