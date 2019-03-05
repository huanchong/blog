---
title: ES6-Function
date: 2019-01-26 14:08:30
tags: ES6
---
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0qs1z3avzj216w0lggpe.jpg)
### 函数
#### 函数的形参带默认值
```js
function demo(url, timeout = 500, callback = function(){}){
    //....函数体
}
```
当函数调用时传入的实参是undefined或不传递时，会取默认值；**null**值是有效值，不会取默认值
```js
function demo(url, timeout = 500, callback = function(){}){
    console.log('timeout:',timeout)
}
demo('url',null) // timeout: null

function demo(url, timeout = 500, callback = function(){}){
    console.log('timeout:',timeout)
}
demo('url',undefined) // timeout: 500
```
- 函数形参默认值可以为表达式
  ```js
    function getValue(value){
        return value+5
    }
    function add(first, second=getValue(first)){
        return first + second
    }
    console.log(add(1,2)) // 3
    console.log(add(1)) // 7
  ```
- 函数形参区域有自己独立的块级作用域及暂时性死区，与函数体的作用域相分离，所以函数形参区域不能访问函数体中的变量并且由于暂时性死区的原因，不能访问没初始化的变量
  ```js
    function getValue(value){
        return value+5
    }
    function add(second=getValue(first), first){
        return first + second
    }
    console.log(add(undefined,2)) // Uncaught ReferenceError: first is not defined
  ```
#### 剩余参数
在函数形参的最后一位，使用<b>...具名参数名字</b> 表示除了其他前面的参数，剩下的参数都存放在这个具名参数的**数组**中
```js
function add(first, ...rest){
    // rest为数组
}
```

#### 函数构造器的增强能力
使用Function构造器动态创建函数
```js
var add = new Function('first','second','return first + second')
console.log(add(1,2)) // 3

//使用默认参数
var add = new Function('first','second = first','return first + second')
console.log(add(1)) // 2
```
#### 扩展运算符
在数组前面加上...，将数组分割成单个参数依次传入
```js
var arr = [-10,-23,0,12]
console.log(Math.max(...arr)) // 12
```

#### 箭头函数
##### 特点
- 常用来替代匿名函数表达式
- 箭头函数中没有this绑定，内部继承外部函数的this指向，因此使用call()\apply()\bind()不能修改内部的this指向
- 箭头函数不能用作构造器，所以不能使用new来定义新的类型、没有原型
  ```js
  let ArrowType = () => {}
  let obj = new ArrowType() // ArrowType is not a constructor
  ```
##### 使用
单一参数，函数体只包含单一表达式
```js
var getValue = value => value 
//等价于
var getValue = function(value){
    return value
}
```
多个参数，函数体只包含单一表达式
```js
var add = (first, second) => first + second
//等价于
var add = function(first, second){
    return first + second
}
```
多个参数，函数体包含多个表达式
```js
var compare = (num1, num2) => {
    let result = num1 > num2
    return result
}
```
花括号表示函数体，如果想返回一个对象，则使用()将对象括起来
```js
var obj = () => ({a:123, b:234})
//使用()括号包起来，来告诉js引擎这是返回一个对象字面量
```

