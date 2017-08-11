# 开发ChromeExtension

## Chrome Extension用什么语言开发的？
*   html + js

## 一个Chrome Extension的组成
以下为一个标准的Chrome Extension的组成元素，其中前两个是基础部分，只要有这两部分一个Chrome Extension就能运行。
后两部分分别运行于不同的Context中, 所以他们之间的相互通信是要通过专门的通信机制来进行的。
### manifest.json
Chrome Extension的描述文件，他描述了Extension的一些属性(例如: 名字，版本，功能描述，使用权限等)
### Sources
包括Extension icon和点击icon后的弹窗文件popup.html
### Background
Background, 运行于Extension Context, 一个浏览器实例对应一个Background运行实例, 它可以使用绝大多数Chrome APIs,用于处理一些Extension事务。
推荐使用[eventPage](https://developer.chrome.com/extensions/event_pages)
### Content Scripts
[Content Scripts](https://developer.chrome.com/extensions/content_scripts), 运行于页面Context, 一个标签对应一个Context Scripts运行实例, 可以通过DOM读取或修改页面信息。
它只可以使用有限的APIs:
*   extension ( getURL , inIncognitoContext , lastError , onRequest , sendRequest )
*   i18n
*   runtime ( connect , getManifest , getURL , id , onConnect , onMessage , sendMessage )
*   storage

## 通信机制
*   popup.html与background.html是同环境的，所以cookie什么的是可以共享的
*   content scripts是与web page同环境的，所以它们的cookie什么的也是共享的
*   而back ground与content scripts这间的通信就可以Simple one-time requests或是Long-lived connections方式相互通信
*   [详细文档](https://developer.chrome.com/extensions/messaging)

## Wapp-Extension开发
### 开发背景
公司的App开发中使用自研的Wapp框架来加载H5页面，这个Wapp框架中包含了一个wapp.js的H5 SDK。而这个SDK与APP原生内容这间有很强的
相关性，需要APP给出一个Ready事件wapp.js中的APIs才能生效，这就导致H5页的开发必需要APP中调试，这给调试增加了很大的困难。为了
能让H5页面的开发者能方便的调试wapp相关的功能，就要开发一个Chrome Extension来为wapp.js在浏览器中运行提供环境。
### 功能开发
主要是两部分，一部分是给用户提供一个登录到APP后台的页面，一部份是对wapp.js进行支持。针对这两个部分的具体开发如下：
*   基础部分，提供APP API的访问支持，包括数据加密解密，跨域访问等
*   登录部分，利用基础部分登录到APP后台，并将用户信息存到Cookie中与eventPage.js共享，当打开一个新的标签后content.js会从
eventPage.js获取到用户信息并存到cookie当中以便页面能够访问到。
*   wapp.js支持部分包括了wappSupport.js和content.js, wappSupport.js通过重写window.open来拦截wapp接口进行具体的功能支持
然后通过content.js将wappSupport.js注入到页面中以便wappSupport.js与wapp之间的互动。