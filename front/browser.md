* [事件](#事件)
  * [事件触发](#事件触发)
  * [事件注册](#事件注册)
  * [事件阻止](#事件阻止)
  * [事件代理](#事件代理)
* [同源策略](#同源策略)
  * [Cookie](#cookie)
  * [DOM](#dom)
  * [AJAX](#ajax)
     * [JSONP](#jsonp)
     * [CORS](#cors)
        * [简单请求](#简单请求)
        * [非简单请求](#非简单请求)
           * [预检请求](#预检请求)
  * [跨文档通信](#跨文档通信)
     * [片段识别符](#片段识别符)
     * [window.name](#windowname)
     * [跨文档通信API](#跨文档通信api)
* [存储](#存储)
  * [Cookie](#cookie-1)
* [渲染机制](#渲染机制)
  * [Load 和 DOMContentLoaded](#load-和-domcontentloaded)
  * [图层](#图层)
  * [重绘（Repaint）和回流（Reflow）](#重绘repaint和回流reflow)
* [JavaScript的执行机制(Event Loop)](#javascript的执行机制event-loop)
  * [同步任务和异步任务](#同步任务和异步任务)
  * [宏任务和微任务](#宏任务和微任务)

# 事件 #
## 事件触发 ##
事件触四个阶段
- window往目标对象父级处传播，遇到注册的捕获事件会触发(捕获阶段)
- 传播到目标对象，触发注册的事件(目标阶段)
  + 如果给一个目标节点同时注册冒泡和捕获事件，事件触发会按照注册的顺序执行。
- 目标对象父级处往window传播，遇到注册的冒泡事件会触发(冒泡阶段)
- 触发目标对象的默认行为(默认阶段)
## 事件注册 ##
通常我们使用 addEventListener 注册事件，该函数的第三个参数可以是布尔值，也可以是对象。<br>
对于布尔值 useCapture 参数来说，该参数默认值为 false 。useCapture 决定了注册的事件是捕获事件还是冒泡事件。<br>
对于对象参数来说，可以使用以下几个属性
 - capture，布尔值，和 useCapture 作用一样
 - once，布尔值，值为 true 表示该回调只会调用一次，调用后会移除监听
 - passive，布尔值，表示永远不会调用 preventDefault 
## 事件阻止 ##
  - 一般来说，我们只希望事件只触发在目标上，这时候可以使用 stopPropagation 来阻止事件的进一步传播。通常我们认为 stopPropagation 是用来阻止事件冒泡的，其实该函数也可以阻止捕获事件。<br>
  - stopImmediatePropagation 同样也能实现阻止事件，还能阻止该事件目标执行别的注册事件。
  - preventDefault 阻止事件关联的默认动作，如果 Event 对象的 cancelable 属性是 fasle，那么就没有默认动作，或者不能阻止默认动作。
## 事件代理 ##
如果一个节点中的子节点是动态生成的，那么子节点需要注册事件的话应该注册在父节点上
事件代理的方式相对于直接给目标注册事件来说，有以下优点
  - 节省内存
  - 不需要给子节点注销事件
# 同源策略 #
浏览器安全的基石是"同源策略"（same-origin policy）。同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。<br>
所谓"同源"指的是"三个相同"。
  - 协议相同
  - 域名相同
  - 端口相同
 
 非同源的受限行为
  - Cookie、SessionStorage、LocalStorage 和 IndexDB 无法读取
  - DOM 无法获得
  - AJAX 请求不能发送
## Cookie ##
Cookie 是服务器写入浏览器的一小段信息，只有同源的网页才能共享。  
浏览器允许通过设置document.domain共享 Cookie，此方法只适用于不同子域(一级域名相同)
## DOM ##
如果两个网页不同源，就无法拿到对方的DOM。典型的例子是iframe窗口和window.open方法打开的窗口，它们与父窗口无法通信。  
设置document.domain属性，就可以规避同源政策，拿到DOM。此方法只适用于不同子域(一级域名相同)
## AJAX ##
同源政策规定，AJAX请求只能发给同源的网址，否则就报错。
有三种方法规避这个限制
  - JSONP
  - CORS
  - WebSocket
### JSONP ###
JSONP是服务器与客户端跨源通信的常用方法。最大特点就是简单适用，老式浏览器全部支持，服务器改造非常小。  
它的基本思想是，网页通过添加一个<script>元素，向服务器请求JSON数据，这种做法不受同源政策限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。
### CORS ###
CORS全称是"跨域资源共享"（Cross-origin resource sharing）。它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。  
满足以下条件就属于简单请求
  - 请求方法是以下三种方法之一
    - HEAD
	- GET
	- POST
  - 人为设置HTTP的头信息不超出以下几种字段
	- Accept
	- Accept-Language
	- Content-Language
	- Last-Event-ID
	- Content-Type：只限于三个值 application/x-www-form-urlencoded、multipart/form-data、text/plain
#### 简单请求 ####
对于简单请求，浏览器直接发出CORS请求。具体来说，就是在头信息之中，增加一个Origin字段。  
如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。
  - Access-Control-Allow-Origin 服务器许可域名
  - Access-Control-Allow-Credentials 该字段可选，它是一个布尔值，表示是否允许发送Cookie。
  - Access-Control-Expose-Headers 该字段可选，可以获取的其他响应头字段信息

如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。另一方面，开发者必须在AJAX请求中打开withCredentials属性  
需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。
#### 非简单请求 ####
非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。
##### 预检请求 #####
"预检"请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。除了Origin字段，"预检"请求的头信息包括两个特殊字段。
  - Access-Control-Request-Method 用来列出浏览器的CORS请求会用到哪些HTTP方法
  - Access-Control-Request-Headers 该字段可选，是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段

服务器响应的CORS相关字段
  - Access-Control-Allow-Origin 服务器许可域名
  - Access-Control-Allow-Methods 它的值是逗号分隔的一个字符串
  - Access-Control-Allow-Headers 该字段可选，是一个逗号分隔的字符串，表明服务器支持的所有头信息字段
  - Access-Control-Allow-Credentials 该字段可选，它是一个布尔值，表示是否允许发送Cookie
  - Access-Control-Max-Age 该字段可选，用来指定本次预检请求的有效期，单位为秒。在此期间，不用发出另一条预检请求。
	
一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都会有一个Access-Control-Allow-Origin头信息字段。
## 跨文档通信 ##
  - 片段识别符
  - window.name
  - 跨文档通信API
### 片段识别符 ###
片段标识符（fragment identifier）指的是，URL的#号后面的部分，通过监听hashchange事件得到通知。
### window.name ###
浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。
### 跨文档通信API ###
跨文档通信 API（Cross-document messaging）。这个API为window对象新增了一个window.postMessage方法，允许跨窗口通信，不论这两个窗口是否同源。

postMessage方法的第一个参数是具体的信息内容，第二个参数是接收消息的窗口的源（origin）

父窗口和子窗口都可以通过message事件，监听对方的消息。
message事件的事件对象event，提供以下三个属性。
  - event.source 发送消息的窗口
  - event.origin 发送消息的窗口的源
  - event.data 消息内容

# 存储 #
| 特性 | Cookie | SessionStorage | LocalStorage | IndexedDB |
| --- | --- | --- | --- | --- |
| 生命周期 | 一般由服务器生成，可以设置过期时间 | 页面关闭就清理 | 除非被清理，否则一直存在 | 除非被清理，否则一直存在 |
| 存储大小 | 4K | 5M | 5M | 无限 |
| 通信 | 每次都会携带在 header 中 | 不参与 | 不参与 | 不参与 |
## Cookie ##
对于 cookie，我们需要注意安全性。

| 属性 | 作用 |
| --- | --- |
| value | 如果用于保存用户登录态，应该将该值加密，不能使用明文的用户标识 |
| http-only | 不能通过 JS 访问 Cookie，减少 XSS 攻击 |
| secure | 只能在协议为 HTTPS 的请求中携带 |
| same-site | 规定浏览器不能在跨域请求中携带 Cookie，减少 CSRF 攻击 |
	
# 渲染机制 #
浏览器的渲染机制一般分为以下几个步骤  
  - 处理 HTML 并构建 DOM 树。
  - 处理 CSS 构建 CSSOM 树。
  - 将 DOM 与 CSSOM 合并成一个渲染树。
  - 根据渲染树来布局，计算每个节点的位置。
  - 调用 GPU 绘制，合成图层，显示在屏幕上。
![渲染机制](../images/paint.png)
CSS 的解析会阻塞脚本的执行，而脚本会阻塞 HTML 的解析。

## Load 和 DOMContentLoaded ##
Load 事件触发代表页面中的 DOM，CSS，JS，图片已经全部加载完毕。<br>
DOMContentLoaded 事件触发代表初始的 HTML 被完全加载和解析，不需要等待 CSS，JS，图片加载。
## 图层 ##
一般来说，可以把普通文档流看成一个图层。特定的属性可以生成一个新的图层。不同的图层渲染互不影响，所以对于某些频繁需要渲染的建议单独生成一个新图层，提高性能。但也不能生成过多的图层，会引起反作用。
## 重绘（Repaint）和回流（Reflow） ##
重绘和回流是渲染步骤中的一小节，但是这两个步骤对于性能影响很大。
  - 重绘是当节点需要更改外观而不会影响布局的
  - 回流是布局或者几何属性需要改变就称为回流
  
  回流必定会发生重绘，重绘不一定会引发回流。回流所需的成本比重绘高的多，改变深层次的节点很可能导致父节点的一系列回流。

# JavaScript的执行机制(Event Loop) #
JavaScript语言的一大特点就是单线程，所有任务需要排队，前一个任务结束，才会执行后一个任务。<br>
## 同步任务和异步任务 ##
为了提高效率，把任务分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。
  - 同步任务 在主线程上排队执行的任务
  - 异步任务 进入 Event Table 并注册回调函数

执行过程
  - 同步和异步任务分别进入不同的执行场所
  - 当指定的事情完成时，Event Table 会将回调函数移入 Event Queue
  - 主线程内的任务执行完毕为空，会去 Event Queue 读取对应的函数，进入主线程执行。

上述过程会不断重复，也就是常说的Event Loop(事件循环)。

## 宏任务和微任务 ##
  - macro-task(宏任务) 包括整体代码script，setTimeout，setInterval
  - micro-task(微任务) Promise，process.nextTick

不同类型的任务会进入对应的 Event Queue <br>
事件循环的顺序，决定js代码的执行顺序。进入整体代码(宏任务)后，开始第一次循环。接着执行所有的微任务。然后再次从宏任务开始，找到其中一个任务队列执行完毕，再执行所有的微任务。
