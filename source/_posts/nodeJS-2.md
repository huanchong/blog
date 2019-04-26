---
title: nodeJS核心模块
date: 2019-04-15 13:31:26
tags: node.js
---
## 核心模块
### 全局对象与全局变量
- node环境中，全局对象为global；浏览器环境中，全局对象时window
- 所有全局变量均挂载在全局对象上，作为全局对象的属性，其中包含：
  - 显式定义全局对象的属性
  - 隐式定义的变量(没有定义直接赋值的变量)
  - 这里建议使用 var、let、const 定义变量以避免引入全局变量，因为全局变量会污染 命名空间，提高代码的耦合风险。
#### 常用的全局变量
process：描述当前 Node.js 进程状态 的对象，提供了一个与操作系统的简单接口
- process.argv是命令行参数数组，第一个元素是 node，第二个元素是脚本文件名，从第三个元素开始每个元素是一个运行参数
- process.nextTick(callback)将callback放入异步micro task队列中，如果callback的执行时间需要很长，则通过这个方式可以不阻塞当前主线程
console:控制台标准输出
- console.log()常规输出，也可以使用类似c语言printf()命令的占位符输出console.log('%d year', 1993)
- console.error()输出错误
- console.trace()输出当前调用栈

### 事件驱动 events模块
事件发射与监听器：events.EventEmitter 对象
- EventEmitter.on(event, listener) 为指定事件注册一个监听器，接受一个字符串event 和一个回调函数listener
- EventEmitter.emit(event, [arg1], [arg2], [...]) 发射 event 事件，传递若干可选参数到事件监听器的参数表
- EventEmitter.once(event, listener) 为指定事件注册一个单次监听器，即监听器最多只会触发一次，触发后立刻解除该监听器
- EventEmitter.removeListener(event, listener) 移除指定事件的某个监听器，listener 必须是该事件已经注册过的监听器
- EventEmitter.removeAllListeners([event]) 移除所有事件的所有监听器， 如果指定 event，则移除指定事件的所有监听器
  ```js
    var events = require('events');
    var emitter = new events.EventEmitter();
    emitter.on('event', function(arg1, arg2) { console.log('listener1', arg1, arg2);
    });
    emitter.on('event', function(arg1, arg2) { console.log('listener2', arg1, arg2);
    });
    emitter.emit('someEvent', 'hello', 'world');
    // listener1 hello world
    // listener2 hello world
  ```
> 大多数时候我们不会直接使用 EventEmitter，而是在对象中继承它。包括 fs、net、 http 在内的，只要是支持事件响应的核心模块都是 EventEmitter 的子类，**模块的实例对象可以直接使用on()方法监听事件**


### 文件系统 fs模块
- fs.readFile() 异步读取文件内容
- fs.readFileSync() 同步读取文件内容
- fs.open() 打开文件
- fs.read() 读取文件(从指定的文件描述符 fd 中读取数据并写入 buffer 指向的缓冲区对象)，需要先执行fs.open()打开文件后，在其回调函数中使用fs.read().一般来说，除非必要，否则不要使用这种方式读取文件，因为它要求你手动管理缓冲区 和文件指针，尤其是在你不知道文件大小的时候，这将会是一件很麻烦的事情。
  ```js
    var fs = require('fs');
    fs.open('content.txt', 'r', function(err, fd) { 
        if (err) {
            console.error(err);
            return; 
        }
    var buf = new Buffer(8);
    fs.read(fd, buf, 0, 8, null, function(err, bytesRead, buffer) {
        if (err) { 
            console.error(err); 
            return;
        }
            console.log('bytesRead: ' + bytesRead);
            console.log(buffer);
        })
    });
    // bytesRead: 8
    // <Buffer 54 65 78 74 20 e6 96 87>
  ```
### HTTP 模块
HTTP模块封装了一个HTTP服务器与HTTP客户端
#### HTTP服务器（基于http.Server对象实现）
http.createServer()会创建一个http.Server实例对象
```js
var http = require('http');
http.createServer(function(req, res) { 
    res.writeHead(200, {'Content-Type': 'text/html'}); 
    res.write('<h1>Node.js</h1>');
    res.end('<p>Hello World</p>');
}).listen(3000);
console.log("HTTP server is listening at port 3000.");
```
http.createServer的回调函数中的两个参数 req 和res，分别是 http.ServerRequest和http.ServerResponse 的实例，表示请求和响应信息
##### http.ServerRequest 是 HTTP 请求的信息，所有的业务逻辑都是根据请求传递的内容展开的，所以是后端开发者最关注的内容
包含一些常用属性：
![](https://ws1.sinaimg.cn/large/e4d30300ly1g25hbftx9rj21760j2gpa.jpg)
http.ServerRequest的事件：
- data :当请求体数据到来时，该事件被触发。该事件提供一个参数 chunk，表示接 收到的数据。如果该事件没有被监听，那么请求体将会被抛弃。该事件可能会被调 用多次
- end :当请求体数据传输完成时，该事件被触发，此后将不会再有数据到来
- close: 用户当前请求结束时，该事件被触发

###### 获取GET请求内容：使用url.parse来解析get请求中url后面的参数
```js
var http = require('http'); 
var url = require('url'); 
var util = require('util');
http.createServer(function(req, res) { 
    res.writeHead(200, {'Content-Type': 'text/plain'}); 
    res.end(util.inspect(url.parse(req.url, true))); // util.inspect是将对象转换成字符串的工具函数
}).listen(3000);
```
###### 获取POST请求内容
使用http.ServerRequest的data事件收集post请求体内容，然后使用querystring.parse来解析post请求传过来的内容
```js
var http = require('http');
var querystring = require('querystring'); 
var util = require('util');
http.createServer(function(req, res) { 
    var post = '';
    req.on('data', function(chunk) { 
        post += chunk;
    });
    req.on('end', function() {
        post = querystring.parse(post); 
        res.end(util.inspect(post));
    });
}).listen(3000);
```
>  不要在真正的生产应用中使用上面这种简单的方法来获取 POST 请 求，因为它有严重的效率问题和安全问题，这只是一个帮助你理解的示例。

##### http.ServerResponse 是返回给客户端的信息，决定了用户最终能看到的结果
三个重要的成员函数，用于返回响应头、响应内容以及结束 请求:
- response.writeHead(statusCode, [headers]):向请求的客户端发送响应头。 statusCode 是 HTTP 状态码，如 200 (请求成功)、404 (未找到)等。headers 是一个类似关联数组的对象，表示响应头的每个属性。该函数在一个请求内最多只 能调用一次，如果不调用，则会自动生成一个响应头
- response.write(data, [encoding]):向请求的客户端发送响应内容。data 是 一个 Buffer 或字符串，表示要发送的内容。如果 data 是字符串，那么需要指定 encoding 来说明它的编码方式，默认是 utf-8。在 response.end 调用之前， response.write 可以被多次调用
- response.end([data], [encoding]):结束响应，告知客户端所有发送已经完 成。当所有要返回的内容发送完毕的时候，该函数 必须 被调用一次。它接受两个可 选参数，意义和 response.write 相同。如果不调用该函数，客户端将永远处于 等待状态

#### HTTP 客户端（作为客户端向服务器发起请求）
- http.request(options, callback) 发起 HTTP 请求
  - option 是 一个类似关联数组的对象，表示请求的参数，callback 是请求的回调函数
  - callback 传递一个参数，为http.ClientResponse的实例
  - http.request 返回一个http.ClientRequest的实例
  ```js
    var http = require('http');
    var querystring = require('querystring');
    var contents = querystring.stringify({
    name: 'byvoid',
    email: 'byvoid@byvoid.com',
    address: 'Zijing 2#, Tsinghua University',
    });
    var options = {
    host: 'www.byvoid.com',
    path: '/application/node/post.php', method: 'POST',
    headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'Content-Length' : contents.length
        }
    };
    var req = http.request(options, function(res) { res.setEncoding('utf8');
    res.on('data', function (data) {
            console.log(data);
        });
    });
    req.write(contents);
    req.end(); //  不要忘了通过 req.end() 结束请求，否则服务器将不会收到信息
  ```
- http.get(options, callback) get请求的简化版，唯一的区别在于http.get 自动将请求方法设为了 GET 请求，同时**不需要手动调用 req.end()**

##### http.ClientRequest 对象（http.request()返回的实例对象）
对象含有的方法：
- write()服务器发送请求体
- end()写结束以后必须调用 end 函数以通知服务器，否则请求无效

##### http.ClientResponse 对象（http.request()回调函数中的参数为该实例对象）
提供三个事件：
- data 数据达到是触发
- end 数据传输结束后触发
- close 连接结束后触发
![](https://ws1.sinaimg.cn/large/e4d30300ly1g25puxxjy2j216q09sgn9.jpg)

三个特殊的函数：
- response.setEncoding([encoding]):设置默认的编码，当 data 事件被触发时，数据将会以 encoding 编码。默认值是 null，即不编码，以 Buffer 的形式存储。常用编码为 utf8
- response.pause():暂停接收数据和发送事件，方便实现下载功能
- response.resume():从暂停的状态中恢复