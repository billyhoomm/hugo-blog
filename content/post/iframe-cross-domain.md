---
# 文章标题
title: "iframe之间的跨域和通信"
# 文章副标题
subtitle: ""
# 文章日期
date: 2016-04-04T21:30:50+08:00
# 标签
tags: ["浏览器"]
# 分类
categories: ["浏览器"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

### iframe之间的跨域和通信

本周的合规二期开发接触到了很多iframe相关的通信，经验不是很足，iframe通信这些在之前接触较少，而主要用到的通信方式就是window.name，支持2M的数据量，当iframe的页面跳到其他地址时，window.name值可以保持不变。

 #### 非跨域
- 父页面调用子页面：`FrameName.window.childMethod();`
- 子页面调用父页面：`parent.window.parentMethod();`
- 在用FrameName.window得到子页面的窗体之后，就可以像访问同一页面的DOM元素一样来进行DOM操作

#### 跨域
- **主域相同**

只有两个页面同域才能才能访问winodw.name，所以在父子页面的主域名相同的场景下可以将子页面改为与父页面同域（xxx.com）即可（可以通过document.domain设置），这样的话同样可以做到父子页面的双向通信。

- **不同域**

父页面调用子页面：

可以利用 location 对象的 hash 值，我们需要在父页面设置 iframe的 src 后面多加个#data 字符串（data为需要传递的数据），然后在子页面中通过某种方式能即时的获取到这儿 data，这儿可以在子页面中通过 setInterval 方法设置定时器， 监听 location.href 的变化即可获得上面的 data 信息

子页面向父页面传递数据：

实现的技巧是在父子两个iframe之外利用一个代理 Iframe Z，它嵌入到子页面中，并且和父页面必须保持是同域，然后我们通过它充分利用上面非跨域的方式就能把子页面的数据传递给 iframeC，同时iframeC和主页面又是同域的，所以它们之间同样可以以非跨域的方式通信，在这里的可以使用一个经常使用的属性 window.top (也可以使用window.parent.parent)，它返回对载入浏览器得最顶层 window 对象的引用，这样就能直接条用父页面中方法啦。
