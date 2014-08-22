## ES6系列之Module & Class

`Ecmascript 6`简称`es6`,是`javascript`下一代标准,还处在开发阶段,估计2014年底发布,有关更多浏览器对`es6`的支持情况,<a href="http://kangax.github.io/compat-table/es6/" target="_blank">点击这里</a>

今天说说`es6`里新增的`Module`和`Class`.

---

### Class

关于`class`其实前端已有太多的模拟了,因为js本身的弱类型决定了只要你有想法什么编程模式都可以模拟出来,跟`class`相关的`oop`模式早已在后端领域扎根了,前端`class`概念大多是通过`function`函数来实现,现在我们来看看`es6`提供了什么语法?,我们先以下例子来说明,这样比较直观

 * 提示下,下面的例子最后实现的是一个backbone里的模型与视图

```js

class Model {
	constructor(properties){
		this.properties = properties;
	}
	toObject(){
		return this.properties;
	}
}

```

可以看到`es6`原生提供了`class`关键字来定义类,提供了`constructor`来实现后端的构造函数,内部以`this`来代表当前实例

除了构造函数,普通函数除掉了`function`关键字,目前没有提供关于方法的保护属性比如`private`,`public`,`protect`,要是以后能提供这些关键字的话倒是不错

下面我们再来看看`class`里的继承,`es6`提供了`extends`来支持

```js

// 先定义一个父类,这里是视图类
class View{
	// 构造函数
	constructor(options){
		this.model = options.model;
		this.template = options.template;
	}
	render(){
		return _.template(this.template, {name: 'feenan', info: 'haha'});
	}
}

// 然后我们定义一个继承上面视图的日志子类,然后重载render方法
class LogView extends View{
	render(){
		var complied = super();
		console.log(complied);
	}
}

```

通过上面的例子我们知道通过在定义类后面加上`extends`关键字实现继承

这里重载父类方法不需要额外加任务关键字，不像后端有些语言要加上`override`

在重载方法里使用`super()`,默认会调用同名的父类方法

目前`es6`里的`class`定义只是一些基础的功能，相信以后还会增加一些别的特性,比如提高属性或者方法的封装性.


### Module

目前的前端模块规范有`CMD`,`AMD`,不过`commonjs`也可以用于前端开发只是需要工具来支持

为什么需要模块呢?大家可以看看我之前的一篇文章,<a href="http://www.ifeenan.com/~posts/JavaScript/2014-06-30-%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81web%E6%A8%A1%E5%9D%97.md" target="_blank">传送门:)</a>

模块解决的问题无非是这几点:

* 统一模块定义

* 统一模块之间的依赖调用

* 统一模块使用

我们先来看看`es6`里的模块定义

> export 导出模块定义文件里的属性或者方法

```js
// module.js

export var obj = {
	name: 'feenan',
	action: 'hello'
};

var _privateFn = function(){
	console.log('hello feenan!');
};

export function print(){
	_privateFn();
}

```

> export default 导出模块默认的属性或者方法

```js
// module1.js

export default function print(){
	console.log('hello feenan!');
}

```

像上面这样定义的模块,在使用的时候导入模块方法的时候可以省掉别名，这个下面会讲到


定义完模块之后,可以来使用它了.

> import 从模块文件中导入功能到使用的文件中

```js

// 使用export 定义的模块功能
import {obj, print} from './module'

// 使用export default 定义的模块功能
import md from './module1'

```

使用import的时候有两种情况

* 用export 定义的模块,import的时候后面一定得写上{},里面的功能名称得跟模块定义里的相匹配,否则会报错

* 用export default 定义的模块功能,import 后面不用跟上{},而且导入的名称也可以随意写,不必跟模块定义里的相同

> module 从模块文件中导出所有的功能到使用的文件中

```js

module all from  './module';

```

使用`module`关键字可以导入模块内所有支持导出的方法,后面的别名可以随意命名.


### 一个完整的例子


下面我们使用`module`和`class`来写一个例子,利用上面的`Model`与`View`,下面是完整的代码

* model.js

```js

// 定义模型数据 
export class Model {
	constructor(properties){
		this.properties = properties;
	}
	toObject(){
		return this.properties;
	}
}

```

* log.view.js  此处的父类View其实也可能做成模块导入进来

```js

// 导入underscore
import { underscore } from './underscore';

// 获取_变量
_ = underscore._;

// 定义一个视图
class View{
	// 构造函数
	constructor(options){
		this.model = options.model;
		this.template = options.template;
	}
	render(){
		return _.template(this.template, {name: 'feenan', info: 'haha'});
	}
}

export class LogView extends View{
	// 重载render方法
	render(){
		var complied = super();
		console.log(complied);
	}
}

```

* underscore.js 这里只写个样子,因为代码太长

```js

export  var underscore = {}; 

// 下面是underscore源码部分
(function() {
	// 这里代码太长就不写了....
	// ......
}).call(underscore);

```

* main.js 这里是调用的地方

```js

import { Model } from './model';
import { LogView } from './log.view';

var m = new Model({
	name: 'feenan',
	info: 'frontender'
});
var v = new LogView({
	model: m,
	template: 'hello  <%= name%> ! <%= info%>'
});

v.render();

```

上面就是完整代码，下面说说怎么运行此例子.

### 运行

关于上面的例子可以通过安装`traceur`模块来进行测试

> npm install -g traceur

运行例子如下

> traceur main.js

### 总结

`es6`里的`Module`与`Class`统一了前端模块的调用与依赖加载,以及类的规范定义,真心觉的这个很不错,当所有浏览器都原生支持这个特性的时候,相信现在的`require.js`,`seajs`基本都用不上了,而且模块加载的性能也会上一个台阶.

