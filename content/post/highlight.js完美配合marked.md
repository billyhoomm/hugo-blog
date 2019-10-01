---
# 文章标题
title: "highlight.js完美配合marked"
# 文章副标题
subtitle: ""
# 文章日期
date: 2018-05-26T23:14:20+08:00
# 标签
tags: ["JavaScript", 'html']
# 分类
categories: ["JavaScript"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

> 前言：Using highlight.js with marked

> 这是我的自建博客系统使用`Marked.js `来`parse`markdown文件再结合`highlight.js`来让代码高亮的时候碰到的问题，拿出来分享一下~~

> 博客地址：[The blog of billyhu](http://blog.billyhu.com)

## Marked的问题
[在这里我们可以看到`Marked.js `官方文档关于用其配合`highlight.js`做代码高亮的使用介绍](https://marked.js.org/#/USING_ADVANCED.md)

贴出核心代码：
```js
// Create reference instance
var myMarked = require('marked');

// Set options
// `highlight` example uses `highlight.js`
myMarked.setOptions({
  renderer: new myMarked.Renderer(),
  highlight: function(code) {
    return require('highlight.js').highlightAuto(code).value;
  },
  pedantic: false,
  gfm: true,
  tables: true,
  breaks: false,
  sanitize: false,
  smartLists: true,
  smartypants: false,
  xhtml: false
});

// Compile
console.log(myMarked('I am using __markdown__.'));
```
- `renderer`选项new了一个markdown解析器

- `highlight`选项这里引入了`highlight.js`的高亮处理模块

我基于上面配置用markdown写一段代码块注明为`js`代码，`Marked`将markdown解析为html的时候，处理结果是这样：
```html
<pre>
<code class="language-js">
<!--这里面是各种标上了hljs-string、hljs-attr、hljs-literal等类名的span标签-->
<!--highligh.js依据这些类名为其渲染不同的颜色和样式-->
</code>
</pre>
```

**上面这种示例方式有两个问题：**
- `highlight.js`会用到`<code>`标签中的类名`'hljs'`来为代码添加一部分样式，但是上面没有`'hljs'`类

-  `highlight.js`的高亮解析模块需要传入`'lang'`参数确定当前语言，才能依据不同语言针对不同的关键字生成不同的类名（hljs-string、hljs-attr、hljs-literal等），以便做到精确的高亮

所以由于这两个问题，经常发现写的HTML、JS、CSS等代码出现各种莫名其妙的高亮，比如注释被高亮了，关键字却不高亮。。。

## 解决方法
首先这两个模块的官方文档都写得及其含糊。。。很多关键的细节在文档中根本没提到，查看一番源码方才得知方法：

- **解决问题一：**

上面的`setOptions`有一个`langPrefix`选项，允许你在`<code>`标签标明代码的类名之前自定义一个前缀（默认的前缀为`'language-'`）：
```js
langPrefix:'hljs ',
```
加了前缀之后，上面的代码变成了：
```html
<pre>
<code class="hljs js">
<!--这里面是各种标上了hljs-string、hljs-attr、hljs-literal等类名的span标签-->
<!--highligh.js依据这些类名为其渲染不同的颜色和样式-->
</code>
</pre>
```
这样就搞定了第一个问题，这里官方介绍说是给类名加前缀，我这里是把前缀带上空格，就直接变成了两个类名（也许是官方有让你这样搞的意思，但是好歹说明一下啊，不然谁想得出来-_-）

- **解决问题二：**

看了两个模块的源码就知道`Marked`的`highlight`选项的函数是允许传入多个参数的`code, lang, callback`

同时`highlight.js`存在`hljs.highlight(lang, code)`函数，允许传入lang参数指定语言

于是最开始的`highlight`选项变成下面这样
```js
highlight: function (code, lang, callback) {
  return hljs.highlight(lang, code).value;
}

```

下面是最终的**正确配置：**
```js
myMarked.setOptions({
  renderer: new myMarked.Renderer(),
  highlight: function(code, lang, callback) {
    return require('highlight.js').highlight(lang, code).value;
  },
  langPrefix:'hljs ',
  pedantic: false,
  gfm: true,
  tables: true,
  breaks: false,
  sanitize: false,
  smartLists: true,
  smartypants: false,
  xhtml: false
});
```

> 参考文档：

- [Marked官方文档](https://marked.js.org/#/USING_ADVANCED.md)

- [一个关于Marked代码高亮的Pull request](https://github.com/markedjs/marked/pull/418#issuecomment-57291402)
