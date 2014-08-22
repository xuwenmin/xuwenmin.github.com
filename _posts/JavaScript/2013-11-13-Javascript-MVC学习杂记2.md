---
layout: post
category : JavaScript
tagline: ""
tags : [js,mvc]
---
{% include JB/setup %}

继续Javascript MVC 学习的探索，上次说到一个Model类，负责创建实际类，以及类实例化，这次接着添加ORM元素，即对象持久化特征。代码如下

---

```js
//基于原型的继承
if(typeof Object.create!=="function"){
   Object.create=function(o){
      function F(){}
      F.prototype=o;
      return new F();
   }
}
var Model={
   prototype:{
      init:function(){
        console.log('Model.prototype.init');
      },
      find:function(){
        console.log('Model.prototype.find');
      }
   },
   inherited:function(){
      //console.log('exec inherited')
   },
   created:function(){
      //console.log('exec created!');
   },
   create:function(){
      var object=Object.create(this);
      object.parent=this;
      object.prototype=object.fn=Object.create(this.prototype);

      object.created();//创建完成方法

      this.inherited(object); //继承父类属性方法

      return object;
   },
   init:function(){
      var instance=Object.create(this.prototype);
      instance.parent=this;
      instance.init.apply(instance,arguments);
      return instance;
   },
   extend:function(o){
      for(var key in o){
        this[key]=o[key];
      }
   },
   include:function(o){
      for(var key in o){
         this.prototype[key]=o[key];
      }
   }
};
//类方法
Model.extend({
   records:{},
   initattr:function(o){
      var obj=this.init();
      for(var key in o){
        obj[key]=o[key];
      }
      return obj;
   },
   find:function(id){
      return this.records[id];
   }
});
Model.include({
   save:function(){
      this.parent.records[this.id]=this;
   }
});
var User=Model.create();
//类实例方法
User.include({
   getname:function(){
      console.log(this.name);
   }
});
var user=User.initattr({"name":"xuwm","age":12,"id":1});
user.save();
var u=User.find(1);
u.getname();
```

跟上次不一样的地方是，添加了一个`records`对象，负责保存创建的类实例，还有一个根据实例ID属性查询的find类方法。

实例运行跟上次一样，保存代码为`demo.js` ,命令行切换到`NODE`的目录，输入`node demo.js`,即可看到结果。

一点心得，尽当以后温故:)