---
# 文章标题
title: "跨域方案CORS使用总结"
# 文章副标题
subtitle: ""
# 文章日期
date: 2016-11-22T20:57:50+08:00
# 标签
tags: ["cors", '跨域']
# 分类
categories: ["浏览器"]
# 草稿标识，true代表正式环境下不显示
draft: false
---


> 前言

最近不管是在公司项目还是自已的个人项目中用到的跨域方案都不再是JSONP了，而是变成了CORS（Cross-origin resource sharing），简称为跨域资源共享，是一个W3C标准。

为什么跨域方案从JSONP的盛极一时到现在的CORS大行其道，还是因为CORS做了JSONP做不到的事，我们都知道JSONP只能使用GET形式请求数据，当传输的数据量比较大的时候，就需要使用POST形式才能搞定，于是CORS就出来了。下面是我折腾过程中的一些总结。

## 一、简介
CORS需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10。而使用CORS的方法及其简单，首先在浏览器端不需要有任何处理，只需要写出普通的ajax代码即可，浏览器会根据url参数来确定ajax请求是否跨源，从而自动向服务器发送附加的头信息，所以只需要在服务器端进行处理（发送响应标头）就行。

## 二、简单请求
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）

只要同时满足以下两大条件，就属于简单请求。

（1) 请求方法是以下三种方法之一：
- HEAD
- GET
- POST

（2）HTTP的头信息不超出以下几种字段：
- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

凡是不同时满足上面两个条件，就属于非简单请求

#### 客户端：
对于简单跨域请求，浏览器在进行处理的时候会自动在头信息之中，添加一个Origin字段。
```js
GET /cors HTTP/1.1
Origin: http://blog.billyh.cn
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
这个字段说明了本次请求来自哪个源（协议 + 域名 + 端口），服务器根据这个来确定是否同意请求。

#### 服务器：
服务器相应的处理代码（以express为例）：
```js
app.all("*", function (req, res, next) {
  res.header('Access-Control-Allow-Origin', '*');//-必须-设置来源域名，*表示所有的都可以请求
  res.header('Access-Control-Allow-Credentials', 'true');//-可选-设置是否允许发送cookie
  res.header('Access-Control-Expose-Headers', 'Foo');//-可选-设置自定义的字段
});
```
上面字段中第一个是必须设置的，否则浏览器将会抛出错误导致跨域失败，其他为可选（剩余这两个字段我还没尝试过，不介绍啦）

## 三、非简单请求
对于不同时满足上面所说的条件的额请求就是非简单请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json的。

对于非简单请求，浏览器会默认自动帮你发一个OPTIONS请求，到服务器端请求服务器确认该请求的合法性，服务器端必须得有相应的响应标头处理该请求，并认真返回200响应，然后浏览器才会再次发出正常的、你需要的请求。

#### 客户端：
比如我有这样一段ajax请求：
```js
$.ajax({
    type:"PUT",
    url:"http://www.baidu.com",
    dataType:"json",
    headers:{token:'wuSheng'},
    success:function:(data){
        //处理代码
    }
})
```
上面代码的请求方法为PUT，并且有一个自定义的头信息token，自然为非简单请求了，而这是浏览器的自动处理为：
```js
OPTIONS /cors HTTP/1.1
Origin: http://blog.billyh.cn
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: token
Host: www.baidu.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
#### 服务器：
同样以express为例：
```js
app.all("*", function (req, res, next) {
  res.header('Access-Control-Allow-Origin', '*');
//-必须-设置来源域名，*表示所有的都可以请求
  res.header("Access-Control-Allow-Headers", "Content-Type,Content-Length, Authorization, Accept,X-Requested-With,token");
//-必须-表明服务器支持的所有头信息字段
  res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
//-必须-表明服务器支持的所有跨域请求的方法
  if (req.method == 'OPTIONS') {
    res.send(200);//如果请求方法为OPTIONS则返回200
  } else {
    next();
  }
});
```
通过以上的响应处理即可让非简单请求成功跨源