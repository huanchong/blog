---
title: ES6-Destructuring
date: 2019-02-19 16:55:29
tags: ES6
---
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0qs1z3avzj216w0lggpe.jpg)
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
3. 单纯的解构赋值，如果当前对象没有该属性，可以获取到对象继承的属性
   ```js
   let o = Object.create({a:'1',b:'2'}) //o继承了{a:'1',b:'2'}对象
   let {a} = o
   console.log(o) //{}
   console.log(a) //1
   ```
4. 解构赋值是浅拷贝，即如果一个键的值是复合类型的值（数组、对象、函数）、那么解构赋值拷贝的是这个值的引用，而不是这个值的副本
   ```js
    let obj = { a: { b: 1 } };
    let { a } = obj;
    obj.a.b = 3;
    console.log(a.b) // 3
   ```
##### 对象的扩展运算符
- **这是ES2018引入的规则，之前是不支持的**
- 扩展运算符的解构赋值也是浅拷贝
  ```js
  let obj = { a: { b: 1 } };
    let { ...x } = obj;
    obj.a.b = 3;
    console.log(x.a.b) // 3
  ```
- 扩展运算符后面必须接一个变量，并且放到最后面
  ```js
  let { x, ...{ y, z } } = o; //报语法错误
  ```
- 扩展运算符的解构赋值，不能复制继承自原型对象的属性
  ```js
  const o = Object.create({ x: 1, y: 2 });
    o.z = 3;

    let { x, ...newObj } = o;
    let { y, z } = newObj;
    x // 1  普通的对象的解构赋值，可以获取来自原型对象的属性
    y // undefined  扩展运算符的解构赋值取不到原型对象的属性，因此newObj = {z:3}，只有z属性
    z // 3
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