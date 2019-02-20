---
title: ES6-Set/Map
date: 2019-02-20 14:50:33
tags: ES6
---
### 背景
ES6新增了两种集合类型Set、Map，用来实现不同的功能
- Set 是不包含重复值的列表，常被用来检查某个值是否存在
- Map 是键值对的集合，多用来作为缓存，存储和提取数据。Object的属性类型都会默认转成字符串，比如obj['5']与obj[5]是等价的；Map的属性类型可以其他类型

### Set
1. new Set()创建Set类型
2. add()向Set中添加新的元素
3. size属性返回当前Set列表包含几个元素
4. has()判断当前调用的Set是否包含某个值
5. delete()删除指定元素
6. clear()清空当前Set列表
7. Set不会对元素做强制类型转换，比如5与'5'是两个不同的元素
8. Set列表不存在索引，所以无法像数组一样通过索引获取某一个值，可以使用forEach()依次遍历列表中的元素
9. 对于只想存储对象类型元素的列表，并且担心Set列表中的对象强引用造成内存泄漏等问题，建议使用new WeakSet()创建弱引用的Set列表，要求所有元素必须为对象类型，否则会报错。当Weak Set中的对象在Weak Set之外不存在任何引用时，该对象就会被销毁，也就不会存在干扰垃圾回收，内存泄漏的问题(与传统的Set用法还有些区别，感兴趣的可以自己进一步了解下)
#### 将数组转换成Set，可以删除掉重复的元素
```js
let arr = [1,2,3,4,5,5,5,4]
let set = new Set(arr)
console.log(set) // Set(5) {1, 2, 3, 4, 5}
set.forEach((value) => {
    console.log(value) // 1,2,3,4,5
})
```
#### 将Set转换成数组，使用扩展运算符
```js
let arr = [1,2,3,4,5,5,5,4]
let set = new Set(arr)
let array = [...set]
console.log(array) // [1, 2, 3, 4, 5]
```
#### 常用来删除数组重复数组的方法
```js
[...new Set(array)]
```
### Map
Map类型是键值对的有序列表，键和值可以为任何类型，特别是对键的类型不会强制转换(内部管理键的值是使用Object.is()全等于的方式，因此5与'5'是两个不同的键)
1. new Map()创建Map列表
2. set(key,value)创建键值对
3. get(key)索引键值
4. 使用对象作为键，每一个不同的对象实例都不相等 Object.is({},{}) //false
   ```js
   Object.is({},{}) //false

   let map = new Map(),
       key1 = {},
       key2 = {};
    map.set(key1, 'Tome');
    map.set(key2,'1234');
    console.log(map.get(key1)) //Tom
    console.log(map.get(key2)) // 1234
   ```
5. has(key) 判断Map列表是否存在该键值对
6. delete(key) 删除对应的键值对
7. clear() 清空本Map列表
8. size属性，返回Map列表内有几个键值对
9. new WeakMap() 创建弱引用Map，当Weak Map中键在Weak Map之外不存在任何引用时，该对象就会被销毁
#### Map初始化数组
```js
let map = new Map([['name','Tom'],['age',20],[3,99]]) // 数组中有多个含有两个子元素的子数组，子数组中第一个元素作为Map列表的key，第二个值作为Map列表的value
console.log(map.size) //3
console.log(map.get('name')) //Tom
console.log(map.get('age')) //20
console.log(map.get(3)) //99
```
forEach(value, key, ownerMap)方法
- value 当前位置键值对的值
- key 当前位置键值对的键
- ownerMap 被调用的Map列表
```js
map.forEach((value, key, ownerMap)=>{
console.log(key + ' ' + value)
console.log(ownerMap === map)
})

name Tome
true
age 20
true
3 99
true
```

