---
# 文章标题
title: "浏览器的Http请求重发机制"
# 文章副标题
subtitle: ""
# 文章日期
date: 2018-04-26T10:26:50+08:00
# 标签
tags: ["浏览器", 'http']
# 分类
categories: ["浏览器"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

> 前言：近期由于业务需要写了一个逻辑，主要是给几万个用户群发消息，前端发送请求后，后台会加一个时间间隔分批发送，全部发送完毕后再回包。结果就出现了一个现象：**每一条消息都被重复群发了两次。**经过慢慢分析后台的日志打印时间，再根据对浏览器的行为经验，最后自然是找到了原因所在。

## 原因分析
从日志上看第二次开始发送的时间恰好和第一次开始发送的时间距离一分钟，基本就可以确定是因为接口一分钟内没有回包，然后浏览器又重新发送了一次请求过来，稍微验证了下果真如此。

但是浏览器为什么会在等待了一分钟就会自动帮我们重新发送请求？

根本不科学，浏览器应该不会没有原因的这么秀操作

## http规范
于是google搜了一把类似问题，发现大家都碰到过，原因前辈也都找出来了，和http规范有关：

> If an HTTP/1.1 client sends a request which includes a request body, but which does not include an Expect request-header field with the "100-continue" expectation, and if the client is not directly connected to an HTTP/1.1 origin server, and if the client sees the connection close before receiving any status from the server, the client SHOULD retry the request.

意思即，如果发送一个请求到服务器端，但是请求头里面不包含“ 100-continue ”，并且客户端没有直接连接到原始的 HTTP/1.1 服务器，此时，如果客户端在接收到服务器发送的 HTTP 状态之前发现服务器主动关掉连接，那么客户端应该重试请求。

## 真正原因
接下来就是查Nginx配置了，看了下Nginx默认配置的超时时间就是一分钟
```nginx
  server {
   send_timeout 60s;
}
```
这样看就完全满足上述重发请求的条件了：

- 请求头里面不包含“ 100-continue ”
- 客户端没有连接到原始HTTP服务器（Nginx做了反向代理）
- Nginx超过60s主动关掉了连接

## 总结
这里在客户端开发其实需要注意下，一般请求很少有超过60s不能回包的，为了避免踩这种坑，当我们有类似的这种服务器需要很长时间才能处理完数据的时候，尽量就可以提前回包，让服务器自己继续处理后续的数据。

至于客户端要怎么显示和提示用户等待，应该就很好做了。


