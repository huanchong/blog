---
title: ES6-Symbol类型
date: 2019-02-19 21:32:28
tags: ES6
---
### 意义
Symbol是JS新引入的一个基本类型，拓宽了定义对象属性的类型(传统的对象是所有属性都是字符串，如今新增了symbol类型的属性)，同时定义对象属性是不可枚举，通过Object.keys()\Object.getOwnPropertyNames()两种方法是访问不到对象的symbol类型属性的，使得对象属性及值的访问更加的安全
### 使用
1. 使用全局Symbol函数创建symbol类型
```js
let firstName = Symbol('first name') //传入的参数作为symbol变量的描述
let person = {}
person[firstName] = 'Tom'
console.log(person) // {Symbol(first name): "Tom"}
console.log(person[firstName]) // 'Tom'
console.log(firstName) // Symbol('first name') symbol变量的描述信息存储在内部属性[[Description]]中，只有显式或隐式的调用toSting()时，会被读取到，console.log()会隐式的调用toString()方法，所以才打印出来了；String(firstName) 也会隐式调用toSting()、firstName.toString()则是显式调用
console.log('first name' in person) // false  'first name'只是symbol变量的描述信息而已
```
注：**symbol变量最大的特点是不会被转换成字符串和数字类型，所以也就保证了symbol值的唯一性和安全性**

2. ES6新定义的Object.getOwnPropertySymbols()检索symbol类型的对象属性
   Object.keys()\Object.getOwnPropertyNames()两种方法是访问不到对象的symbol类型属性

### 结尾
由于目前使用的场景还不多，所以这里不赘述过多，感兴趣的同学可以去网上查查更多的用法
   