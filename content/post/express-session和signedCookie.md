---
# 文章标题
title: "express-session和signedCookie"
# 文章副标题
subtitle: ""
# 文章日期
date: 2017-6-12T16:41:50+08:00
# 标签
tags: ["NodeJS", 'auth', 'session', 'cookie', 'http']
# 分类
categories: ["http/tcp"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

> hi 这是一篇周总结

## 使用session进行身份认证以及signedCookie

> session运行在服务器端，当客户端第一次访问服务器时，可以将客户的登录信息保存。 
当客户访问其他页面时，可以判断客户的登录状态，做出提示，相当于登录拦截。 
session可以和Redis或者数据库等结合做持久化操作，当服务器挂掉时也不会导致某些客户信息（购物车）丢失。 

### session的工作流程： 
使用 cookie 有一个很大的弊端，cookie 中的所有数据在客户端就可以被修改，数据非常容易被伪造，那么一些重要的数据就不能存放在 cookie 中了，而且如果 cookie 中数据字段太多会影响传输效率。为了解决这些问题，就产生了 session，session 中的数据是保留在服务器端的。

session 的运作通过一个 session_id 来进行。session_id 通常是存放在客户端的 cookie 中，比如在 express 中，默认是 connect.sid 这个字段，当请求到来时，服务端检查 cookie 中保存的 session_id 并通过这个 session_id 与服务器端的 session data 关联起来，进行数据的保存和修改。

这意思就是说，当你浏览一个网页时，服务端随机产生一个 1024 比特长的字符串，然后存在你 cookie 中的 connect.sid字 段中。当你下次访问时，cookie 会带有这个字符串，然后浏览器就知道你是上次访问过的某某某，然后从服务器的存储中取出上次记录在你身上的数据。由于字符串是随机产生的，而且位数足够 多，所以也不担心有人能够伪造。伪造成功的概率比坐在家里编程时被邻居家的狗突然闯入并咬死的几率还低。

session 可以存放在 1）内存、2）cookie本身、3）redis 或 memcached 等缓存中，或者4）数据库中。线上来说，缓存的方案比较常见，存数据库的话，查询效率相比前三者都太低；

express 中操作 session 要用到 express-session (https://github.com/expressjs/session ) 这个模块，主要的方法就是session(options)，express-session会 默认使用内存来存 session，比较方便调试，其中 options 中包含可选参数。

### express-session模块的常用参数: 
- secret:一个String类型的字符串，作为服务器端生成session的签名。 
- name:返回客户端的key的名称，默认为connect.sid,也可以自己设置。 
- resave:(是否允许)当客户端并行发送多个请求时，其中一个请求在另一个请求结束时对session进行修改覆盖并保存。默认为true。但是(后续版本)有可能默认失效，所以最好手动添加。
- saveUninitialized:初始化session时是否保存到存储。默认为true， 但是(后续版本)有可能默认失效，所以最好手动添加。
- cookie:设置返回到前端key的属性，默认值为{ path: ‘/’, httpOnly: true, secure: false, maxAge: null }。
- store: session 的存储方式，默认存放在内存中，也可以使用 redis，mongodb 等。express 生态中都有相应模块的支持。
- genid: 产生一个新的 session_id 时，所使用的函数， 默认使用 uid2 这个 npm 包。
- rolling: 每个请求都重新设置一个 cookie，默认为 false。

**express-session的一些方法:**

- Session.destroy():删除session，当检测到客户端关闭时调用。

- Session.reload():当session有修改时，刷新session。

- Session.regenerate()：将已有session初始化。

Session.save()：保存session。

### 简单的代码示例

```js
//app.js中添加如下代码(已有的不用添加)
var express = require('express');
var cookieParser = require('cookie-parser');
var session = require('express-session');

app.use(cookieParser('sessiontest'));
app.use(session({
    secret: 'sessiontest',//与cookieParser中的一致
    resave: true,
    saveUninitialized:true
}));
```
```js
//修改router/index.js,第一次请求时我们保存一条用户信息。
router.get('/', function(req, res, next) {
    var user={
        name:"Chen-xy",
        age:"22",
        address:"bj"
    }
  req.session.user=user;
  res.render('index', {
      title: 'the test for nodejs session' ,
      name:'sessiontest'
  });
});
```
```js
//修改router/users.js，判断用户是否登陆。
router.get('/', function(req, res, next) {
    if(req.session.user){
        var user=req.session.user;
        var name=user.name;
        res.send('你好'+name+'，欢迎来到我的家园。');
    }else{
        res.send('你还没有登录，先登录下再试试！');
    }
})
```

### 关于signedCookie 

上面提到的使用cookie 虽然很方便，但是使用 cookie 有一个很大的弊端，cookie 中的所有数据在客户端就可以被修改，数据非常容易被伪造

其实计算机领域有个名词叫 签名，我们可以用这个来使cookie变得安全和难以破解。

比如我们现在面临一个网站，它用 cookie 来记录登陆的用户凭证。相应的 cookie 是这样：`dotcom_user = RoronoaZoro`，它说明现在的用户是 RoronoaZoro（索隆）这个用户。如果我在浏览器中装个插件，把它改成`dotcom_user = Shanks`，服务器一读取，就会误认为我是 Shanks（香克斯）。然后我就可以进行 Shanks才能进行的操作了。如果我们把 cookie 改成 dotcom_user=admin ，说不定可以把这个网站给黑掉。。。

现在我有一些数据，不想存在 session 中，想存在 cookie 中，怎么保证不被篡改呢？那就是签名。

假设我的服务器有个秘密字符串，是 `this_is_my_secret_string`，我为用户 cookie 的 dotcom_user 字段设置了个值 RoronoaZoro。cookie 本应是`{dotcom_user: 'RoronoaZoro'}`这样的。

而如果我们签个名，比如把 dotcom_user 的值跟我的 secret_string 做个 sha1
```js
sha1('this_is_my_secret_string' +'RoronoaZoro')===
'4850a42e3bc0d39c978770392cbd8dc2923e3d1d'
```

然后把 cookie 变成这样
```js
{
  dotcom_user: 'RoronoaZoro',  'dotcom_user.sig': '4850a42e3bc0d39c978770392cbd8dc2923e3d1d',
}
```
这样一来，用户就没法伪造信息了。一旦它更改了 cookie 中的信息，则服务器会发现 hash 校验的不一致。
