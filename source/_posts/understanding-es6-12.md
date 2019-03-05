---
title: ES6-Modules
date: 2019-02-25 14:44:01
tags: ES6
---
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0qs1z3avzj216w0lggpe.jpg)
### 模块的导入导出
1. 导出export
   ```js
   export let a = 111 //导出基本类型
   export function add(){} // 导出类
   export class Rectangle{} // 导出函数

   function multiply(num1, num2){
       return num1*num2
   }
   export { multiply }
   ```
2. 导入import
   ```js
   import { num } from './example.js' //导入单个模块
   import * as Example from './example.js' //导入所有模块并合并在一个命名空间对象上
   ```
   注：
   - 当代码执行到import导入语句时，会根据引用的路径，执行导出模块的代码，也就是完成导出模块的实例化，并且将该模块实例保存在内存中，同一应用的任何模块引用此导出模块时，都会复用已在内存中的模块实例，而不会重新进行导出模块的实例化
   ```js
   // 多次import同一模块，只在实例化一次模块，之后都会复用这个模块实例
   import {sum} from './example.js'
   import {multiply} from './example.js'
   import {add} from './example.js'
   ```
   - 导入的常量是导出变量的只读引用
   ```js
   import { num } from './example.js' // 不能给num重新赋值，同时当example.js中的num值变化时，这里的num也会跟着变化
   num = 'Tom' //error报错
   ```
3. 重命名导入和导出名称
   ```js
   function sum (num1, num2){
       return num1 + num2
   }
   // 导出模块
   export {sum as add}

    // 引入模块
   import {add as sum } from './example.js'
   ```
4. 导出模块默认值
   ```js
   //法一
   export default function add(){}
   //法二
   function add(){}
   export default add
   //法三
   function add(){}
   function multiply(){}
   export {add as default, multiply} //导出多个，并设置默认导出
   ```
5. 导入模块默认值
   ```js
   import sum from './example.js' //不适用大括号{}，名字也可以随便取
   import sum ,{ multiply } from './example.js' // 既有默认值sum，也有非默认值，非默认值要放到{}中，并且名称要和导出模块的名称保持一致
   //上面的写法等价于下面的写法
   import {default as sum, multiply} from './example.js'
   ```
6. 导入再导出
   ```js
   import { add } from './example.js'
   export { add }

   //合并到一句的写法：
   export { add } from './example.js'
   //从example.js中导入add，并命名为sum导出
   export { add as sum } from './example.js'
   ```
7. 无绑定的导入(比如css文件等，不需要明确指定引入的模块内容)
   ```js
   import './style.css'
   ```
### Web浏览器中模块执行的顺序
新型浏览器针对script标签增加type='module'的属性，来支持模块化语法的资源引入
1. 遇到`<script type='module'>` 的标签时，会立即下载对应的js资源，下载完后会解析js，遇到import语句会取请求对应的资源
2. 下载和解析`<script type='module'>` 资源的时候，并不会影响html接下来的解析
3. 在html解析加载完成之前只会做下载和解析工作，不会执行js代码，执行js的顺序会严格按照`<script>`显式引入和import隐式引入的顺序来依次执行
#### 案例分析：
```js
<script type='module' src='module1.js'></script>

<script type='module'>
    import {sum} from './example.js'
    let result = sum(1,2) 
</script>

<script type='module' src='module2.js'></script>
```
正确的顺序：
1. 下载并解析module1.js
2. 递归下载并解析在module1.js中使用import引入的资源
3. 解析内联模块
4. 递归下载并解析内联模块中使用import引入的资源example.js
5. 下载并解析module2.js
在html解析完成之前，都不会有任何JS执行，当html解析完成之后，DOMContentLoaded触发之前，开始执行js，执行js的顺序如下：
1. 递归执行module1.js中引入的资源(按照import的顺序依次执行)
2. 执行module1.js
3. 递归执行内联模块import的资源
4. 执行内联模块的代码
5. 递归执行module2.js中引入的资源(按照import的顺序依次执行)
6. 执行module2.js

注意：`<script type='module'></script>`采取的就是defer属性的方式，因此设不设置defer没有任何影响；但是设置async属性后，会**按照async的模式，下载解析完，并且所有包含的import资源也都下载解析后按照引入顺序立即执行js**
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0iwdd6h68j227m0j642z.jpg)