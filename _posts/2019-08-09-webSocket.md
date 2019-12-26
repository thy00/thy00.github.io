---
title:  socket.io
categories: 
 - 通信协议
tags: 学习
---



定义：标准RFC6455，并由RFC7936补充

> RFC 是一系列以编号排定的文件，它由一系列草案和标准组成。几乎所有互联网通信协议均记录在 RFC 中，例如 HTTP 协议标准

**轮询**

**comet**   http长连接

## **webSocket**

第一次发送http请求升级协议为webSocket

升级标识为http header的upgrade字段

> GET /chat HTTP/1.1
> Host: server.example.com
> **Upgrade: websocket**
> **Connection: Upgrade**
> Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
> Origin: http://example.com
> Sec-WebSocket-Protocol: chat, superchat
> Sec-WebSocket-Version: 13
>
> In compliance with [RFC2616], header fields in the handshake may be sent by the client in any order, so the order in which different header fields are received is not significant.
>

返回消息

> HTTP/1.1 **101** Switching Protocols
>  Upgrade: websocket
>  Connection: Upgrade
>  Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
>  Sec-WebSocket-Protocol: chat

传输数据帧

![img](https://user-gold-cdn.xitu.io/2019/8/9/16c73c0f44b9d25a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



[webSocket 协议]: https://juejin.im/post/5d4cbc0cf265da038f47fa37



## socket io

socket io不仅仅是websocket的封装，支持很多不支持websocket的浏览器，使用升降级的机制

engine.io是socket io的底层库，写了一套协议进行升降级以及通讯的规范

当使用websocket协议通讯的时候

>  4[^message]2[^event]0[^confirm]["XX[^6]",{XX[^7]}]

engine.io

> 0 open：从服务端发出，标识一个新的传输方式已经打开。
>
> 1 close：请求关闭这条传输连接，但是它本身并不关闭这个连接。
>
> 2 ping：客户端周期性发送ping，服务端响应pong。注意这个与uwsgi自带的ping/pong不一样，uwsgi里面发送ping，而浏览器返回pong。
>
> 3 pong：服务端发送。
>
> 4 message：实际发送的消息。
>
> 5 upgrade：在转换transport前，engine.io会发送探测包测试新的transport（如websocket）是否可用，如果OK，则客户端会发送一个upgrade消息给服务端，服务端关闭老的transport然后切换到新的transport。
>
> 6 noop：空操作数据包，客户端收到noop消息会将之前等待暂停的轮询暂停，用于在接收到一个新的websocket强制一个新的轮询周期。

socket发送消息到对应客户端是维护了一个消息队列

> socket.io服务器发送消息要基于engine.io消息包装，所以归结到底还是调用的engine.io套接字中的`send()`方法。engine.io为每个客户端都会维护一个消息队列，发送数据都是先存到队列里面待拉取，websocket除了探测帧之外的其他数据帧也都是通过该消息队列发送。



socket io通信过程的参数说明

### 请求连接

> http://127.0.0.1/socket.io/?EIO[^1]=3&transport[^2]=polling&t[^3]=MAkXxBR

返回结果:

response header

> HTTP/1.1 200 OK
> content-type: application/octet-stream   [^4]
> connection: keep-alive
> set-cookie: io=c1996d8b-5720-4697-977b-3d5224725d86
> server: netty-socketio/1.7.17
> access-control-allow-origin: http://183.6.56.146:8000
> access-control-allow-credentials: true
> content-length: 118

response body

> ÿ0{"sid":"c1996d8b-5720-4697-977b-3d5224725d86","upgrades":["websocket"],"pingInterval":25000,"pingTimeout":15000}

payload其实包含两个packet，第一个packet是engine.io的OPEN消息，类型为0，它的内容为pingInterval，pingTimeout，sid等；第二个packet类型是4(message)，而它的数据内容是0，表示socket.io的CONNECT。而其中的看起来乱码的部分实则是前面提到的payload编码中的长度的编码\x00\x01\x00\x09\xff和\x00\x02\xff。

### 升级协议

> http://127.0.0.1/socket.io/? EIO=3&transport=polling&t=MAkXxEz&sid[^5]=9c54f9c1759c4dbab8f3ce20c1fe43a4

升级到websocket协议进行数据传输

### 发送消息

> ws://127.0.0.1/socket.io/?EIO=3&transport=websocket&sid=9c54f9c1759c4dbab8f3ce20c1fe43a4

request body

> 4[^message]2[^event]0[^confirm]["XX[^6]",{XX[^7]}]

[^1]: engine.io版本号
[^2]: 传输协议
[^3]: 唯一字符串，使用yeast生成
[^4]: 字节流方式传输数据
[^5]: engine.io 用来识别客户端的标志
[^6]: event名称，服务端监听
[^7]: event的具体数据
[^message]: engine.io通信的标识，4代表服务端和客户端交换数据
[^event]: 事件的标志
[^confirm]: 事件必须确认，返回对应的确认消息
