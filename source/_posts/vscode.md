---
title: vscode常用插件与配置
date: 2019-01-28 14:36:41
tags: 工具
---

### 背景
前端开发现在最火的编辑器当属VSCode了，其中丰富的插件应用可以帮助提高我们的开发效率和拓展编辑器的能力。
前几天因为mac pro有一批电脑硬盘有问题，于是便重新换了硬盘，导致之前安装的app及所有相关配置都需要重新装一遍，借此机会也总结下我安装的插件应用及配置

### 常用插件
#### 主题类
介绍两款比较受欢迎的主题，更换主题的方法
1. Atom One Dark Theme
2. Dracula Official
   切换主题的方法
   ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzmc08q50kj20k60dyaiq.jpg)
   ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzmc4d23sdj214e0tmjv8.jpg)

#### 编程工具类
1. Auto Close Tag: 自动补齐html便签
2. Auto Rename Tag: 当你修改标签名时，会同时更改初始和结束标签
3. Babel ES6/ES7: 高亮ES6/ES7语法
4. Beautify: 美化javascript, JSON, CSS, Sass, and HTML 
5. Bookmarks: 为需要的代码行做记录，并支持直接跳转
6. Bracket Pair Colorizer: 当代码嵌套太多时，不同组[]/{}等符号具有不同的颜色，便于区分同组符号
7. CSS Peek: 标签中设置的class，支持直接跳转到对应css区域
8. EditorConfig for VS Code: 定义和维护代码风格（行尾结束符、缩进风格等）,,[参考链接](https://juejin.im/post/5b50269b5188251a9f249ed2)
9. ES7 React/Redux/GraphQL/React-Native snippets: 提供语法快捷方式
10. filesize: 在编辑器底部的状态栏中会展示当前编辑文件的size大小，点击具体大小值会展示具体信息
    ![](https://ws1.sinaimg.cn/large/e4d30300ly1fznhn748a9j21dg0ne0wq.jpg)
11. Git History: 查看文件、某一代码行的git更改历史
12. Git History Diff: 也是用来查看文件、某一代码行的git更改历史，同时可以切换具体某次提交，查看所有文件的diff(个人觉得比Git History使用更加方便，功能也更加强大)
    ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzni9ca9xij20ls0ogat3.jpg)
13. Import Cost: 展示引入的依赖包的大小
    ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzniijeclej20xo08k40f.jpg)
14. Markdown All in One: markdown语法的快捷方式
15. Markdown Preview Enhanced: 增加markdown文件的预览功能，可以实时查看效果
16. minapp: 微信小程序标签、属性的智能补全(支持原生小程序、mpvue 和 wepy 框架)
17. npm Intellisense 智能import已安装的npm pkg，显示的pkg为package.json中dependencies中的依赖
    ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzohgq91uwj214k0aaq5u.jpg)
    ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzohgzhlcij20xo0futb1.jpg)
18. open in browser: 针对html文件，支持右键打开浏览器浏览
19. Path Autocomplete: 引用路径的智能提示
20. Prettier - Code formatter: 代码格式化
21. TODO Highlight: 对部分代码块做todo标记，用来提醒自己在最后上线前需要处理的事宜
    在User Settings 中增加todohighlight的配置
    ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzorix5hwmj20xc0u80x3.jpg)
    具体配置可以参考[官网wiki](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)
    command+shift+P 唤起命令行面板
    ![](https://ws1.sinaimg.cn/large/e4d30300ly1fzorn6m3nvj20y204ydgm.jpg)

### 配置
#### shell命令使用zsh：
重写User Settings文件即可
```js
"terminal.integrated.shell.osx": "zsh"
```
![](https://ws1.sinaimg.cn/large/e4d30300ly1fzos207xtkj21g20hcwn9.jpg)

#### 更改文件保存方式
设置文件保存的方式，提高开发效率，也防止忘记保存的情况出现。可以直接重写User Settings的配置
```js
"files.autoSave": "onFocusChange"
```
也可以进入Settings的可视化配置界面，自定义配置自己的偏好，比如字体、间距等
![](https://ws1.sinaimg.cn/large/e4d30300ly1fzos9rr3z6j21ha0uyn20.jpg)