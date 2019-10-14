---
# 文章标题
title: "关于JS模块模式的写法和作用"
# 文章副标题
subtitle: ""
# 文章日期
date: 2016-06-09T23:17:50+08:00
# 标签
tags: ["JavaScript"]
# 分类
categories: ["JavaScript"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

> 前言

本来对于js中模块模式这个词还没有很清晰的定义，但在看一个小项目的时候发现了用模块模式定义工具函数的作用，而且已经被很多大公司使用的ECMAScript标准第六版，正式支持"类"和"模块"，下面这种写法可能以后会被取代吧，不过也是目前流行的写法，暂且记录着。
### 简介
首先我们来看看Module模式的基本特征：

- 模块化，可重用
- 封装了变量和function，减少全局变量的使用(不遗留和污染全局变量)
- 私有方法和私有变量全部隐藏（更安全），并提供公有接口
### Module模式最简单的方式：
```js
var Calculator = function (eq) {
    //这里可以声明私有成员
 
    var eqCtl = document.getElementById(eq);
 
    return {
        // 暴露公开的成员
        add: function (x, y) {
            var val = x + y;
            eqCtl.innerHTML = val;
        }
    };
};
```
需要如此调用：
```js
var calculator = new Calculator('eq');
calculator.add(2, 2);
```
每次用的时候都要new一下，也就是说每个实例在内存里都是一份copy，浪费了很多内存
### 立即执行函数写法：
```js
var module1 = (function(){
　　　　var _count = 0;
　　　　var m1 = function(){
　　　　　　//...内部代码
　　　　};
　　　　var m2 = function(){
　　　　　　//...内部代码
　　　　};
　　　　return {
　　　　　　m1 : m1,
　　　　　　m2 : m2
　　　　};
　　})();
```
使用上面的闭包写法，外部代码无法读取内部的_count变量，并且私有方法m1和m2不会暴露，而且函数会立即执行，将私有方法集合保存到module1变量（module1本来是一个方法，因为它立即执行，所以会伪装成一个变量），之后使用module1.m1()或module1.m2()的方式访问私有变量（_count）并且可以传入参数执行内部代码，这样的调用方法会让内存中只存在一份copy，节约了内存。

后面还有比较高级的扩展方法，不过此文章只为了阐述基础，而且已经满足了大部分情况的使用了，倘若以后在实际开发中用到了高级扩展的写法，再进行总结。