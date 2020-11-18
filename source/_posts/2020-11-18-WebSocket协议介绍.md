---
title: WebSocket协议介绍
date: 2020-11-18 17:54:17
tags: websocket
categories: 技术
---


WebSocket 是由HTML5提出的一个独立的协议标准。WebSocket可以分为协议（Protocol）和API两部分，分别由IETF和W3C制定了标准。它跟HTTP协议基本没有关系，只是为了兼容现有浏览器的握手规范而对 HTTP 协议的一种补充。更加确切的说 WebSocket 利用了 HTTP 协议来建立连接，仅此而已。
<!--more-->

# WebSocket 协议介绍

现在 WebSocket 协议已经成了标准协议，所有主流浏览器都已经很好的支持其基础功能。

WebSocket 协议实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯的目的。它与HTTP一样通过已建立的TCP连接来传输数据，但是它和HTTP最大不同是：

1. WebSocket是一种双向通信协议。在建立连接后，WebSocket服务器端和客户端都能主动向对方发送或接收数据，就像Socket一样；

2. WebSocket需要像TCP一样，先建立连接，连接成功后才能相互通信。

![握手示意图](https://s3.ax1x.com/2020/11/18/Dnp0vn.png)

上图对比可以看出，相对于传统HTTP每次请求-应答都需要客户端与服务端建立连接的模式，WebSocket是类似Socket的TCP长连接通讯模式。一旦WebSocket连接建立后，后续数据都以帧序列的形式传输。在客户端断开WebSocket连接或Server端中断连接前，不需要客户端和服务端重新发起连接请求。在海量并发及客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显。

# WebSocket 协议的建立过程

WebSocket 连接必须由浏览器发起，请求协议是一个标准的HTTP请求（也就是说，WebSocket的建立是依赖HTTP的）。请求报文格式如下：

```javascript
GET ws://localhost:3000/ws/chat HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Origin: http://localhost:3000
Sec-WebSocket-Key: client-random-string
Sec-WebSocket-Version: 13
```
该请求和普通的HTTP请求有几点不同：
1. 其中 HTTP 头部字段 Upgrade: websocket 和 Connection: Upgrade 很重要，告诉服务器通信协议将发生改变，转为 WebSocket 协议。
2. Sec-WebSocket-Key是用于标识这个连接，并非用于加密数据；
3. Sec-WebSocket-Version指定了WebSocket的协议版本。

支持 WebSocket 的服务器端在确认以上请求后，应返回状态码为101 Switching Protocols的响应：

```javascript
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: nRu4KAPUPjjWYrnzxDVeqOxCvlM=
```

该响应代码 101 表示本次连接的HTTP协议即将被更改，更改后的协议就是 Upgrade: websocket 指定的WebSocket协议。

版本号和子协议规定了双方能理解的数据格式，以及是否支持压缩等等。如果仅使用 WebSocket 的 API ，就不需要关心这些。

现在，一个 WebSocket 连接就建立成功，浏览器和服务器就可以随时主动发送消息给对方。消息有两种，一种是文本，一种是二进制数据。通常，我们可以发送JSON格式的文本，这样，在浏览器处理起来就十分容易。

为什么 WebSocket 连接可以实现全双工通信而 HTTP 连接不行呢？实际上HTTP协议是建立在TCP协议之上的，TCP协议本身就实现了全双工通信，但是HTTP 协议的请求－应答机制限制了全双工通信。WebSocket连接建立以后，接下来的通信就不使用 HTTP 协议了，直接互相发数据。

安全的 WebSocket 连接机制和 HTTPS 类似。首先，浏览器用 wss://xxx 创建 WebSocket 连接时，会先通过HTTPS创建安全的连接，然后，该HTTPS 连接升级为 WebSocket 连接，底层通信走的仍然是安全的 SSL/TLS 协议。

# WebSocket 、Ajax 轮询 和 Long Poll(长轮询) 原理解析

说到 WebSocket ，那就不得不说说 Ajax 轮询 和 Long Poll(长轮询)。

Ajax 轮询 和 Long Poll(长轮询) 都是 HTTP 请求的应用，都属于非持久连接。

首先来说说 Ajax 轮询。Ajax 轮询的原理非常简单，让浏览器每隔一定的时间就发送一次请求，询问服务器是否有新信息。

Long Poll(长轮询) 其实原理跟 Ajax 轮询 差不多，都是采用轮询的方式，不过采取的是阻塞模型，也就是说，客户端发起连接后，如果没消息，服务器不会马上告诉你没消息，而是将这个请求挂起（pending），直到有消息才返回。返回完成或者客户端主动断开后，客户端再次建立连接，周而复始。我们可以看出Long Poll(长轮询) 已经具备了一定的实时性。

上面这两种应用都是非常消耗资源。Ajax 轮询需要服务器有很快的处理速度和资源。Long Poll(长轮询) 需要有很高的并发，也就是说同时连接数的能力。同时也受到客户端的连接数限制，比如老早的IE6，客户端同事连接数为2。尽管如此，在过去 Ajax 轮询 和 Long Poll(长轮询) 还是有广泛的应用，特别是实时聊天，短消息推送等方面， Long Poll(长轮询) 是除了 Flash 之外唯一的选择。

相对于 HTTP 连接的非持久连接来说，WebSocket 则是持久连接。

上面已经说了 WebSocket 是类似 Socket 的TCP长连接通讯模式。一旦 WebSocket 连接建立后，后续数据都以帧序列的形式传输。而且浏览器和服务器就可以随时主动发送消息给对方，是全双工通信。在海量并发及客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显。

# WebSocket API

WebSocket 客户端的 API 和流程非常简单：创建 WebSocket 对象，然后指定 open、message等事件的回调即可。其中 message 是客户端与服务器端通过WebSocket通信的关键事件，想要在收到服务器通知后做点什么，写在message事件的回调函数里就好了：

# WebSocket 构造函数

WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例。

```javascript
var ws = new WebSocket('ws://localhost:8080/ws');
```

执行上面语句之后，客户端就会与服务器进行连接。实例对象的所有属性和方法参见 <https://developer.mozilla.org/en-US/docs/Web/API/WebSocket>

# 参考资料
<https://www.css88.com/archives/9293>
