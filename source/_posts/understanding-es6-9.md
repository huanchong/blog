---
title: ES6-Class
date: 2019-02-22 10:54:41
tags:
---
### 背景
在ES6之前，官方都不支持类的语法，很多第三方类库都通过模拟实现了类似类的用法，最终ES6引入了类
### 类的介绍
#### 类的声明
##### ES6之前的写法：称为自定义类型
```js
function Person(name){
    this.name = name
}
Person.prototype.sayName = function(){
    return this.name
}
```

##### ES6的写法：
```js
class Person {
    //构造器
    constructor(name){
        this.name = name //自有属性
    }
    sayName(){
        return this.name
    }
}
let person = new Person('Tom')
console.log(person.sayName()) //Tom
console.log(person instanceof Person) //true
console.log(person instanceof Object) //true

console.log(typeof Person) //function
console.log(typeof Person.sayName) //undefined
console.log(typeof Person.prototype.sayName) //function
```
- `console.log(typeof Person) //function` 通过打印结果可以看到，ES6对类的声明类似ES5自定义类型写法的语法糖，本质上class定义的Person仍然是一个函数
- `console.log(typeof Person.sayName) //undefined  console.log(typeof Person.prototype.sayName) //function` 通过打印结果可以看到，ES6类中声明的方法实际也是挂载到原型上的，这个ES5自定义类型中定义的方法是一样的

类表达式：
```js
let Person = class {
    //构造器
    constructor(name){
        this.name = name //自有属性
    }
    sayName(){
        return this.name
    }
}

//具名类表达式
let Person = class Person2{
    //构造器
    constructor(name){
        this.name = name //自有属性
    }
    sayName(){
        return this.name
    }
}
**这里的Person2只能在类的方法中被访问到**
```
##### 使用ES6声明类的优势
1. 类声明不会被提升，类声明类似于let，在程序执行到达类声明之前，是处于暂时性死区中的，访问会报错。ES5中的函数定义法，函数是可以被提升到当前作用域顶部的
2. 类声明中的代码强制在严格模式下运行'use strict'，且不能退出
3. 类的所有方法不可枚举，ES5的自定义类型只能通过Object.defineProperty()来定义属性描述符规定其不可枚举
4. 调用类生成实例时，必须使用new关键词，会自动调用类中的构造器constructor函数。不适用new 调用类会报错；这是自定义类型的函数是没办法规避的
5. 类的方法不能使用new 关键词调用，会抛出错误
6. 试图在类的方法中重写类名(相当于const类名)，会抛出错误；在类外部可以重写类名(相当于let类名)

#### 类是一等公民
在编程中，能被当做值来使用的就称为一等公民，也就是说，就像函数那样，能够作为函数的参数、函数的返回值、能用来给变量赋值等操作
```js
function createClass(classDef){
    return new classDef()
}
let obj = createClass(class {
    sayHi(){
        console.log('Hi')
    }
})

obj.sayHi() //Hi
```
#### 类的访问器属性
```js
class Person {
    constructor(name){
        this.value = name
    }
    get name(){
        console.log('getter')
        return this.value
    }
    set name(name){
        console.log('setter')
        this.value = name
    }
}
let person = new Person('Tom')
console.log(person.name)
person.name = 'Lisa'
console.log(person.name)
/*
getter
Tom
setter
getter
Lisa
*/
```
#### 为类添加生成器方法(定义Symbol.iterator)
```js
class Collection {
    constructor(){
        this.items= []
    }
    *[Symbol.iterator](){
        yield *this.items.values() //this.items.values()是数组内置的迭代器，这里相当于把this.items的生成器合并在类生成器中了
    }
}
let collection = new Collection()
collection.items.push(1)
collection.items.push(2)
collection.items.push(3)

for(let value of collection){
    console.log(value)
}
/*
1
2
3
*/
```
#### 静态成员
我们之前已经实验过，在类中定义的方法，是把此方法属性加到类的原型上的，类的实例也都继承了这些方法，可以访问；
如果不希望实例可以访问，只能被类自身访问，则需要使用静态成员
##### ES5传统的实现写法
```js
function Person(name){
    this.name = name
}
//添加在原型上，所有实例都可以访问
Person.prototype.getName = function(){
    return this.name
}
静态方法，实例访问不到
Person.sayHi = function(){
    console.log('Hi')
}
```
#### ES6写法
```js
class Person {
    constructor(name){
        this.name = name
    }
    //添加到原型上，所有实例都可以访问
    getName(){
       return this.name 
    }
    static sayHi(){
       console.log('Hi') 
    }
}
let person = new Person('Tom')
console.log(person.getName()) //Tom
// person.sayHi() // 抛出异常 person.sayHi is not a function
Person.sayHi()//Hi
```
#### 使用extends关键词完成类的继承
```js
class Rectangle {
    constructor(length, width){
        this.length = length;
        this.width = width;
    }
    getArea(){
        return this.length * this.width
    }
}
class Square extends Rectangle {
    constructor(length){
        //使用super()调用父类的构造函数
        super(length, length)
    }
}
let square = new Square(3)
console.log(square.getArea()) //9
console.log(square instanceof Square) //true
console.log(square instanceof Rectangle) //true
```
- 继承父类的类叫派生类，派生类如果显式定义了构造器，就必须在访问this之前，要显式地使用super()调用父类的构造器，完成父类初始化this，并通过继承传递给派生类，否则在派生类中访问this时会报错
- 在派生类中才可以使用super关键词，指代父类，super()表示先初始化父类的构造器，然后派生类继承父类的this，所以在派生类中this也就可以访问了
  ```js
  class Rectangle {
    constructor(){
		console.log('call parent')
        this.length = 10
        this.width = 5
    }
    getArea(){
        return this.length * this.width
    }
  }
    class Square extends Rectangle {
        constructor(){
            console.log('call child1')
            super()
            console.log('call child2')
        }
        getValue(){
            console.log('length',this.length)
            console.log('width',this.width)
        }
    }
    let square = new Square()
    square.getValue()
    /*
    call child1
    call parent
    call child2
    length 10
    width 5
    */
  ```
- 如果派生类没有显式定义构造器，则系统在初始化的时候会默认调用super()，并使用创建实例时传递的所有参数
  ```js
  class Square extends Rectangle {
      
  }
  // 等价于
  class Square extends Rectangle {
      constructor(..args){
          super(...args)
      }
  }
  ```
> 注意：如果派生类与父类初始化的参数完全相同，可以选择省略constructor的形式
```js
class Rectangle {
    constructor(length, width){
        this.length = length;
        this.width = width;
    }
    getArea(){
        return this.length * this.width
    }
}
class Square extends Rectangle {
    
}
let square = new Square(3,3)
console.log(square.getArea()) //9
```
> 如果派生类与父类的初始化参数不一致，就得自己写构造器constructor，手动调用super()，否则会得到意想不到的结果
```js
class Rectangle {
    constructor(length, width){
        this.length = length;
        this.width = width;
    }
    getArea(){
        return this.length * this.width
    }
}
class Square extends Rectangle {
    
}
let square = new Square(3) //这里只传了1个参数，派生类使用默认的构造器传递参数，与父类构造器参数不一致，导致有些参数为undefined，调用计算方法是肯定为NaN
console.log(square.getArea()) //NaN
```
#### 静态成员也会被继承
```js
class Person {
    constructor(name){
        this.name = name
    }
    static sayHi(){
       console.log('Hi') 
    }
}
class Student extends Person {

}
Person.sayHi()//Hi 
Student.sayHi()//Hi 
```
