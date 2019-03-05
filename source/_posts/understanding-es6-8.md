---
title: ES6-Iterator/Generator
date: 2019-02-20 16:36:10
tags: ES6
---
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0qs1z3avzj216w0lggpe.jpg)
### 迭代器与生成器
#### 背景
传统的遍历迭代集合的方式使用for循环，ES6引入了迭代器对象，使得操作集合更加便捷，Set、Map、for...of、扩展运算符 都用到了迭代器，**注：普通的对象不是可迭代的，因为没有Symbol(Symbol.iterator)属性**
#### 迭代器Iterator
迭代器表示可以被遍历迭代的对象，以编程的方式返回集合中的下一项，迭代器对象都拥有next()方法，返回{value:'',done:bool},value表示当前迭代位置的返回值，done表示是否迭代结束，结束时其值为false

#### 生成器Generator
是一个返回迭代器对象的函数
1. 使用 function *函数名(){} 来创建生成器
2. 使用生成器函数表达式创建生成器：let iterator = function *(){}  **注：不能使用箭头函数创建生成器**
3. 函数体中使用yield关键字定义每次迭代器调用next()返回的value结果，yield指令相当于起到暂停代码执行的作用，只有在迭代器调用next()时，才会触发下一个yield继续执行
4. 迭代完yield后，下一次迭代会返回return的值；如果return放在中间，则迭代到return行后，后面的yield也不会执行了
```js
function *createIterator(){
    yield 1;
    yield 2;
    yield 3;
    return 123
}
let iterator = createIterator()
console.log(iterator.next()) //{value: 1, done: false}
console.log(iterator.next()) //{value: 2, done: false}
console.log(iterator.next()) //{value: 3, done: false}
console.log(iterator.next()) //{value: 123, done: true}
console.log(iterator.next()) //{value: undefined, done: true}
```
### 可迭代对象与for...of
#### 可迭代对象(iterable)是指带Symbol(Symbol.iterator)属性的对象，Symbol(Symbol.iterator) 本质上是一个生成器函数，所有的集合对象(数组、Set、Map)及字符串的原型上都默认携带这个属性，因此都是可迭代对象
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0d6x83uz6j211u0naq6h.jpg)
#### 可迭代对象可以使用ES6新增的for-of方法遍历
for-of的优势：使用传统for循环时，需要通过逻辑控制遍历条件，特别是多级嵌套时会导致复杂度增加，容易出错
for-of循环的原理：
1. 首先内部会自动调用集合的Symbol.iterator方法，获取到默认的迭代器对象
2. 然后在每次循环时，都会自动调迭代器的next()，将返回结果对象的value值赋给临时变量上(迭代器返回的结果对象的value就是可迭代对象上定义的值)
3. 下一轮循环继续调用next()，直到遇到返回的结果对象的done属性值为true，停止遍历
   ```js
   let arr = [1,2,3]
   let iterator = arr[Symbol.iterator]()
   console.log(iterator.next()) //{value: 1, done: false}
   console.log(iterator.next()) //{value: 2, done: false}
   console.log(iterator.next()) //{value: 3, done: false}
   ```
### 创建可迭代对象
因为我们正常创建的对象是没有Symbol.iterator属性的，所以是不可迭代的。我们可以自定义一个Symbol.iterator属性
```js
let obj = {
    items:[],
    *[Symbol.iterator](){ //使用Symbol类型要使用[]
        for(let item of this.items){
            yield item
        }
    }
}
obj.items.push(1)
obj.items.push(2)
obj.items.push(3)
console.log(obj) // {items: Array(3), Symbol(Symbol.iterator): ƒ}
for(let value of obj){
    console.log(value) //1,2,3
}
```
### 集合内置的迭代器
ES6具有内置迭代器的集合对象类型：数组、Set、Map
- entries():返回一个包含键值对的迭代器，迭代返回的结果对象value是键值对数组
- values():返回一个包含集合中的值的迭代器
- keys():返回一个包含集合中的键的迭代器
#### 在for-of中使用上面三种的其中一种都可以，下面是显式指定迭代器的写法：
```js
let colors = ['red', 'green', 'blue']
let tracking = new Set([123,456,789])
let data = new Map()

data.set('title','understanding ES6')
data.set('format', 'ebook')

for(let entry of colors.entries()){
    console.log(entry)
}
for(let entry of tracking.entries()){
    console.log(entry)
}
for(let entry of data.entries()){
    console.log(entry)
}
/*
[0, "red"]
[1, "green"]
[2, "blue"]
[123, 123]
[456, 456]
[789, 789]
["title", "understanding ES6"]
["format", "ebook"]
*/

for(let value of colors.values()){
    console.log(value)
}
for(let value of tracking.values()){
    console.log(value)
}
for(let value of data.values()){
    console.log(value)
}
/*
red
green
blue
123
456
789
understanding ES6
ebook
*/
for(let key of colors.keys()){
    console.log(key)
}
for(let key of tracking.keys()){
    console.log(key)
}
for(let key of data.keys()){
    console.log(key)
}
/*
0
1
2
123
456
789
title
format
*/
```
#### 三种几个类型默认的迭代器是什么呢？
当都有for-of循环没有显式指定迭代器时，每种集合类型都有一个默认的迭代器供循环调用
- values():数组与Set的默认迭代器
- entries():Map的默认迭代器
```js
let colors = ['red', 'green', 'blue']
let tracking = new Set([123,456,789])
let data = new Map()

data.set('title','understanding ES6')
data.set('format', 'ebook')

//与使用colors.values()相同
for(let value of colors){
    console.log(value)
}
/*
red
green
blue
*/

//与使用tracking.values()相同
for(let value of tracking){
    console.log(value)
}
/*
123
456
789
*/

//与使用data.entries()相同
//使用了数组解构，方便操作迭代器返回的结果对象[0, "red"]格式
for(let [key,value] of data){
    console.log(key + ' ' + value)
}
/*
title understanding ES6
format ebook
*/
```
### NodeList的迭代器
使用document.getElementsByXXX获取的文档对象模型(DOM)是一种NodeList类型，其比较大的特点是，有类似数组的length属性，同时具有默认迭代器，因此NodeList可以使用for-of迭代

### 扩展运算符
扩展运算符可以应用到任何可迭代的集合上，扩展运算符会调用默认的迭代器，返回相应的结果对象value值

### 生成器委托：可以将多个生成器合并到一起
```js
function *numberIterator(){
    yield 1;
    yield 2;
}
function *colorIterator(){
    yield 'red';
    yield 'blue';
}
function *combinedInterator(){
    yield *numberIterator();
    yield *colorIterator();
    yield true
}

let interator = combinedInterator()
console.log(interator.next()) //{value: 1, done: false}
console.log(interator.next()) //{value: 2, done: false}
console.log(interator.next()) //{value: "red", done: false}
console.log(interator.next()) //{value: "blue", done: false}
console.log(interator.next()) //{value: true, done: false}
console.log(interator.next()) //{value: undefined, done: true}
```
### 传递参数给迭代器
```js
function * createIterator(){
    let first = yield 1;
    console.log('first',first)
    let second = yield first *3;
    console.log('second',second)
    yield second * 5
}
let iterator = createIterator()

console.log(iterator.next(3)) //第一个next()中的参数不起作用
console.log(iterator.next(2))
console.log(iterator.next(4))
console.log(iterator.next())
/*
{value: 1, done: false}
first 2
{value: 6, done: false}
second 4
{value: 20, done: false}
{value: undefined, done: true}
*/
```
1. 执行第一个next(),第一个next()比较特殊，传进来的参数被忽略
2. 执行完第一个'yield 1'后，代码被暂停
3. 执行第二个next(),next()中传递的参数2作为'let first = yield 1'中的返回值，赋值给first变量，所以运行'yield first *3'的结果是2x3=6
4. 执行第三个next(),next()中传递的参数4作为second的值，以此类推
   
### 使用生成器、迭代器实现异步任务运行
执行异步任务的传统方法是创建一个带有回调的函数，可以很完美的解决数量比较少的任务；但是如果有多层嵌套回调，或者按顺序执行一系列任务时，使用生成器和迭代器可以完成异步任务的实现，yield指令可以起到很好暂停代码执行的作用
1. 创建一个调用生成器并启动迭代器的函数
2. 调用该函数，传入期望执行任务的生成器
   ```js
   function run(taskDef){
       //调用生成器，得到迭代器
       let task = taskDef()
        //启动任务
       let result = task.next()
       // 使用递归 保证迭代器的调用
       function step(){
           if(!result.done){
               result = task.next()
               step()
           }
       }
       //开始处理
       step()
   }

    run(function*(){
        console.log(1);
        yield;
        console.log(2);
        yield;
        console.log(3);
    })
    /*
    1
    2
    3
    */
   ```
