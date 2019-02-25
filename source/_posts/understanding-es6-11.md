---
title: ES6-Promise
date: 2019-02-24 15:06:44
tags: ES6
---
### Promise与异步编程
#### 创建一个待执行状态的Promise
new Promise()会立即执行，因此如果需要在合适的位置创建Promise，可以使用一个函数进行包装，合适的位置进行调用
```js
let fs = require('fs')
function readFile(fileName){
    return new Promise((resolve, reject) => {
        //异步操作
        fs.readFile(fileName, (err, contents)=> {
            //读取错误时
            if(err){
                reject(err)
                return;
            }
            //读取成功时
            resolve(contents)
        })
    })
}
let file = readFile('readme.txt')
// 同时监听成功或失败
file.then(data =>{

},err=>{

})
```
#### 直接返回一个已执行状态的Promise对象
```js
Promise.resolve()
Promise.reject()
```
- 传递给Promise.resolve()的参数，如果是一个Promise对象，则会原样返回，如果不是一个Promise对象，针对非Promise的thenable对象，会做进一步包装，将其转换成Promise对象再返回（基本类型的值也会被自动转换成Promise对象）
- 非Promise的thenable对象：具有接受resolve、reject参数的then()方法的对象
  ```js
  let thenable = {
      then(resolve,reject){
          console.log('enter thenable')
          resolve(42)
      }
  }
  let p1 = Promise.resolve(thenable)
  console.log('before then')
  p1.then(data =>{
      console.log(data)
  }).catch(err => {
      console.log(err)
  })
  /*
  before then
  enter thenable
  42
  */
  ```
  上例中
  1.程序按顺序执行，遇到`Promise.resolve(thenable)`时会把thenable中的then()中的异步函数放入microtask队列中，等主程序执行完后调用
  2. 执行`console.log('before then')`
  3. 执行p1.then()这里，监听p1的状态，一旦由pending变为resolve或reject，便将其放入microtask队列中，在主程序执行完后执行
  4. 因此先打印`before then`，然后再打印`enter thenable`，最后是`42`

#### then()、catch()执行完后都会返回一个Promise对象，如果没有显式指定return，则会返回一个默认的Promise

#### 响应多个Promise
1. Promise.all([, , ,])参数是一个可迭代的对象(常用数组)，每个元素都是Promise对象
   ```js
   let p1 = new Promise((resolve, reject) => {
       resolve('p1')
   })
   let p2 = new Promise((resolve, reject) => {
       resolve('p2')
   })
   let p3 = new Promise((resolve, reject) => {
       resolve('p3')
   })
   let p4 = Promise.all([p1, p2, p3])
   p4.then(value => {
       console.log(Array.isArray(value)) //true
       console.log(value)  //["p1", "p2", "p3"]
   })
   ```
   `Promise.all([]).then()`then()中的参数是一个数组，成员是你传入的多个Promise对象中resolve的值
   ```js
   let p1 = new Promise((resolve, reject) => {
       resolve('p1')
   })
   let p2 = new Promise((resolve, reject) => {
       reject('p2')
   })
   let p3 = new Promise((resolve, reject) => {
       resolve('p3')
   })
   let p4 = Promise.all([p1, p2, p3])
   p4.then(value => {
       console.log(Array.isArray(value)) //true
       console.log(value)  //["p1", "p2", "p3"]
   }).catch(err => {
        console.log('err',err)
        }
    )
   ```
   如果Promise对象中触发reject状态，则会立即触发`Promise.all([]).catch()`catch()中的参数是触发reject()返回的值

2. Promise.race([, , ,]) 多个Promise对象竞赛，谁先触发状态变化，立刻停止
   ```js
   let p1 = new Promise((resolve, reject) => {
       resolve('p1')
   })
    let p2 = Promise.resolve('p2')
    let p3 = new Promise((resolve, reject) => {
        resolve('p3')
    })
    let p4 = Promise.race([p1, p2, p3])
    p4.then(value => {
        console.log(value)  
    }).catch(err => {
    console.log('err',err)
    })
    // p1
   ```