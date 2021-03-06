---
title: 浏览器的线程与进程
date: 2019-02-17 13:05:51
tags:
---
### 背景
作为前端工程师，我们的程序的大多数场景都是运行在浏览器环境中，所以今天也想聊一下浏览器中渲染我们页面的一些自己概念和原理

### 知识点
进程和线程是操作系统里的知识，了解了进程和线程的概念，有助于理解浏览器的运行机制
#### 进程
进程是操作系统进行资源分配和调度的最小单位，电脑中每一个软件的运行都是一个进程，每个进程都有独立的一块内存，各个进程之间相互隔离。如果把cpu比作一个工厂的话，进程就相当于一个个独立的车间
#### 线程
线程是程序执行中的一个单一的顺序控制流程，说白了就相当于是车间里的每个工人，每个人都有自己的分内工作，互相协同合作完成一个程序车间的正常运行
#### 进程与线程的区别和关系
1. cpu中包含许多进程，一个进程包含一个或多个线程
2. 进程之间相互独立，同一进程下的多个线程之间共享进程的内存空间(包括代码段、数据集、堆等)及一些进程级的资源(如打开文件和信号)，但是有些共享内存空间同一时刻只允许一个线程运行(比如js引擎线程执行时，GUI渲染引擎就会被挂起)
### 多进程与多线程
- 多进程：只同一个时间里，计算机系统允许同时运行多个进程。比如计算机同时开着聊天工具和音乐软件，保证你能在同时听歌的情况下和好友聊天
- 多线程：在一个进程中，有多个不同分工的线程同时运行，相互合作完成各自的任务(浏览器中运行的多个线程，共同完成页面的渲染展示)

### 浏览器的多进程架构
在"更多工具" -> "任务管理器"中可以查看当前浏览器中运行的进程
我们可以看到的但不限于以下几种：
- Browser主进程：协调、主控作用(浏览器的主进程，负责浏览器界面的显示、各个页面的管理、是所有其他类型进程的祖先、负责它们的创建和销毁等工作，它有且仅有一个)
- GPU进程：用于硬件加速，3D绘制(当且仅当 GPU 硬件加速打开的时候才会被创建，主要用于对 3D 图形加速调用的实现)
![](https://ws1.sinaimg.cn/large/e4d30300ly1g09k86tal4j20ui0lm7ea.jpg)
- Renderer渲染进程：每个tab页的展示
  ![](https://ws1.sinaimg.cn/large/e4d30300ly1g09keeue0tj20ui0lmgw9.jpg)
- 第三方扩展插件进程
  ![](https://ws1.sinaimg.cn/large/e4d30300ly1g09ker79jkj20ui0lmtjg.jpg)
  
**下面也主要介绍渲染进程：**
chrome浏览器每个tab页都是一个进程，使用多个进程来隔离页面，这样做的好处就是每个tab页互不影响，当一个tab崩溃的时候不会影响其他网页，同时进程之间是不共享资源和地址空间的，安全性也能得到更多的保障
#### 浏览器内核
又叫渲染引擎，也就是浏览器的渲染进程，负责对网页语法的解释(html\css\js)并最终渲染呈现给用户
常见的浏览器内核：
- Blink(chrome内核)
- Webkit(safari内核)
- Gecko(Firefox内核)
每一个tab页都会开启一个浏览器渲染引擎实例，也就是一个独立的浏览器渲染进程
浏览器的渲染进程(浏览器内核)又包含多个线程
1. GUI渲染线程
   - 负责解析HTML、CSS -> 构建DOM树、CSSOM、Render树 -> layout布局 -> 绘制
   - 当js操作DOM等原因导致界面发生Repaint或Reflow时，都会触发GUI线程执行
   - GUI渲染线程与JS引擎线程是互斥的，当JS引擎线程执行时，GUI渲染引擎会被挂起保存到一个队列里，当JS引擎线程空闲时，再开始执行
2. JavaScript引擎线程
   - 又称为js内核，常用的是Google的V8引擎、火狐的JaegerMonkey引擎等
   - 解析JavaScript脚本，执行程序
   - 一个渲染引擎中只包含一个js引擎线程，同时JavaScript语言也是单线程模式，因为如果JavaScript是多线程的方式来操作这些UI DOM，则可能出现UI操作的冲突，这也就是为什么Web Worker API支持在主线程的基础上多开一个线程处理js逻辑，但不允许操作DOM的原因，并且有诸多限制，感兴趣的可以了解一下
   - js代码作为一个个任务，由任务队列统一调度分配，JS引擎一直等待任务队列中任务到来，有任务过来时，JS引擎线程便开始执行
   - 因为JavaScript引擎线程会导致GUI渲染线程挂起的原因，常常会出现js阻塞页面渲染的现象，这是我们应该尽量避免的
3. 事件触发线程
   - 我们经常会用到的点击事件等
   - 当js引擎线程执行代码到点击事件时，浏览器会将该事件任务添加到事件触发线程中并开始监听点击事件，当对应的事件满足条件被触发时，该线程就把触发事件后的回调任务添加到task queue队列中，排队等待JS引擎线程来执行回调函数
4. 定时触发器线程
   - setTimeOut\setTimeInterval
   - 当JS引擎线程执行到setTimeOut\setTimeInterval 关键词时，会把定时器任务添加到定时触发器线程中，定时触发器线程开始执行倒数，当倒数之间到了后，将回调任务添加到task queue任务队列中，等待JS引擎线程来执行
5. 异步请求线程
   - 当发送异步ajax请求时，浏览器会开启异步请求线程进行http请求
   - 当检测到请求状态发生变更后，将回调函数添加到task queue任务队列中，等待JS引擎线程来执行

### 延伸扩展
[从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理](https://segmentfault.com/a/1190000012925872)
[CSS3硬件加速也有坑](http://web.jobbole.com/83575/)
[Tasks,microtasks](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
[Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)