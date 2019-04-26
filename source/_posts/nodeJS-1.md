---
title: nodeJS入门指南
date: 2019-04-14 12:49:38
tags: node.js
---
### node.js简介
- node.js不是一种独立语言，是一个可以让js在服务器端运行的平台
- 区别与传统语言平台依靠多线程来实现高并发的设计思路，node采用单线程、异步式I/O、事件驱动式的程序设计模型
- node.js使用的JavaScript引擎为V8引擎，还使用了高效的 **libev 和 libeio 库支持事件驱动和异步式 I/O** 来代替传统平台的多线程模式，带来性能的提升（传统多线程模式对于 高并发的访问，一方面线程长期阻塞等待，另一方面为了应付新请求而不断增加线程，因此 会浪费大量系统资源，同时线程的增多也会占用大量的 CPU 时间来处理内存上下文切换， 而且还容易遭受低速连接攻击。）
- 有别于php等语言，在使用php构建web服务时，必须外部依赖一个Nginx、Apache等http服务器，然后通过http服务区的模块加载或CGI调用，才能将php的执行结果呈现给用户；而node.js内置了http服务器模块，无需再额外搭建
- node.js可以调用c/c++代码，对于性能要求比较高的模块可以使用c/c++来实现
![](https://ws1.sinaimg.cn/large/e4d30300ly1g2242fwxptj20wm0mign9.jpg)

![](https://ws1.sinaimg.cn/large/e4d30300ly1g224ukryq5j20l00gcdgz.jpg)

node.js只在第一次运行时会读取文件内容，以后都会直接访问内存，也就是当文件实时改动时，不会触发重新编译文件内容，所以需要使用**supervisor、pm2**等工具，对node服务进行实时管理

### 异步式I/O 与事件驱动式编程
#### 阻塞与线程
> 什么是阻塞(block)呢?线程在执行中如果遇到磁盘读写或网络通信(统称为 I/O 操作)， 通常要耗费较长的时间，这时操作系统会剥夺这个线程的 CPU 控制权，使其暂停执行，同 时将资源让给其他的工作线程，这种线程调度方式称为 阻塞。当 I/O 操作完毕时，操作系统 将这个线程的阻塞状态解除，恢复其对CPU的控制权，令其继续执行。这种 I/O 模式就是通 常的同步式 I/O(Synchronous I/O)或阻塞式 I/O (Blocking I/O)  


> 异步式 I/O (Asynchronous I/O)或非阻塞式 I/O (Non-blocking I/O)则针对 所有 I/O 操作不采用阻塞的策略。**当线程遇到 I/O 操作时，不会以阻塞的方式等待 I/O 操作 的完成或数据的返回，而只是将 I/O 请求发送给操作系统，继续执行下一条语句**。当操作 系统完成 I/O 操作时，以**事件的形式**通知执行 I/O 操作的线程，线程会在特定时候处理这个 事件。为了处理异步 I/O，线程必须有事件循环，不断地检查有没有未处理的事件，依次予 以处理。

#### 多线程与单线程的比较：
对于需要长时间计算的I/O处理操作
- 多线程模式：如果想实现高并发，就必须开启多个线程
![](https://ws1.sinaimg.cn/large/e4d30300ly1g225j31u3mj20qu0oiaci.jpg)

- 单线程模式：I/O异步，通过事件方式通知线程，即可完成高并发
![](https://ws1.sinaimg.cn/large/e4d30300ly1g225kaxr3sj20po0swju4.jpg)

单线程事件驱动的异步式 I/O 比传统的多线程阻塞式 I/O 究竟好在哪里呢?
- 简而言之， 异步式 I/O 就是少了多线程的开销。对操作系统来说，创建一个线程的代价是十分昂贵的， 需要给它分配内存、列入调度，同时在线程切换的时候还要执行内存换页，CPU 的缓存被 清空，切换回来的时候还要重新从内存中读取信息，破坏了数据的局部性。(基于多线程的模型也有相应的解决方案，如轻量级线程(lightweight thread)等。事件驱动的单线程异步模型与多线 程同步模型到底谁更好是一件非常有争议的事情，因为尽管消耗资源，后者的吞吐率并不比前者低)
- 异步式编程的缺点在于不符合人们一般的程序设计思维，存在回调地狱的情况（使用async、promise等编写方式）
![](https://ws1.sinaimg.cn/large/e4d30300ly1g225r057v1j216k0ceada.jpg)

### node.js的事件循环机制
> Node.js 程序由事件循环开始，到事件循环结束，所有的逻辑都是事件的回调函数，所以 Node.js 始终在事件循环中，程序入口就是 事件循环第一个事件的回调函数。事件的回调函数在执行的过程中，可能会发出 I/O 请求或 直接发射(emit)事件，执行完毕后再返回事件循环，事件循环会检查事件队列中有没有未 处理的事件，直到程序结束.
> 
> 与其他语言不同的是，Node.js 没有显式的事件循环，类似 Ruby 的 EventMachine::run() 的函数在 Node.js中是不存在的。Node.js的事件循环对开发者不可见，由libev库实现。libev支持多种类型的事件，如 ev_io、ev_timer、ev_signal、ev_idle 等，在 Node.js 中均被 EventEmitter 封装。libev 事件循环的每一次迭代，在 Node.js 中就是一次 Tick，libev 不 断检查是否有活动的、可供检测的事件监听器，直到检测不到时才退出事件循环，进程结束

![](https://ws1.sinaimg.cn/large/e4d30300ly1g22cm3qesvj20ps0s4q59.jpg)

### npm包管理器
#### 创建npm包
Node.js 根 据 CommonJS 规范实现了包机制，开发了 npm来解决包的发布和获取需求。
使用如下命令初始化创建npm包
```shell
npm init
```
Node.js 的包是一个目录，其中包含一个 JSON 格式的包说明文件 package.json。严格符 合 CommonJS 规范的包应该具备以下特征:
- package.json 必须在包的顶层目录下; 
  - 作为npm包的描述性文件，其中常用的字段:
  - **main**:作为包模块的入口（Node.js 在调用某个包时，会首先检查包中 package.json 文件的 main 字段，将其作为包的接口模块，如果 package.json 或 main 字段不存在，会尝试寻找 index.js 或 index.node 作 为包的接口）
  - name:包的名称
  - description:包的简要说明
  - version:版本字符串
  - keywords:关键字数组，通常用于搜索
  - maintainers:维护者数组，每个元素要包含 name、email (可选)、web (可选)字段
  - contributors:贡献者数组，格式与maintainers相同。包的作者应该是贡献者数组的第一个元素
  - bugs:提交bug的地址，可以是网址或者电子邮件地址
  - licenses:许可证数组
  - repositories:仓库托管地址数组
  - dependencies:包的依赖，一个关联数组，由包名称和版本号组成
- 二进制文件应该在 bin 目录下;
- JavaScript 代码应该在 lib 目录下;
- 文档应该在 doc 目录下;
- 单元测试应该在 test 目录下。
Node.js 对包的要求并没有这么严格，只要顶层目录下有 package.json，并符合一些规范 即可。当然为了提高兼容性，我们还是建议你在制作包的时候，严格遵守 CommonJS 规范

#### 本地模式与全局模式
全局模式的安装：
> npm [install/i] -g [package_name]

>为什么要使用全局模式呢?多数时候并不是因为许多程序都有可能用到它，为了减少多 重副本而使用全局模式，而是因为本地模式不会注册 PATH 环境变量。举例说明，我们安装 supervisor 是为了在命令行中运行它，譬如直接运行 supervisor script.js，这时就需要在 PATH 环境变量中注册 supervisor。npm 本地模式仅仅是把包安装到 node_modules 子目录下，其中 的 bin 目录没有包含在 PATH 环境变量中，不能直接在命令行中调用。而当我们使用全局模 式安装时，npm 会将包安装到系统目录，譬如 /usr/local/lib/node_modules/，同时 package.json 文 件中 bin 字段包含的文件会被链接到 /usr/local/bin/。/usr/local/bin/ 是在PATH 环境变量中默认 定义的，因此就可以直接在命令行中运行 supervisor script.js命令了

使用全局模式安装的包并不能直接在 JavaScript 文件中用 require 获 得，因为 require 不会搜索 /usr/local/lib/node_modules/

![](https://ws1.sinaimg.cn/large/e4d30300ly1g22dbja4ptj217809emyq.jpg)
区别：
- 当我们要把某个包作为工程运行时的一部分时，通过本地模式获取
- 如果要 在命令行下使用，则使用全局模式安装

#### 创建全局链接
> npm 提供了一个有趣的命令 npm link，它的功能是在本地包和全局包之间创建符号链 接。我们说过使用全局模式安装的包不能直接通过 require 使用，但通过 npm link命令 可以打破这一限制。举个例子，我们已经通过 npm install -g express 安装了 express， 5 这时在工程的目录下运行命令:
> **npm link express**
./node_modules/express -> /usr/local/lib/node_modules/express
我们可以在 node_modules 子目录中发现一个指向安装到全局的包的符号链接。通过这种方法，我们就可以把全局包当本地包来使用了

>除了将全局的包链接到本地以外，使用 npm link命令还可以将本地的包链接到全局。 使用方法是在包目录( package.json 所在目录)中运行 npm link 命令。如果我们要开发 一个包，利用这种方法可以非常方便地在不同的工程间进行调试和测试

##### 具体使用案例：
现在有两个模块，npm-link-module为我们开发的npm包；npm-link-test为我们要使用上面的npm包来做测试
```shell
cd npm-link-module  //进入开发的npm包目录里
npm link //将本地的包链接到全局

cd npm-link-test  //进入到测试项目中
npm link npm-link-module  //将全局包npm-link-module 链接到本地node_modules路径下，这样本地就可以使用require()引用了
```

#### 包的发布
- npm adduser 创建账号
- npm whoami 命令测验是否已经取得了账号
- npm publish 发布包 
- 通过https://www.npmjs.com/ 去搜索自己发布的包
- 日后有更新时，更改package.json中的version版本，重新npm publish即可
- npm unpublish 取消已发布的包
  
### node命令行调试(调试功能由 V8 提供)
使用node debug xxx.js 即可开启调试模式
![](https://ws1.sinaimg.cn/large/e4d30300ly1g22ef0cf5zj216s0x8dmr.jpg)