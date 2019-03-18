---
title: 写chrome插件前必须要知道的
date: 2019-03-05 15:29:08
tags: chrome
---
### 介绍
浏览器插件本质上就是利用前端的html\css\javascript等技术，借用浏览器对外提供的API，实现各种不同的功能

### 插件组成
首先先看一下我们要开发的插件的各个文件组成  

manifest.json :我们在项目的根文件必须要有一个命名为manifest.json的文件，它是整个插件的功能入口，用来告诉浏览器插件的一些基本信息，及需要加载和执行的资源文件等，里面包含一些必填项：
- name 插件的名字
- version 插件版本
- manifest_version manifest的版本
- description 插件的描述(选填)
```js
{
    "name": "hello extension",
    "description": "Base Level Extension",
    "version": "1.0",
    "manifest_version": 2,
}
```
- icons 在扩展程序管理界面，呈现的图标，可以设置不同的尺寸的选项，用来适配不同的场景
```js
{
    "icons": {
        "16": "images/extension_icon16.png",
        "32": "images/extension_icon32.png",
        "48": "images/extension_icon48.png",
        "128": "images/extension_icon128.png"
      },
}
```
- background 在扩展程序被开启时，会首先触发执行background 选项中指定的js资源，并且一直存在于扩展程序的整个生命周期，直到扩展程序被关闭或者删除，background一直被用来做任务和状态的控制管理工作
```js
// scripts 子选项指定了需要执行的js文件的路径及文件名
{
    "background": {
        "scripts":["background.js"],
        "persistent": false
    }
}
```
background.js 通常最外层使用监听事件`chrome.runtime.onInstalled.addListener(()=>{})`初始化插件，监听插件安装成功后，会触发对应的逻辑开始工作
- permissions 属性值为一个数组，申请chrome API的权限，只有在这个选项中申请了才能使用（比如通过 XMLHttpRequest 跨域请求数据、访问浏览器选项卡（tabs）、获取当前活动选项卡（activeTab）、浏览器通知（notifications）、设置插件激活规则（declarativeContent）、类似localStorage的存储（storage）等）
```js
{
    "permissions": [
        "http://xxx.com/api"
        "tabs",
        "activeTab",
        "notifications",
        "declarativeContent",
        "storage"
    ],
}
```

#### 与用户实现UI交互的功能主要有两个：browser_action、page_action 两者选择其中一个
- browser_action 提供的功能面向所有网站，插件图标一直是激活的状态
  ```js
  {
      "browser_action": {
        "default_popup": "popup.html"        // 指定click插件图标时，展示的页面
        "default_icon": "images/icon.png", // 指定浏览器工具栏展示的插件的图标 
        "default_title": "display tooltip" // 当鼠标悬浮在插件图片上方时，展示的文字，叫tooltip；如果没有这个配置选项，则悬浮时显示menifest.json中的name选项值
    }
  }
  ```
- page_action 只针对对应目标网站提供的功能，插件会在background页设置激活插件的规则，只有满足条件的网页的插件才是激活的，其他网站插件是不可用的，插件的icon是灰的
  ![](https://ws1.sinaimg.cn/large/e4d30300ly1g0s0byu0w2j20mw01y3yx.jpg)
  在`background script`里，`chrome.runtime.onInstalled.addListener(()=>{}) `初始化中使用`chrome.declarativeContent `定义插件被激活的规则
    ```js
    {
        "page_action":{
            "default_popup": "popup.html", // 指定click插件图标时，展示的页面
            "default_icon": "hello_extension.png" // 指定浏览器工具栏展示的插件的图标 
            "default_title": "display tooltip"
    },
    }
    ```
    ```js
        chrome.runtime.onInstalled.addListener(function() {
        // Replace all rules ...
        chrome.declarativeContent.onPageChanged.removeRules(undefined, function() {
        // With a new rule ...
        chrome.declarativeContent.onPageChanged.addRules([
            {
            // That fires when a page's URL contains a 'g' ...
            conditions: [
                new chrome.declarativeContent.PageStateMatcher({
                pageUrl: { urlContains: 'g' }, //url的内容中包含字母g的，插件才会被激活
                })
            ],
            // And shows the extension's page action.
            actions: [ new chrome.declarativeContent.ShowPageAction() ]
            }
        ]);
        });
    });
    ```
#### popup.html
popup.html 是使用browser_action或者page_action 指定的default_popup选项，表示点击插件图标会触发的html资源，其与普通的html页面的区别就是不能使用内联script，其他都一样
it can contain links to stylesheets and script tags, but does not allow inline JavaScript.

#### Content Script 对当前web页面注入扩展程序代码，也就是在当前web页面的上下文环境中，可以访问页面的DOM信息并且修改DOM，具有操作Chrome API的权限，并且可以把信息传递给父级插件
1. content script工作在一个独立的空间，它使得不会访问和调用web page或者其他content script中定义的变量和函数，两者唯一共享的是DOM
2. Content Script与Page action 或Browser action通信的方式是通过传递Message的方式，正确的机制应该是：
   - Page action 或Browser action 中的js使用`chrome.tabs.sendMessage()`发布一个请求信息，传递出去当前的tab页id及其他信息
   - 注意：从Page action 或Browser action发布消息到Content Script必须用`chrome.tabs.sendMessage()`，从Content Script发布简单的信息到Page action 或Browser action 可以使用`chrome.runtime.sendMessage()`
   ```js
    chrome.tabs.sendMessage(
        tabs[0].id, //当前激活的tab页id
        {greeting: "hello"}, //需要传递的信息
        function(response) { //用来接收反馈的回调函数
        console.log(response.farewell);
    });
   ```
   - Content Script完成初始化，设置`chrome.runtime.onMessage().addListener()`监听事件，等待另一侧请求信息的发布
   ```js
   chrome.runtime.onMessage.addListener(
    function(request, sender, sendResponse) {
        console.log(sender.tab ?
                    "from a content script:" + sender.tab.url :
                    "from the extension");
        if (request.greeting == "hello")
        sendResponse({farewell: "goodbye"});
    });
   ```
   - Content Script将信息通过Message发布订阅的方式，发布出去
   - Page action 或Browser action 中的js`chrome.tabs.sendMessage()`最后一个参数是一个回调函数，用来接收另一侧反馈回来的数据，最后同步到UI上
  ```js
    chrome.tabs.sendMessage()
  ```
3. Content script有两种注入方式：编程式动态注入、声明式注入
4. 编程式动态注入：获取`activeTab`权限后，使用`chrome.tabs.executeScript()`来注入js代码片段，使用`chrome.tabs.insertCSS()`注入css代码片段
   
   ```js
    //manifest.json
    {
      "name": "My extension",
      ...
      "permissions": [
        "activeTab"
      ],
    }
  ```
  
  在Page action 或Browser action 中的js中实行编程式动态注入content script，其访问和更改的就是web page上的DOM属性

  ```js
  chrome.runtime.onMessage.addListener(
    function(message, callback) {
      if (message == “changeColor”){
        chrome.tabs.executeScript({
          code: 'document.body.style.backgroundColor="orange"'
        });
      }
   });
   // 你也可以注入一个文件
   chrome.runtime.onMessage.addListener(
    function(message, callback) {
      if (message == “runContentScript”){
        chrome.tabs.executeScript({
          file: 'contentScript.js'
        });
      }
   });
   ```
5. 声明式注入，在manifest.json中使用content_scripts来定义
  ```js
    {
        "name": "My extension",
        ...
        "content_scripts": [
        {
            "matches": ["http://*.nytimes.com/*"],
            "css": ["myStyles.css"],
            "js": ["contentScript.js"]
        }
        ],
        ...
    }
  ```
  - matches：必填 数组，用来匹配注入Content Script的页面地址。可以使用通配符来指代：'*'指代任意长度字符；'?'指代单个字符
  - css：选填 数组，在页面的任何DOM创建和展示之前，按照数组中定义的顺序，依次注入
  - js：选填 数组，按照数组中定义的顺序，依次注入
  - run_at：Content Script注入的时机
    - 如果是document_start, 文件将在所有CSS加载完毕，但是没有创建DOM并且没有运行任何脚本的时候注入
    - 如果是document_end，则文件将在创建完DOM之后，但还没有加载类似于图片或frame等的子资源前立刻注入
    - 如果是document_idle，浏览器会在document_end和发出window.onload事件之间的某个时机注入。具体的时机取决与文档加载的复杂度，为加快页面加载而优化

  ```js
  {
      "content_scripts" : [{
        "matches" : ["*://*/*"],
        "js" : ["content.js", "jquery.js"],
        "css" : ["style.css"],
        "run_at" : "document_end"
    }],
  }
  ```

- options_page 设置插件的选项页面，配置了此选项后，在插件上鼠标右键时，会有一个‘选项’按钮，点击后会进入options_page对应的页面
  ```js
  {
      "options_page":"options.html"
  }
  ```
  ![](https://ws1.sinaimg.cn/large/e4d30300ly1g0s6ny6sqgj20hs0bagm8.jpg)

- chrome_url_overrides 设置可替换的chrome默认页面
  - newtab：新建标签时打开的页面
  - bookmarks：书签页面
  - history：历史记录
  ```js
  {
      "chrome_url_overrides":{
        "newtab": "newTab.html"
    }
  }
  ```
  ![](https://ws1.sinaimg.cn/large/e4d30300ly1g0s8jgnb5ej2122068q3h.jpg)
### 常用到的Chrome API
#### chrome.tabs
1. chrome.tabs.create(object createProperties, function callback)
创建新的标签。注： 无需请求manifest的标签权限，此方法也可以被使用。
**Parameters**
createProperties ( object )
    - windowId ( optional integer )
    创建新标签的目标窗口。默认是当前窗口 。
    - index ( optional integer )
标签在窗口中的位置。 值在零至标签数量之间。
    - url ( optional string )
标签导航的初始页面。完整的URL 必须包含一个前缀 (如 'http://www.google.com', 不能写为 'www.google.com')。 相对 URL则与扩展所在的页面相对， 默认值为新标签页面。
    - selected ( optional boolean )
标签是否成为选中标签。默认为true。
    - pinned ( optional boolean )
标签是否被固定。默认值为false。
    - callback ( optional function )


    Callback function
    回调 参数 应该如下定义：

    function(Tab tab) {...};
    tab ( Tab )
    所创建的标签的细节，包含新标签的ID。
<br>
2. chrome.tabs.executeScript(integer tabId, object details, function callback)
向页面注入JavaScript 脚本执行
**Parameters**
   1. tabId ( optional integer )
   运行脚本的标签ID；默认为当前窗口所选中的标签。
   1. details ( object )
   要执行的脚本内容，可选code或者file，但不能同时选两者。
       - code ( optional string )
       要执行的脚本代码。
       - file ( optional string )
       要执行的脚本文件。
       - allFrames ( optional boolean )
       true的时候，给所有frame执行脚本。默认为false，只给顶级frame执行脚本。
   1. callback ( optional function )
   所有脚本执行后会被调用的回调。

3. chrome.tabs.query(object queryInfo, function callback)
获取经过特定过滤条件筛选的标签页信息，如果没有特定过滤条件则是获取所有标签页信息
    1. queryInfo 为特定的属性，用来对标签页进行过滤。
        常用的属性有：（所有属性均为boolean值）
        - active 标签在他们的窗口中是否是激活状态
        - currentWindow 标签是否在当前窗口
  
        ```js
            chrome.tabs.query({
                active:true,
                currentWindow:true
            },function(tabs){
                //...
            })
        ```
    2. callback 是获取到特定标签页信息后的回调函数，参数为标签的信息
#### chrome.contextMenus 当前页面，选中内容后右键展示的内容
![](https://ws1.sinaimg.cn/large/e4d30300ly1g0whlgg7g5j20ba0eiwl4.jpg)
1. 在manifest.json中设置权限`{"permissions": ["contextMenus", "storage"]}`
2. 放在background.js中，初始化之后的回调函数中，使用`chrome.contextMenus.create({})`创建内容目录
   ```js
   chrome.runtime.onInstalled.addListener(function() {
    //...
        chrome.contextMenus.create({
            "id": "sampleContextMenu",
            "title": "Sample Context Menu",
            "contexts": ["selection"]
        });
    });
   ```
### 插件安装
1. 进入扩展程序管理界面 chrome://extensions
2. 开启'开发者模式'
   ![](https://ws1.sinaimg.cn/large/e4d30300ly1g0s8xyee82j227k060t9t.jpg)
3. 可以选择'加载已解压的扩展程序'，将本地开发目录上传上去即可
4. 可以使用'打包扩展程序'，生成.ctx后缀的扩展程序，就可以发布到google应用市场了
### 插件调试
#### 注意：
- 每次刷新页面，都会执行browser_action或page_action 指定的popup.html资源
- background指定的js资源只在插件安装时执行，之后就一直存在进程中，不会重复执行
#### 调试步骤
1. 在浏览器工具栏右键插件图标
   ![](https://ws1.sinaimg.cn/large/e4d30300ly1g0s91fijjaj20hs0ba3z4.jpg)
2. 点击'审查弹出内容'，进入控制台
3. 切换到Source栏，即可看到browser_action或page_action 指定的资源，设置断点
4. 切换到Console栏，输入location.reload()，发现原有的页面重新进行了加载，同时browser_action或page_action 指定的资源也重新执行了一遍，并在设置的断点处停下来，等待debug

### 参考链接
[chrome扩展程序](https://developer.chrome.com/extensions)  
[如何从零写一个Chrome扩展](https://www.zhihu.com/question/20179805)