---
title: ES6-Object
date: 2019-02-14 16:58:56
tags: ES6
---
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0qs1z3avzj216w0lggpe.jpg)
### 对Object的扩展
#### 对象的类别
1. 普通对象
2. 奇异对象
3. 标准对象
4. 内置对象

#### 对象字面量语法的扩展
1. 属性初始化器的简写语法(和对象key同名的变量名可以省略)
```js
let time = 500, name = 'Tom'
let obj = {
    name,
    time
}
```
2. 属性方法的简写语法
```js
var person = {
    name: 'Tom',
    sayHello(){
        console.log('hello world')
    }
}
```
3. 使用[]包含计算属性变量名，特别用于**遍历数组创建对象时属性名是未知的情况**
```js
var arr = ['name','age','weight']
var value = ['Tome','18','130']
var person = Object.create(null)
for(let key in arr){
    person = Object.assign(person, {
        [arr[key]]: value[key]
    })
    
}
console.log(person) // {name: "Tome", age: "18", weight: "130"}
```
#### 新增的Object方法
1. Object.is()
   用于比较严格相等，修复了'==='部分怪异行为
   ```js
   console.log(+0 === -0) //true
   console.log(Object.is(+0, -0)) //false

   console.log(NaN === NaN) //false
   console.log(Object.is(NaN, NaN)) //true
   ```
2. Object.assign()
   - 混入(Mixin),第一个参数为属性接受者对象，将后面多个供应者对象的属性与方法统一合并到接受者上，并且将接受者对象返回；
   - 命名相同的属性，后者供应者对象的属性会覆盖前面的
   - 供应者的访问器属性会被转换成接受者的数据属性
   ```js
    var o, d, obj={}

    o = { get foo() { return 17; } };
    Object.assign(obj, o)
    console.log(obj) // {foo: 17}

    d = Object.getOwnPropertyDescriptor(obj, "foo");
    console.log(d) // {value: 17, writable: true, enumerable: true, configurable: true}
   ```
   使用Object.assign()后，o的get属性被转换成obj的数据属性，也就是d.value = 17
#### 更改对象的原型
ES5中添加了Object.getPrototypeOf()方法来获取指定对象的原型
ES6中添加了修改指定对象原型的方法：Object.setPrototypeOf(指定对象, 目标原型)，这样设置完后，指定对象就继承了新的目标原型的方法和属性
#### 使用super引用访问对象的原型
super是指向当前对象的原型的一个指针，实际上就是Object.getPrototypeOf(this)的值，使用super引用后，便可以访问原型对象的任何属性和方法
注意：super只能在ES6语法的简写属性方法中使用，否则会报错
```js
// 在ES6简写方法语法中引用super可以正常访问
var person= {
getName(){
	console.log('super',super.constructor())
}}
person.getName() 

// 使用传统的属性方法的语法，引用super会报错
var person= {
getName: function(){
    // Uncaught SyntaxError: 'super' keyword unexpected here
	console.log('super',super.constructor())
}}
person.getName()
```


   
