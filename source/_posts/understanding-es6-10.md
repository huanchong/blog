---
title: ES6-Array
date: 2019-02-24 13:42:09
tags: ES6
---
### 对数组功能的增强
#### 数组的创建
1. Array.of()
   数组构造器new Array()创建数组时，因为传递参数的个数与类型不同，会出现怪异的行为，因此引进了Array.of()方法创建数组
   ```js
   let items = new Array(2) //如果只有一个参数，并且是数值型，new Array()构造器就把参数理解成为数组的长度，所以元素都为undefined
   console.log(items[0]) //undefined
   console.log(items[1]) //undefined

    items = new Array(2,1) //如果参数超过1个，则new Array()构造器就把参数理解成为数组的元素
    console.log(items[0]) //2
   console.log(items[1])  //1

   //所以这里作为对参数理解的统一，使用Array.of()都会把参数理解成为元素
   items = Array.of(2)
   console.log(items[0]) //2
   ```
2. Array.from()
   将类数组对象(具有数值类型的索引+length长度属性，比如DOM的NodeList)转换成数组类型
   传统的比较快捷的方法是使用slice
   ```js
   Array.prototype.slice.call(arrayLike)
   ```
   ES6提供的解决方法Array.from()，支持将**类数组对象、可迭代对象(包含Symbol.iterator属性)**转换成数组类型
   ```js
   Array.from(arrayLike)
   ```
#### 数组实例的新方法
1. find()\findIndex() 根据过滤条件返回想要的第一个满足条件的元素值
   ```js
   let numbers = [12,20,33,45]
   let value = numbers.find(value => value>30)
   let index = numbers.findIndex(value => value>30)
   console.log(value) //33
   console.log(index) //2
   ```
2. fill()数组元素的赋值填充，第一个必填参数为想要赋新值的元素，支持指定第二、第三个参数表示赋值开始和结束的位置
   ```js
   let arr = [1,2,3,4]
   arr.fill(1) //将元素全部重新赋值
   console.log(arr.toString()) //1,1,1,1
   arr.fill(0,1)
   console.log(arr.toString()) //1,0,0,0
   arr.fill(10,1,3)
   console.log(arr.toString()) //1,10,10,0
   ```
3. copyWithin()允许在数组内部复制自身的元素
#### 类型化数组
不是严格意义上的数组，主要是为JS提供快速按位计算的能力，fill()\copyWithin()方法都是基于此创建的，平时使用的场景比较少，感兴趣的可以继续了解下