---
title: js高级特性
date: 2019-04-26 15:26:04
tags: JavaScript
---
### 作用域
- let\const 声明的变量，在块级作用域中可见
- var声明的变量，在函数作用域中可见
- 最外层是全局作用域，node环境中全局对象为global，浏览器环境中全局对象为window
- 作用域链，当当前作用域内查找不到同名变量时，会去外层作用域中查找
- 函数作用域是静态作用域(词法作用域)，也就是说函数作用域的嵌套父子关系是在函数定义的时候，就确定了，而不是在函数调用的时候才确定
```js
var scope = 'top';
var f1 = function() { 
    console.log(scope);
};
f1(); // 输出 top
var f2 = function() { 
    var scope = 'f2'; 
    f1();
};
f2(); // 输出 top
```
### 闭包
当一个函数返回它内部定义的一个函数时，就产生了一个闭包， 闭包不但包括被返回的函数，还包括这个函数的定义环境
```js

var generateClosure = function() { 
    var count = 0;
    var get = function() {
        count ++;
        return count; 
    };
    return get; 
};
var counter1 = generateClosure(); 
var counter2 = generateClosure(); 
console.log(counter1()); // 输出 1 
console.log(counter2()); // 输出 1 
console.log(counter1()); // 输出 2 
console.log(counter1()); // 输出 3 
console.log(counter2()); // 输出 2
```
> 上面这个例子解释了闭包是如何产生的:counter1 和 counter2 分别调用了 generate- Closure() 函数，生成了两个闭包的实例，它们内部引用的 count 变量分别属于各自的 运行环境。**我们可以理解为，在 generateClosure() 返回 get 函数时，私下将 get 可 能引用到的 generateClosure() 函数的内部变量(也就是 count 变量)也返回了，并在内存中生成了一个副本**，之后 generateClosure() 返回的函数的两个实例 counter1 和 counter2 就是相互独立的了。

#### 闭包的用途：实现对象的私有成员
```js
var generateClosure = function() { 
    var count = 0; //私有变量
    var get = function() {
        count ++;
        return count; 
        };
    return get;
};
var counter = generateClosure(); 
console.log(counter()); // 输出 1 
console.log(counter()); // 输出 2 
console.log(counter()); // 输出 3

```
在函数中通过返回一个子函数，对外部暴露一个访问和更新内部变量的接口，外部不能直接访问函数作用域内的变量，完成了对变量私有化的隐藏
### 对象
#### 访问对象属性成员方式
- 点运算符（obj.a）
- 关联数组（obj['a']）,使用关联数组的优势是可以使用变量作为索引，这对于我们前期不知道对象属性key的时候非常有用
#### 上下文对象this
- 上下文对象就是 this 指针，即被调用函数所处的环境
- JavaScript 的任何函数都是被某个对象调用的，在最外部全局作用域中，调用它的是全局对象
- this 指针不属于某个函数，而是函数调用时所属的对象。也就是说谁调用的函数，this就指向谁
  ```js
  var someuser = { 
      name: 'Tom', 
      func: function() {
        console.log(this.name); 
        }
    }; 
    var foo = { 
        name: 'foobar'
    };
    someuser.func(); // 输出 Tom

    foo.func = someuser.func; 
    foo.func(); // 输出 foobar

    name = 'global';
    func = someuser.func; 
    func(); // 输出 global
  ```
  在 JavaScript 中，本质上，函数类型的变量是指向这个函数实体的一个引用，在引用之间赋值不会对对象产生复制行为。我们可以通过函数的任何一个引用调用这个函数，不同之处仅仅在于上下文

  - 改变函数中上下文对象的方式：
    - call，以依次列举参数形式，传递给被调函数参数 func.call(thisArg[, arg1[, arg2[, ...]]])，**返回被调函数执行结果**
    - apply，以传递数组的形式，传递给被调函数参数 func.apply(thisArg[, argsArray])，**返回被调函数执行结果**
    - bind，func.bind(thisArg[, arg1[, arg2[, ...]]]) **返回绑定新的上下文对象的函数**，以后调用时都是固定绑定的上下文对象，会重复调用时适合使用bind方法

### 原型
使用原型和构造函数初始化对象属性的区别：
```js
//原型方式
function Person() {
}
Person.prototype.name = 'Tom'; 
Person.prototype.showName = function () {
    console.log(this.name); 
};
var person = new Person(); 
person.showName();

//构造函数方式
function Person(){
    this.name = 'Tom';
    this.showNmae = function(){
        console.log(this.name);
    }
}
var person = new Person(); 
person.showName();
```
1. 继承方式不同：原型方式，子对象通过原型链可以直接访问父对象的属性；构造函数内定义的属性，子对象必须显式调用父对象才能访问
2. 原型定义的属性和方法是子对象实例共用一套，减少重复开销；构造函数内定义的属性，每次新建一个对象实例，都会在内存中重新创建一套，实例之间不共享
3. 对象的成员函数最好使用原型方式定义，可以减少内存开销；定义在构造函数内部，多个对象会重复创建，同时可能会有运行时闭包开销
4. 一般的变量属性成员，在构造函数中定义，保证每个实例对象数据的独立性，因为在原型上定义意味着所有实例共享这一个，任何对象的主动更改会影响到其他实例对象

#### 原型链
1. JavaScript 中有两个特殊的对象: Object 与 Function，它们都是构造函数，用于生
成对象
2. Object.prototype 是所有对象的祖先，Function.prototype 是所有函数的原型，包括构造函数
3. JavaScript 中的对象分为三类，一类是用户创建的对象，一类是构造函数对象，一类是原型对象
   - 用户创建的对象，即一般意义上用 new 语句显式构造获得的对象
   - 构造函数对象指的是普通的构造函数，即通过 new 调用生成普通对象的那个函数
   - 原型对象 特指构造函数 prototype 属性指向的对象
4. 这三类对象中每一类都有一个 __proto__ 属 性，它指向该对象的原型，从任何对象沿着它开始遍历都可以追溯到 Object.prototype
5. 构造函数对象有 prototype 属性，指向原型对象；通过该构造函数创建对象时，被创建的普通对象的 __proto__ 属性将会指向构造函数的 prototype 属性，也就是该普通对象的原型对象
6. 原型对象有 constructor 属性，指向它对应的构造函数
```js
function Foo() {
}
Object.prototype.name = 'My Object'; Foo.prototype.name = 'Bar';
var obj = new Object();
var foo = new Foo();
console.log(obj.name); // 输出 My Object
console.log(foo.name); // 输出 Bar
console.log(foo.__proto__.name); // 输出 Bar 
console.log(foo.__proto__.__proto__.name); // 输出 My Object 
console.log(foo. __proto__.constructor.prototype.name); // 输出 Bar
```
![](https://ws1.sinaimg.cn/large/e4d30300ly1g2i5w8jfmuj214k0tsq9o.jpg)
1. 在 JavaScript 中，继承是依靠一套叫做原型链(prototype chain)的机制实现的。
2. 属性 继承的本质就是一个对象可以访问到它的原型链上任何一个原型对象的属性

### 原型的深拷贝
日常开发中，如果只需要复制基本的属性时(基本类型、对象、数组等，不包含函数与特殊对象new Date()、正则等)，使用JSON序列化再反序列化的方法是最便捷的方式
```js
JSON.parse(JSON.stringify(obj))
```
#### 如果想要支持成员函数的拷贝：
```js
Object.prototype.clone = function() { 
    var newObj = {};
    for (var i in this) {
    if (typeof(this[i]) == 'object' || typeof(this[i]) == 'function') { 
        newObj[i] = this[i].clone();
    } else {
        newObj[i] = this[i];
    } }
    return newObj; 
};

Array.prototype.clone = function() {
    var newArray = [];
    for (var i = 0; i < this.length; i++) {
    if (typeof(this[i]) == 'object' || typeof(this[i]) == 'function') { 
        newArray[i] = this[i].clone();
    } else {
        newArray[i] = this[i];
    } }
    return newArray;
}
Function.prototype.clone = function () {
  var that = this;
  var newFun = function newFun() {
    return that.apply(this, arguments); //这里构成了闭包
  };
  for (var i in this) {
    newFun[i] = this[i];
  }
  return newFun;
}
```
#### 循环引用的深拷贝
- 使用JSON序列化反序列化会报错
![](https://ws1.sinaimg.cn/large/e4d30300ly1g2iled3rujj20y00fstas.jpg)
- 使用上面常规的递归方式会出现死循环，最后栈溢出
#### 解决方案：引入WeakMap结构
WeakMap对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的
遍历对象时使用WeakMap结构存储，遇到循环引用的对象，通过WeakMap.prototype.has(key)与WeakMap.prototype.get(key)来终止循环遍历问题
```js
function isObj(obj) {
    return (typeof obj === 'object' || typeof obj === 'function') && obj !== null
}

function deepCopy(obj, mapObj = new WeakMap()) {
    if(mapObj.has(obj)) return mapObj.get(obj)
    let cloneObj = Array.isArray(obj) ? [] : {}
    mapObj.set(obj, cloneObj)
    for (let key in obj) {
        cloneObj[key] = isObj(obj[key]) ? deepCopy(obj[key], mapObj) : obj[key];
    }
    return cloneObj
}

```
