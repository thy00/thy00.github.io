---
title:  webSocket
categories: 
 - 通信协议
tags: 学习
---



定义：标准RFC6455，并由RFC7936补充

> RFC 是一系列以编号排定的文件，它由一系列草案和标准组成。几乎所有互联网通信协议均记录在 RFC 中，例如 HTTP 协议标准

**轮询**

**comet**   http长连接

**webSocket**

第一次发送http请求升级协议为webSocket

升级标识为http header

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

### 



[]: https://juejin.im/post/5d4cbc0cf265da038f47fa37	"webSocket 协议"