---
title: async/await浅谈
date: 2019-01-13 18:20:36
tags: javasrcipt
---
![async](https://ws1.sinaimg.cn/large/e4d30300ly1fz55qykkh2j20m80ciwep.jpg)
### 背景意义
众多周知，Promise的出现使用then链有效的解决了原有回调地狱的问题，现在async/await是对Promise的进一步优化,使异步代码维护起来更像是同步操作

### async
- 可以理解为是Generator 函数的语法糖
- async函数返回一个Promise对象，所以可以在async函数调用后面使用.then()添加回调函数；
- 如果async函数中return的是一个基本类型，会被Promise.resolve()函数进行包装成一个Promise对象，基本类型的值将作为resolve()的参数进行传递
```js
async function test(){
	return 20
}
test().then((data)=>{
    console.log(data) //20
})
```
async返回基本类型时，会被包装成Promise对形象

- 如果async函数中没有显式return，则会return一个没有参数的Promise对象
```js
async function test(){
	console.log('print async') //print async
}
test().then((data)=>{
    console.log('enter...') //enter...
    console.log(data) //undefied
})
```
以上说明async函数没有显式return时，会默认return一个不带任何参数的Promise对象

### await
- await只能在async函数中使用
- await当接收一个Promise对象时，会等待Promise返回的结果，回调的resolve函数参数作为 await 表达式的值，async后面的代码也会一直等待
```js
function f1(){
    return new Promise((resolve,reject)=>{
        resolve(20)
    })}
async function f2(){
    let num = await f1()
    console.log('num',num) //num 20
}
f2()
```
- await当接收一个基本类型值时，await没有任何作用
```js
async function test(){
	let a = await 20
	console.log('a',a) //a 20
}
test()
```
- 任何一个await语句后面的 Promise 对象变为reject状态，那么整个async函数都会中断执行
```js
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
f().then((data)=>{
        console.log(data)
    })
    .catch((err)=>{
        console.log(err) //出错了
    })
```
