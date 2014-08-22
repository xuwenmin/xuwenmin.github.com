---
layout: post
category : JavaScript
tagline: ""
tags : [js基础]
---
{% include JB/setup %}


下面是摘至`<Javascript 高级程序设计第三版>`里的一段话

是关于对象转换数字值的一些规则

`"在应用于对象时，先调用对象的valueOf()方法以取得一个可供操作的值。然后对该值应用前述规则。如果结果是NaN,则在调用toString()方法后再应用前述规则...."`

通过上面的描述，我们知道，当需要把对象转换成数字值时，先调用`valueOf`方法，假如返回NaN,则再调用对象的`toString`方法。

所以写了下面的测试代码.

```js
var a={
    valueOf:function(){
        return "admin";
    },
    toString:function(){
        return "2";
    }
}

var b={
    toString:function(){
        return "2";
    }
}

var c={
    valueOf:function(){
        return "4";
    }
}

console.log(+a); // print NaN
console.log(+b); // print 2
console.log(+c); // print 4
```

经测试发现，当valueOf和toString方法同时存在的时候，只会按valueOf的返回值来转换数字值，哪怕toString方法可以返回数字，结果也是NaN.

测试的浏览器信息为

    Google Chrome   31.0.1650.63 (正式版本 238485) m
    操作系统        Windows 
    Blink           537.36 (@163124)
    JavaScript      V8 3.21.18.13

 
不知道这是不是chrome的一个改进，还是什么，特记录下来，方便给别人参考。

 

