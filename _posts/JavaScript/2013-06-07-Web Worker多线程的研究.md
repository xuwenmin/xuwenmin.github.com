---
layout: post
category : JavaScript
tagline: ""
tags : [web Worker]
---
{% include JB/setup %}

HTML5支持了Web Worker这样的API，允许网页在安全的情况下执行多线程代码。不过Web Worker实际上受到很多限制，因为它无法真正意义上共享内存数据，只能通过消息来做状态通知，所以甚至不能称之为真正意义上的“多线程”。

---

Web Worker的接口使用起来很不方便，它基本上自带一个sandbox，在沙箱中跑一个独立的js文件，通过 postMessage和 onMessge来和主线程通信：

<!--more-->

```js
var worker = new Worker("my.js");
var bundle = {message:'Hello world', id:1};
worker.postMessage(bundle); //postMessage可以传一个可序列化的对象过去
worker.onmessage = function(evt){
    console.log(evt.data);    //比较worker中传回来的对象和主线程中的对象
    console.log(bundle);  //{message:'Hello world', id:1}
}
```

```js
//in my.js
onmessage = function(evt){
    var data = evt.data;
    data.id++;
    postMessage(data); //{message:'Hello world', id:2}
}
```

得到的结果可以发现，线程中得到的data的id增加了，但是传回来之后，并没有改变主线程的bundle中的id，因此，线程中传递的对象实际上copy了一份，这样的话，线程并没有共享数据，避免了读写冲突，所以是安全的。保证线程安全的代价就是限制了在线程中操作主线程对象的能力。

这样一个有限的多线程机制使用起来是很不方便的，我们当然希望Worker能够支持让代码看起来具有同时操作多线程的能力，例如，支持看起来像下面这个样子的代码：

```js
var worker = new ThreadWorker(bundle /*shared obj*/);
 
worker.run(function(bundle){
    //do sth in worker thread...
    this.runOnUiThread(function(bundle /*shared obj*/){
        //do sth in main ui thread...
    });
    //...
});
```

这段代码里面，我们启动一个worker之后，能够让任意代码跑在worker中，并且当需要操作ui线程（比如读写dom）时，可以通过this.runOnUiThread回到主线程执行。

那么如何实现这个机制呢？ 看下面的代码：

```js
function WorkerThread(sharedObj){
    this._worker = new Worker("thread.js");
    this._completes = {};
    this._task_id = 0;
    this.sharedObj = sharedObj;
 
    var self = this;
    this._worker.onmessage = function(evt){
        var ret = evt.data;
        if(ret.__UI_TASK__){
            //run on ui task
            var fn = (new Function("return "+ret.__UI_TASK__))();
            fn(ret.sharedObj);
        }else{
            self.sharedObj = ret.sharedObj;
            self._completes[ret.taskId](ret);
        }
    }
}
 
WorkerThread.prototype.run = function(task, complete){
    var _task = {__THREAD_TASK__:task.toString(), sharedObj: this.sharedObj, taskId: this._task_id};
    this._completes[this._task_id++] = complete;
    this._worker.postMessage(_task);
}
```

上面这段代码定义了一个ThreadWorker对象，这个对象创建了一个运行thread.js的Web Worker，保存了共享对象SharedObj，并且对thread.js发回的消息进行处理。

如果thread.js中传回了一个__UI_TASK__消息，那么运行这个消息传过来的function，否则执行run的complete回调
我们看看thread.js是怎么写的：

```js
onmessage = function(evt){
    var data = evt.data;
 
    if(data && data.__THREAD_TASK__){
        var task = data.__THREAD_TASK__;
        try{
            var fn = (new Function("return "+task))();
 
            var ctx = {
                threadSignal: true,
                sleep: function(interval){
                    ctx.threadSignal = false;
                    setTimeout(_run, interval);
                },
                runOnUiThread: function(task){
                    postMessage({__UI_TASK__:task.toString(), sharedObj:data.sharedObj});
                }
            }
 
            function _run(){
                ctx.threadSignal = true;
                var ret = fn.call(ctx, data.sharedObj);
                postMessage({error:null, returnValue:ret, __THREAD_TASK__:task, sharedObj:data.sharedObj, taskId: data.taskId});
            }
 
            _run(0);
 
        }catch(ex){
            postMessage({error:ex.toString() , returnValue:null, sharedObj: data.sharedObj});
        }
    }
}
```

可以看到，thread.js接收ui线程传过来的消息，其中最重要的是__THREAD_TASK__，这是ui线程传过来的需要worker线程执行的“任务”，由于function是不可序列化的，因此传递的是字符串，worker线程通过解析字符串成function来执行主线程提交的任务（注意在任务中将共享对象sharedObj传入），执行完成后将返回结果通过message传给ui线程。我们仔细看一下除了返回值returnValue以外，共享对象sharedObj也会被传回，传回时，由于worker线程和ui线程并不共享对象，因此我们人为通过赋值的方式同步两边的对象（这样是否线程安全？为什么？）

可以看到整个过程其实并不复杂，这么实现之后，这个ThreadWorker可以有以下两种用法：

```js
var t1 = new WorkerThread({i: 100} /*shared obj*/);
 
        setInterval(function(){
            t1.run(function(sharedObj){
                    return sharedObj.i++;
                },
                function(r){
                    console.log("t1>" + r.returnValue + ":" + r.error);
                }
            );
        }, 500);
var t2 = new WorkerThread({i: 50});
       
        t2.run(function(sharedObj){   
            while(this.threadSignal){
                sharedObj.i++;
 
                this.runOnUiThread(function(sharedObj){
                    W("body ul").appendChild("<li>"+sharedObj.i+"</li>");
                });
               
                this.sleep(500);
            }
            return sharedObj.i;
        }, function(r){
            console.log("t2>" + r.returnValue + ":" + r.error);
        });
```

这样的用法从形式和语义上来说都让代码具有良好的结构，灵活性和可维护性。

好了，关于Web Worker的用法探讨就介绍到这里，有兴趣的同学可以去看一下这个项目：https://github.com/akira-cn/WorkerThread.js (由于Worker需要用服务器测试，我特意在项目中放了一个山寨的httpd.js，是个非常简陋的http服务的js，直接用node就可以跑起来）。


