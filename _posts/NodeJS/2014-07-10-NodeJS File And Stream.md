## 说说 NodeJS 里的File 与 Stream

先来个题外话,听闻`express`的作者`TJ`大神已投入到`Go`的怀抱啦!真是遗憾啊，感谢大神对NodeJS社区的贡献.

### File

文件操作对于一般的后端语言来说,是非常基础的功能,今天来说说NodeJS里的文件API,详情API地址可以看这里,<a href="http://nodejs.org/api/fs.html">官方文件API</a>,本篇只说一些常用的方法

---


首先文件api在NodeJS里是唯一的一个具有同步和异步的方法调用,所有的文件相关的方法都在`file`模块内，这个是系统内置模块，请求方法如下

```js

var fs = require('file');

```
接下来就可以测试文件API啦,先来一个常用的`readFile`方法

> fs.readFile(filename, [options], callback)

说到读文件，这里就要先讲讲NodeJS的路径问题

* __dirname 程序目录
* process.cwd() 代表程序当前工作目录
* . 相对于程序路径的上级
* .. 相对于程序路径的上上级

比如这里获取当前运行文件同级目录里的`feenan.js`,代码如下

```js
fs.readFile(__dirname + '/feenan.js', 'utf8', function(err, data){
	// do stuff here	
	if(!err){
		console.log(data);
	}
})

```

方法的第二个参数可以传递数据编码格式，这里设置`utf8`,最后一个参数是一个回调，当文件读取完毕的时候会调用，出错的话，`err`参数有值,否则为`null`,这时就可以对数据进行处理了.

这个方法对应的同步方法是`readFileSync`,一般同步方法就是在后面加上`Sync`

目前这个读取文件的方案，对于小文件还可以接受，但是大文件的话，则性能不是很好，因为这是一次性的分配文件内容大小的内存的，后面会讲到以流的方式来处理.

说完了读取文件，再来说说读取文件夹

> fs.readdir(path, callback)

读取文件夹信息比较简单，传递目录路径，以及回调函数.

```js

fs.readdir(__dirname, function(err, files){
	if(!err){
		files.forEach(function(v){
			console.log(v);
		})
	}
});

```
上面的主要说是读取当前程序所在目录下面的信息,回调函数的`files`是一个数组，里面存放目录列表的名字

这里当获取到目录列表的时候，想读取某个文件怎么办？怎么来区分哪个是目录哪个是文件呢？这里得讲下另外一个常用的api

> fs.stat(path, callback)

上面的API是读取文件属性,这里的文件包括目录以及单个文件,直接来看个例子吧,对上面返回的文件数组进行过滤然后分别显示哪些是目录，哪些是文件

```js

function file(i){
	var filename = files[i];
	fs.stat(__dirname + '/' + filename, function(err, stat){
		// 保存文件属性对象
		stats[i] = stat;
		if(err){
			console.log('file read error!');
		}else{
			if(stat.isDirectory()){
				console.log('目录名: ' + filename);
			}else{
				console.log('文件名: ' + filename);
			}
			i++;
			if(i === files.length){
				done();
			}else{
				file(i);
			}
		}
	});
}
file(0);

```

上面主要是用到了`fs.stat`返回的`stat.isDirectory`方法来检查是否是文件夹

最后贴一个完整的获取目录下的文件信息，并支持命令行输入索引来查看文件信息

```js

var fs = require('fs');
var stdin = process.stdin,
	stdout = process.stdout;

// 异步调用获取当前工作目录下的列表
fs.readdir(__dirname, function(err, files){
	// 列出文件列表之后，提示输入想查看的内容
	var done = function(){
		console.log('');
		stdout.write('输入你的选择: ');
		stdin.resume();
		stdin.setEncoding('utf8');
		stdin.on('data', function(data){
			if(!files[+data]){
				stdout.write('输入你的选择: ');
			}else{
				stdin.pause();
				// 开始读取选择的文件夹或者文件
				var filename = files[+data];
				if(stats[+data].isDirectory()){
					// 开始读取文件夹里的内容
					fs.readdir(__dirname + '/' + filename, function(err, files){
						if(!err){
							console.log('');
							console.log('(' + files.length  + '个文件' + ')');
							files.forEach(function(v){
								console.log('      -' + v);
							})
							console.log('');
						}
					});
				}else{
					// 开始读取文件
					fs.readFile(__dirname + '/' + filename, 'utf8',  function(err, data){
						console.log('');
						console.log(data.replace(/(.*)g/, '    $1'));
					});
				}
			}
		});
	}
	var stats = [];
	if(!files.length){
		console.log('不存在任何文件列表!');
		return;
	}
	console.log('列出你想看的列表内容: ');
	function file(i){
		var filename = files[i];
		fs.stat(__dirname + '/' + filename, function(err, stat){
			// 保存文件属性对象
			stats[i] = stat;
			if(err){
				console.log('file read error!');
			}else{
				if(stat.isDirectory()){
					console.log('目录名: ' + filename);
				}else{
					console.log('文件名: ' + filename);
				}
				i++;
				if(i === files.length){
					done();
				}else{
					file(i);
				}
			}
		});
	}
	file(0);
});

```

### Stream

刚才上面讲到了读取文件时的一个影响性能的地方，这里可以用流来处理，看看代码

```js

var readstream = fs.createReadStream(__dirname + '/demo.js', { Encoding: 'utf8'});
var content = [];

readstream.on('data', function(data){
	content.push(data);
})

readstream.on('end', function(data){
	content.push(data);
	console.log(content.join(''));
})

```

上面用到了文件流里的`createReadStream`方法,参数如下

> fs.createReadStream(path, [options])

选项里可以传入文件编码信息，更多参数可以参考官网API

然后这里的只读流对象提供了读取数据以及数据完成事件，这种方式跟上面的一次性读取内容的区别在于，内存是一块一块分配的，而不是一次性分配很大的内存，这种情况用在网络请求文件的时候，性能尤其明显

下面再来一个可写流，方法参数如下

> fs.createWriteStream(path, [options])

```js

var writestream = fs.createWriteStream(__dirname + '/demo.js', { Encoding: 'utf8'});
writestream.write('hello feenan!');

```

而且有一个方便的方法可以从只读流写入到可写流中，效率也是最好的

```js

var readstream = fs.createReadStream(__dirname + '/demo.js', { Encoding: 'utf8'});
var writestream = fs.createWriteStream(__dirname + '/demo.js', { Encoding: 'utf8'});
readstream.pipe(writestream);

```

其实就是调用了流的`pipe`管道方法,确实人如其名:)


### 总结

文件与流都是`file`模块下面的API，用好这些方法，对构建前端自动化，编写网络服务程序都有很好的帮助!






	

