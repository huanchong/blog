---
title: ES6-Destructuring
date: 2019-02-19 16:55:29
tags: ES6
---
### 对对象与数组的解构
#### 意义
使用解构操作符，可以更方便的提取对象和数组中的的单个成员元素
#### 对象解构
1. 带默认值的解构
   ```js
   let person = {
       name: 'Tom',
       age: '20'
   }
   let { name = 'Eric', age} = person
   ```
2. 赋值给本地不同的变量名
   ```js
   let person = {
       name: 'Tom',
       age: '20'
   }
   let { name:firstName = 'Eric', age:ageOld} = person // 冒号前面的表示要检索的对象的属性名；冒号后面的表示要赋值的变量名
   console.log(firstName) // Tom
   console.log(ageOld) // 20
   ```
#### 数组解构
1. 按照位置依次解构赋值,可以赋默认值
   ```js
   let color = ['red','green','blue']
   let [first, second='yellow'] = color
   console.log(first) // red
   console.log(second)// green

   let [ , , third] = color
   console.log(third) // blue
   ```
2. 互换变量名(无需再借助中间变量)
   ```js
   let a= 1, 
       b= 2;
   [a, b] = [b, a]
   console.log(a,b) //2,1
   ```
3. 使用扩展运算符完成剩余项赋值
   ```js
   let color = ['red','green','blue']
   let [first, ...restColor] = color //剩余项必须放在最后面
   console.log(first) // red
   console.log(restColor)// ["green", "blue"]
   ```
   常用的应用：数组的赋值 （传统的方法是使用 .concat()函数）
   ```js
   let color = ['red','green','blue']
   let [...colorArr] = color
   console.log(colorArr) // ["red", "green", "blue"]    
   ```
#### 函数参数解构
```js
function setcookie(name, value, { secure, path, domain, expires= 500 }){
    //...
}
// 但这样写有个问题，第三个解构的实参必须要传一个对象，否则解构的{ secure, path, domain, expires= 500 } = undefined 会报错
setcookie('load','123') //报错

更完善的写法是函数参数赋默认值
function setcookie(name, value, { secure, path, domain, expires= 500 } = {}){
    //...
}
```