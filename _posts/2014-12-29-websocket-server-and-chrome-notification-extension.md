---
layout: post
title:  "自制简单的 WebSocket 服务器 + Chrome 通知推送插件"
date:   2014-12-29
categories: ["Chrome", "发布", "拿程序说话"]
permalink: /archives/websocket-server-and-chrome-notification-extension/
excerpt_separator: <!--more-->
---

<div align="center">
<img src="https://raw.githubusercontent.com/laobubu/Chrome-WebSocket-Notification-Extension-Example/master/icon128.png" alt="Chrome-WebSocket-Notification-Extension-Example">
</div>

虽然有点炒冷饭了，但是看在博客多日没更新和今天早上糟糕的复变函数考试的份上，就拿出来写一写吧。

WebSocket 的出现使得通过 HTTP 和服务器建立长连接成为可能，这也就方便了编写各种推送程序。另外， Chrome 提供了一套通知中心 API，可以像手机（或者各种坑爹软件）那样跳出推送消息。本文就如何实现一个简单的 NodeJS WebSocket 服务器和 Chrome 通知推送插件讲一下。

<!--more-->

内容有点长。如果是在我网页端阅读的话，目录在左下角，鼠标移过去就能看见……

## 一个基于 NodeJS 的 WebSocket 服务器

### 起步

这里为了省事，就使用了 NodeJS 作为服务器程序。安装 NodeJS，然后利用 `npm install websocket` 安装上 websocket包[^npmwebsocket] 即可。

接下来就是喜闻乐见的编写代码过程。NodeJS 极大的简化了编写，可以创建一个 `index.js` 文件作为 WebSocket 服务器程序：

```javascript
var WebSocketServer = require('websocket').server;
var http = require('http');

var server = http.createServer(function(request, response) {
    console.log((new Date()) + ' 收到普通 HTTP 请求：' + request.url);
    response.writeHead(404);
    response.end();
});
server.listen(8080, function() {
    console.log((new Date()) + ' 服务器已开始运行于端口 8080');
});

wsServer = new WebSocketServer({
    httpServer: server,
    autoAcceptConnections: false
});
```

上述代码在 8080 端口创建了一个简单的 HTTP 服务器，而且对于每一个普通 HTTP 请求，全部直接返回 404。随后在这个 HTTP 服务器上创建了一个 WebSocketServer （WebSocket 服务器）。浏览器在请求创建 WebSocket 连接的时候都是走 HTTP 服务器的端口号的，只是请求头会和普通 HTTP 请求不太一样。至于分辨出那些特殊请求并加以处理的事情，我们交给 WebSocketServer 即可。

### 接受和拒绝连接

一个 WebSocket 连接请求除了 origin（例如 `ws://lab.laobubu.net:8000/work1`）之外还有 protocol（协议，例如 `my-protocol`）。这两者都是用于确定这个连接是用来干什么的。比如一个聊天程序下可以存在“登录认证协议”和“消息通知协议”等等，然后还可以不同的聊天室有不同的 origin。

接着添加代码。当有连接请求时，会触发这个事件。此时你可以决定要不要接受这个连接……

```javascript
wsServer.on('request', function(request) {
    if (!verifyRequest(request)) {
      request.reject();   
      console.log((new Date()) + ' 拒绝连接，其 origin 为 ' + request.origin + ' 。');
      return;
    }

    var connection = request.accept('my-protocol', request.origin);
    console.log((new Date()) + ' 连接已接受，其 origin 为 ' + request.origin + ' 。');

    /* ... */
});
```

对于上面的 `request` 有什么可以用的信息和方法，你可以参考 [WebSocketRequest 的文档](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketRequest.md) 。

这里补充一下，其中 `request.requestedProtocols` 是客户端请求的协议类型，这可能是一个字符串，也可能是一个字符串数组（万一接受多种类型的协议呢）。

还有就是 `request.accept` 对于一个请求只能来一次，而且接受的协议名称必须存在于 `requestedProtocols` 里面，否则服务器接受了连接，客户端也会翻脸（断开连接）的。

### 收发数据

一旦接受了请求，就会创建一个连接（上文的 `connection` ），而发送和接收数据，都在这个里面进行。

接着写程序，就放到上面 `/* ... */` 处。首先是让它在收到消息时能做出反应：

```javascript
connection.on('message', function(message) {
    if (message.type == 'utf8') {
        console.log((new Date()) + ' 收到文本: ' + message.utf8Data);
        /* 这里可以写对收到的文本的处理程序 */
    }
});
```

这里有一条 `message.type == 'utf8'` 是因为一条消息可以是字符串（UTF-8编码的），也可以是 binary 原始数据。对于 binary 可以用 `message.binaryData` 读取。

然后就是发送数据。使用 `connection.sendUTF(str)` 可以简单地发送字符串数据。要发送 JSON ？没问题！使用 `connection.sendUTF(JSON.stringify( your_json_object ))` 即可。

这里我们做的是推送消息。推送应该在连接创建后就开始进行。现在没有什么数据库啥的说法，我们就杜撰一条消息，在连接建立2秒后发送到客户端吧。紧接着上面的代码之后编写：

```javascript
setTimeout(function(){
	var payload = 
	{
		title:	"紧急通知",
		author:	"laobubu",
		time:	(new Date()).toString(),
		data:	"不知道要说啥就是刷存在感而已啦",
		url:	"http://laobubu.net"
	}
	connection.sendUTF(JSON.stringify(payload));
}, 2000);
```

于是在连接创建之后2秒，一条 JSON 消息就会发送到客户端。

不用担心文字内容太长会被截断。每一条文字消息都以 `0x00` 开始，以 `0xFF` 结束。客户端会自动处理这个问题并将一段段的文字拼接起来的，你只要管发送和接收自己的东西就OK了。

如果还需要其他的应用，在 [WebSocketConnection 说明文档](https://github.com/theturtle32/WebSocket-Node/blob/master/docs/WebSocketConnection.md) 中有详尽的使用说明。

### 断开连接

断开有三种可能，一种是客户端断的，一种是服务器主动断开的，一种是 `Gr3at Fire Wa11 is watching you` 。

服务器主动断开很简单，直接调用 `connection.close()` 即可。

客户端断开的话，首先 `connection.connected` 会变成 false，然后就是会触发一个 `close` 事件。可以接着上面的加这段代码：

```javascript
connection.on('close', function(reasonCode, description) {
    console.log((new Date()) + ' 连接已断开');
    /* 其他善后工作 */
});
```

### 试一试？

我准备了 [一个简单的测试页](http://lab.laobubu.net/ws.html)。将你的地址和协议名写进去，点击 CONNECT 就可以了。

接下来该试一试编写…

## 一个 WebSocket 客户端

只要浏览器支持，这一切都很简单。为了偷懒，我就拿上述测试页的代码来讲了，反正很易懂。

### 创建连接

假定你已经准备了 `var websocket;` 来保存这个连接。

```javascript
websocket = new WebSocket('ws://lab.laobubu.net:8000', 'echo-protocol'); 
websocket.onopen = function(evt) {
    console.log('已成功连接！');
}; 
```

如果连接失败，会触发 `error` 事件并调用 `onerror`，紧接着会触发 `close` 事件并调用 `onclose` 函数。

### 发送数据

```javascript
websocket.send('要发送的文本');

//或者发送一个 JSON对象 过去？
websocket.send(JSON.stringify( your_json_object ));
```

### 接收数据

数据到来时会触发 `websocket` 的 `message` 事件。

```javascript
websocket.onmessage = function(evt) {
    var data = evt.data;
    console.log('收到文本: ' + data);
}; 
```

当然可以将文本转换成 JSON 对象，使用 `JSON.parse(evt.data)` 函数即可得到它。

### 连接关闭

你可以主动的调用 `websocket.close()` 来关闭连接。

连接被关闭时，会触发 `close` 事件并调用 `onclose` 函数。我们可以像这样检测到它：

```javascript
websocket.onclose = function(evt) {
    console.log('连接断开！');
}; 
```

如果建立连接时失败，会触发 `error` 事件并调用 `onerror`，紧接着会触发 `close` 事件并调用 `onclose` 函数。

当然为了做一个时刻和服务器通信的玩意儿，可以考虑在连接断开后立即重新连接，这样就能保证随时在线了。

## 一个 Chrome 扩展程序

### 获得弹窗权限

好吧，其实不完全是弹窗。Chrome 提供有关通知（notifications）的 API，效果就是在屏幕右下角（或者右上角）会跳出来一个框。就像各种国产软件的广告一样。

要使你的扩展程序能够跳出那个框，你需要修改你的扩展的 `manifest.json` 文件，在权限那里像这样写明你的 WebSocket 地址的 HTTP 形式（例如 ws 为 http，wss 为 https）和通知功能的权限：

```json
"permissions": [
    "http://lab.laobubu.net:8000/",
    "notifications"
],
```

### 弹一条消息试试看

根据 Chrome API指南[^chromenotificationapi]，一旦拥有权限，可以像这样跳出一个简单的文字窗口：

```javascript
var options = {
    type: "basic",
    title: "重要通知",
    message: "其实就是刷存在感而已。",
    iconUrl: "icon128.png" //图标的URL，可选
};
chrome.notifications.create("这条通知的ID", options, function(notificationId){
    //创建成功后的回调函数
});
```

当然上面的 `"这条通知的ID"` 可以为空字符串，这样的话 Chrome 会随机生成一个；也可以为已经存在通知的 ID，这样的话 Chrome 会把已经存在的那个先关闭，再创建。

### 点击通知时的反应

添加这样的代码，可以在通知框被点击的时候做出反应：

```javascript
chrome.notifications.onClicked.addListener(function(notificationId){
    console.log('有一条通知被点击了，其 ID 为：' + notificationId);

    //点击后立即清除这条通知
    chrome.notifications.clear(notificationId, function(wasCleared) {});
});
```

### 装扮你的通知

写不动了，你还是看 Google Chrome 官方文档[^chromenotificationapi] 吧……

### 将 WebSocket 客户端的程序糊过来

这一步就比较好玩了。将上面的客户端程序复制过来，然后修改 `onmessage` 那里的程序，将收到的 JSON 对象里的信息跳出来。

至于掉线自动重连什么的，那是前面说过的了。

## Blog is cheap. Show me the code.

好吧，上成品。

[github:laobubu/Chrome-WebSocket-Notification-Extension-Example](https://github.com/laobubu/Chrome-WebSocket-Notification-Extension-Example)

### 说明

一个简单的 Chrome 结合 WebSocket 的通知功能扩展

其中使用的 WebSocket 测试服务器来自 [http://lab.laobubu.net/ws_create.html](http://lab.laobubu.net/ws_create.html) 。

其中准备了简单的 `WSKeeper` 类来创建自动重连的 WebSocket 连接，以及 `NManager` 类来创建和管理通知。

### 偷懒方案

你可以直接使用这个扩展，然后修改成你自己的通知扩展。具体操作如下：

1. 修改 `manifest.json` 中的地址
2. 修改 `background.js` 中在 `Main Code` 以下的代码
3. 修改图标

### WSKeeper类

创建自动重连的 WebSocket 连接，具体用法如下：

```javascript
var ws = new WSKeeper(
	"ws://lab.laobubu.net:8000",	//websocket url
	"json-message",					//websocket protocol
	function onMessage(obj) {		//websocket JSON message handler
		//handle JSON object
	}
);
```

如果要断开连接，使用 `ws.disconnect()` 即可。

### NManager类

负责创建和管理通知。

```javascript
var nman = new NManager("sample-prefix-");	//prefix must be unique
```

或者清除一条通知

```javascript
nman.remove(notificationId)
```

使用这个类可以简单粗暴地创建文字通知。这个函数返回的是通知的 ID

```javascript
nman.create("标题", "文字内容", {})
```

也可以玩一些花样：

```javascript
var options = {
	url:	"",	//(可选) 点击提醒时自动打开的 URL
	buttons: [	//(可选) 按钮，数组
		{title: "Button1",	onclick: function(notificationId, index){alert("Hello from message #"+nid)} },
		{title: "Button2", 	url: "http://laobubu.net"}
		//一个按钮对象必须有 title （按钮文字）
		//可选 iconUrl 表示按钮图标 URL
		//可选 onclick 即按钮被按下时的回调函数，或者 url 表示按钮被按下时打开的 URL
	]
};
nman.create(obj.title, obj.data, options);
```

[^npmwebsocket]: **NPM: websocket** [https://www.npmjs.com/package/websocket](https://www.npmjs.com/package/websocket)
[^chromenotificationapi]: **Chrome Notifications API** [https://developer.chrome.com/apps/notifications](https://developer.chrome.com/apps/notifications)