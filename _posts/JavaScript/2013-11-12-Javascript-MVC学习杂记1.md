## Javascript  MVC学习杂记1

这两天看了下<基于MVC的Javascript 富应用开发>,感觉刚开始讲的那个model类，比较有趣，所以自己就造了一个轮子，体会了下，当然也参考了点代码，见下面的代码

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

var User=Model.create();
//类方法
User.extend({
   initattr:function(o){
      var obj=this.init();
      for(var key in o){
        obj[key]=o[key];
      }
      return obj;
   }
});
//类实例方法
User.include({
   getname:function(){
      console.log(this.name);
   }
});
var user=User.initattr({"name":"xuwm","age":12});
user.getname();
```

把上面的代码保存到`demo.js`文件里，然后可以直接在NODE里运行，命令行模式下，`node demo.js`