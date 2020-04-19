---
title: HTTP/2 特性概述
date: 2020-04-18 23:41:56
tags: [HTTP/2]
thumbnail: https://image.linsenx.com/blog/2020-04-18-Xnip2020-04-18_23-49-38.png?imageMogr2/format/jpg/interlace/1/blur/1x0/quality/50|imageslim
toc: true
---

HTTP/2，简称为 **h2**（基于 TLS/1.2 或以上版本的加密链接）或 **h2c**（非加密连接，已被主流浏览器废弃），是 HTTP 协议的第二个主要版本。

HTTP/2 是 HTTP 协议自 HTTP1.1 后发布的首个更新，基于 Google 发明的 HTTP 代替协议 SPDY，并由 httpbis 工作小组开发，该协议2015年2月被批准。

<!--more-->

## HTTP/2 与 HTTP/1.1 比较

HTTP/2 大多数 headers 与 HTTP/1.1 保持一致，因此 HTTP/2 可以完美的向下兼容 HTTP/1.1，但可以通过 HTTP/2 的新特性得到性能上的提升。

在 HTTP/1.1 的基础上，HTTP/2 增加了以下特性：

- 二进制分帧
- 多路复用
- 头部压缩
- 服务端推送

## 二进制分帧

HTTP/2 是基于帧的协议，采用分帧是为了将重要信息封装起来，方便协议解析。而 HTTP/1.1 是文本格式的协议。

下图是 HTTP/2 的帧结构，其中前 9 个字节（Length、Type、Flag、R Stream Identifier）的格式对于每一帧来说都是一致的，所以解析时候只需相应的读取这些字节，就可以知道整个帧的字节数。

![](http://image.linsenx.com/blog/2020-04-18-Untitled.png)

## 多路复用

**队头阻塞：**HTTP/1.1 有一个特新叫做管道化（pipelining），运行一次发送一组请求，但是只能按照发送顺序依次接收响应，因此若有请求应答遇到了问题，之后的请求应答都会被**阻塞**。

**流**：HTTP/2 规范对流(stream)的定义是：“**HTTP/2 连接上独立的、双向的帧序列交换**。”你可以将流看作在连接上的一系列帧，它们构成了单独的 HTTP 请求和响应。

HTTP/2 中，同域名下的所有通信都在一个 TCP 连接上完成，也就是说该 TCP 可以承载任意数量的**流**，由于各个帧之间可以乱序发送（可以通过帧首部的 Stream Identifier 重新组装），因此解决了**队头阻塞**的问题。这大大提高了 TCP 连接的利用率，相较于 HTTP/1.1 中开启多个 TCP 连接达到并行通信的效果，多路复用方案更节省资源。

## 头部压缩

在 HTTP/1.1 中，每次都要发送全量的首部数据，而首部数据大多数都是重复的，这就造成了带宽的浪费。

因此 HTTP/2 使用了 HPACK 算法对头部数据进行了压缩：HTTP/2 中，客户端和服务端维护了“头部表”来记录之前发送的头部数据，对于相同的数据，仅发送一个索引值。

## 服务端推送

服务端推送让服务器具备了在客户端请求资源之前就推送资源的能力，如果合理使用，可以有效降低页面的加载时间。

举例还说就是浏览器原本需要进行 HTML 文档解析，当解析到资源所在位置时才会发起资源请求，而使用了服务端推送就可以在浏览器主动请求外部资源前将资源发送给浏览器。

但是推送也会浪费带宽，因为服务器推送的资源可能已被客户端所缓存，这时候客户端就可以发送 RST_STREAM 帧来拒绝服务器的主动推送。

## Reference

- [https://zh.wikipedia.org/wiki/HTTP/2](https://zh.wikipedia.org/wiki/HTTP/2#HTTP/2%E4%B8%8EHTTP/1.1%E6%AF%94%E8%BE%83)
- [https://zhuanlan.zhihu.com/p/26559480](https://zhuanlan.zhihu.com/p/26559480)
- [HTTP/2基础教程](https://www.ituring.com.cn/book/2020)