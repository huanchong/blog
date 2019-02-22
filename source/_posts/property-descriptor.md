---
title: property-descriptor
date: 2019-02-14 20:50:22
tags:
---
### 属性描述符
我们定义的对象，对应的每一个属性都对应一个‘属性描述符’对象，用来描述该属性的一些特性：
属性描述符有两种形式：**数据描述符或存取描述符**。数据描述符是一个具有值的属性，该值可能是可写的，也可能不是可写的。存取描述符是由getter-setter函数对描述的属性。**属性描述符必须是这两种形式之一；且不能同时是两者**

- 数据描述符独有的：
    - value：改属性的值
    - writable：为true表示该属性的value可以被重写
- 存取描述符独有的：
    - get：获取该属性的访问器函数(getter)，当访问该属性时，该方法会被执行，方法执行时没有参数传入；如果没有 getter 则为 undefined
    - set：该属性的设置器函数(setter)，当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值；如果没有则为undefined，为该属性赋不了值
- 数据描述符和存取描述符都含有的:
    - configurable：为ture时，表示当前属性的‘属性表述符’对象可以被更改，该属性可以使用delete删除，否则就是不允许更改
    - enumerable：为true时，表示当前属性可以被枚举，也就是当前属性是否可以在 for...in 循环和 Object.keys() 中被遍历出来
- (value和writable)与(get和set)是不共存的，只要定义了其中一个，就定下来了该描述符的性质是数据描述符还是存取描述符
  错误的案例：
  ```js
  var o = {};
  Object.defineProperty(o, "a", { get : function(){return 1;},value : 12 } );
  // Uncaught TypeError:Invalid property descriptor.Cannot both specify accessors and a value or writable attribute

  var o = {};
    Object.defineProperty(o, "a", { get : function(){return 1;}, 
                                    configurable : true } );
    Object.defineProperty(o, "a", {value : 12}); // 这里重写了属性描述符，将其变为数据描述符,且没定义的configurable 不会重写
    console.log(o.a) // 12
    d = Object.getOwnPropertyDescriptor(o, "a");
    console.log(d) //{value: 20, writable: false, enumerable: false, configurable: true}
  ```

#### Object.defineProperty(obj, prop, descriptor)创建或修改对象的单个属性
用来定义或修改obj对象上prop属性的descriptor属性描述符
- obj：要在其上定义属性的对象。
- prop：要定义或修改的属性的名称。
- descriptor：object,将被定义或修改的属性描述符
当描述符中省略某些字段时，这些字段将使用它们的默认值。拥有布尔值的字段的默认值都是false。
value，get和set字段的默认值为undefined。一个没有get/set/value/writable定义的属性被称为“通用的”，并被“键入”为一个数据描述符

```js
var bValue, o={};
Object.defineProperty(o, "b", {
    set : function(newValue){
        bValue = newValue;
    },
    enumerable : true,
    configurable : true
});
console.log(o.b) //undefined  没有显示定义getter函数，因此默认值为undefined，所以使用o.b调用时为undefined
o.b= 11
console.log(bValue) // 11 给属性赋值时，调用了setter函数，将值赋值给了bValue
```
```js
var o={};
Object.defineProperty(o, "b", {
    get : function(){
        return 22;
    },
    enumerable : true,
    configurable : true
});
console.log(o.b) //22
o.b = 123
console.log(o.b) //22 因为没有显示定义setter，所以默认值为undefined，赋值动作无效，因此o.b不会变化
```
#### Object.defineProperties(obj, props) 创建或修改对象的多个属性
```js
var  Obj = {
  "a":10
};
Object.defineProperties(Obj,{
  "a1":{
    get: function(){return this.a+1;},
    set: function(val){ this.a = val;}
  },
  "a2":{
    get: function(){return this.a+"test";},
    set: function(val){this.b = val}
  }
});
console.log(Obj.a1); //11
console.log(Obj.a2); //1test
Obj.a1 = 3;
Obj.a2 = 'hello';
console.log(Obj.a1); //4
console.log(Obj.a2); //3test
```
#### 修改属性
1. Writable 属性
当writable属性设置为false时，该属性被称为“不可写”
```js
var o = {}; // 创建一个新对象

// 在对象中添加一个属性与数据描述符的示例
Object.defineProperty(o, "a", {
  value : 37,
  writable : false,
  enumerable : true,
  configurable : true
});
o.a = 123
console.log(o.a) //37 赋值无效
```
2. Enumerable 特性
enumerable定义了对象的属性是否可以在 for...in 循环和 Object.keys() 中被枚举
```js
var o = {};
Object.defineProperty(o, "a", { value : 1, enumerable:true });
Object.defineProperty(o, "b", { value : 2, enumerable:false });
Object.defineProperty(o, "c", { value : 3 }); // enumerable defaults to false
o.d = 4; // 如果使用直接赋值的方式创建对象的属性，则这个属性的enumerable为true

for (var i in o) {    
  console.log(i);  
}
// 打印 'a' 和 'd' (in undefined order)

Object.keys(o); // ["a", "d"]

o.propertyIsEnumerable('a'); // true
o.propertyIsEnumerable('b'); // false
o.propertyIsEnumerable('c'); // false
```
3. Configurable 特性
configurable特性表示对象的属性是否可以被删除及属性描述符是否可以被修改
```js
var o = {};
Object.defineProperty(o, "a", {value : 12, 
                                configurable : false } );
Object.defineProperty(o, "a", {value : 20}); // Uncaught TypeError: Cannot redefine property: a

delete o.a // false 删除失败
console.log(o.a) // 12 还是可以正常访问
```
#### 我们熟悉的默认写法
```js
var o = {};

o.a = 1;
// 等同于 :
Object.defineProperty(o, "a", {
  value : 1,
  writable : true,
  configurable : true,
  enumerable : true
});

var o = { get foo() { return 17; } };
var p = Object.getOwnPropertyDescriptor(o, 'foo')
console.log(p)
// {
//   configurable: true,
//   enumerable: true,
//   get: /*the getter function*/,
//   set: undefined
// }

// 另一方面，
Object.defineProperty(o, "a", { value : 1 });
// 等同于 :
Object.defineProperty(o, "a", {
  value : 1,
  writable : false,
  configurable : false,
  enumerable : false
});


```
#### Object.getOwnPropertyDescriptor(obj, prop) 获取指定对象自有属性的属性描述符
obj：需要查找的目标对象
prop：目标对象内属性名称（String类型）
```js
var o, d;

o = { get foo() { return 17; } };
d = Object.getOwnPropertyDescriptor(o, "foo");
// d {
//   configurable: true,
//   enumerable: true,
//   get: /*the getter function*/,
//   set: undefined
// }

o = { bar: 42 };
d = Object.getOwnPropertyDescriptor(o, "bar");
// d {
//   configurable: true,
//   enumerable: true,
//   value: 42,
//   writable: true
// }

o = {};
Object.defineProperty(o, "baz", {
  value: 8675309,
  writable: false,
  enumerable: false
});
d = Object.getOwnPropertyDescriptor(o, "baz");
// d {
//   value: 8675309,
//   writable: false,
//   enumerable: false,
//   configurable: false
// }
```