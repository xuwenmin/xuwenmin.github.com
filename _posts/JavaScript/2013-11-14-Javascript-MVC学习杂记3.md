---
layout: post
category : JavaScript
tagline: ""
tags : [js,mvc]
---
{% include JB/setup %}


接着上次说，这次准备在Model类里面，增加本地存储功能，用的是`html5`中的`localStorage`,这样方便，页面刷新的时候，自动加载已经添加的数据，下面是静态页的代码。

---


```html
<!DOCTYPE HTML>
<html>
 <head>
  <title> New Document </title>
  <meta name="Generator" content="EditPlus">
  <meta name="Author" content="">
  <meta name="Keywords" content="">
  <meta name="Description" content="">
  <script src="jquery-1.8.3.js" type="text/javascript"></script>
  <script type="text/javascript" >
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
   },
   attributes:function(){
      var result={};
      for(var i in this.parent.attributes){
         var attr=this.parent.attributes[i];
         result[attr]=this[attr];
      }
      return result;
   }
});
User.extend({
  attributes:["name","age","id"],
  //保存本地数据
  savelocal:function(){
     var result=[];
     for(var i in this.records){
        var record=this.records[i].attributes();
        result.push(record);
     }
     localStorage.setItem("Users",JSON.stringify(result));
  },
  //加载本地数据
  loadlocal:function(){
     if (localStorage.getItem("Users")){
         alert(localStorage.getItem("Users"));
         var result=JSON.parse(localStorage.getItem("Users"));
         var html=[];
         for(var i in result){
           html.push("<tr>");
           for(var key in this.attributes){
              var attr=this.attributes[key];
              html.push("<td>");
              html.push(result[i][attr]);
              html.push("</td>");
           }
           html.push("</tr>");
         }
         $("#tabinfo tbody").append(html.join(''));
     }
  }
});
$(function(){
    $(window).bind("beforeunload",function(){
       User.savelocal();//页面关闭的时候保存数据
    });
    $("#butadd").click(function(){
       var name=$("#txtname").val();
       var id=$("#txtid").val();
       var age=$("#txtage").val();
       var user=User.initattr({"name":name,"age":age,"id":id});
       user.save();
       var html=[];
       html.push("<tr>");
       for(var key in User.attributes){
              var attr=User.attributes[key];
              html.push("<td>");
              html.push(user[attr]);
              html.push("</td>");
       }
      html.push("</tr>");
      console.log(user);
      $("#tabinfo tbody").append(html.join(''));
    });
    //初始化的时候从本地数据里加载内容
    User.loadlocal();
});
  </script>
 </head>
  
 <body>
    <p>
      编号:<input type="textbox" id="txtid" style="margin:5px;padding:5px;width:200px;" />
      姓名:<input type="textbox" id="txtname" style="margin:5px;padding:5px;width:200px;" />
      年龄:<input type="textbox" id="txtage" style="margin:5px;padding:5px;width:200px;" />
      <input type="button" id="butadd" style="margin:5px;padding:5px;width:200px;" value="添加" />
    </p>
    <p>
      <table style="border:1px solid green" id="tabinfo">
         <thead>
            <th>编号</th>
            <th>姓名</th>
            <th>年龄</th>
         </thead>
         <tbody>
            
         </tbody>
      </table>
    </p>
 </body>
</html>
```

 将上面的文件保存为index.html,然后可以在支持html5的浏览器里运行，比如chrome，ff等。还有一个jquery1.8.3的js库，此处可以换成jquery的CDN，这是官方提供的CDN地址<http://code.jquery.com/jquery-1.8.3.min.js>

 好了，今天就写到这了，下次continue :)

