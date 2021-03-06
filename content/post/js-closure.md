---
# 文章标题
title: "关于Javascript闭包的总结"
# 文章副标题
subtitle: ""
# 文章日期
date: 2015-10-31T12:20:50+08:00
# 标签
tags: ["JavaScript"]
# 分类
categories: ["JavaScript"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

## 关于闭包这个词的解释
**维基百科中对于闭包的经典解释：**

在计算机科学中，闭包（Closure）是词法闭包（Lexical Closure）的简称，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。

Peter J. Landin 在1964年将术语闭包定义为一种包含环境成分和控制成分的实体。

**我的理解是：**

**`闭包就是能够读取其他函数内部变量的函数。`**

由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

在程序语言中，闭包就是一种语法糖，它以很自然的形式，把我们的目的和我们的目的所涉及的资源全给自动打包在一起，以某种自然、尽量不让人误解的方式让人来使用。至于其具体实现，我个人意见，在不影响使用的情况下，不求甚解即可。在很多情况下，需要在一段代码里去访问外部的局部变量，不提供这种语法糖，需要写非常多的代码，有了闭包这个语法糖，就不用写这么多代码，自然而然的就用了。
## 闭包的用途：
闭包可以用在许多地方。它的最大用处有两个，一个是可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中。请看下面的代码：
```js
function f1(){

var n=999;

nAdd=function(){n+=1}

function f2(){
alert(n);
}

return f2;

}

var result=f1();

result(); // 999

nAdd();

result(); // 1000
```
**个人解释：**

在这段代码中，result实际上就是闭包f2函数。它一共运行了两次，第一次的值是999，第二次的值是1000。这证明了，函数f1中的局部变量n一直保存在内存中，并没有在f1调用后被自动清除。

为什么会这样呢？原因就在于f1是f2的父函数，而f2被赋给了一个全局变量，这导致f2始终在内存中，而f2的存在依赖于f1，因此f1也始终在内存中，不会在调用结束后，被垃圾回收机制（garbage collection）回收。

这段代码中另一个值得注意的地方，就是"nAdd=function(){n+=1}"这一行，首先在nAdd前面没有使用var关键字，因此nAdd是一个全局变量，而不是局部变量。其次，nAdd的值是一个匿名函数（anonymous function），而这个匿名函数本身也是一个闭包，所以nAdd相当于是一个setter，可以在函数外部对函数内部的局部变量进行操作。

**javascript高级程序设计一书中对此的解释：**

当闭包f2在f1中被返回后，它的作用域链被初始化为包含f1函数的活动对象和全局变量对象。这样，f2函数就可以访问在f1函数定义的所有变量。更为重要的是，f1函数在执行完毕后，其活动对象也不会被销毁，因为f2函数的作用域链仍然在引用这个活动对象。换句话说，当f1函数返回后，其执行环境的作用域链会被销毁，但它的活动对象仍然会留在内存中；直到f2函数被销毁后，f1函数的活动对象才会被销毁。

**闭包经常用于创建含有隐藏数据的函数：**
```js
var db = (function() {
// 创建一个隐藏的object, 这个object持有一些数据
// 从外部是不能访问这个object的
var data = {};
// 创建一个函数, 这个函数提供一些访问data的数据的方法
return function(key, val) {
if (val === undefined) { return data[key] } // get
else { return data[key] = val } // set
}
// 我们可以调用这个匿名方法
// 返回这个内部函数，它是一个闭包
})();

db('x'); // 返回 undefined
db('x', 1); // 设置data['x']为1
db('x'); // 返回 1
// 我们不可能访问data这个object本身
// 但是我们可以设置它的成员
```
## 关于闭包中的this对象：
我们知道。this对象是运行时基于函数的执行环境绑定的：在全局函数中this等于Window，而当函数被作为某个对象的方法调用时，this等于那个对象。不过，**`匿名函数的执行环境具有全局性`**，因此this通常指向Window。

代码片段一:
```js
var name = "The Window";
var object = {
name : "My Object",
getNameFunc : function(){
return function(){
return this.name;
};
}
};
alert(object.getNameFunc()());
```
代码片段二:

```js
var name = "The Window";
var object = {
name : "My Object",
getNameFunc : function(){
var that = this;
return function(){
return that.name;
};
}
};
alert(object.getNameFunc()());
```
代码片段一中返回一个匿名函数，而匿名函数返回this.name ,结果是"The Window"，即全局变量name的值，为什么没有取到其包含作用域的this对象呢？原因就在于每个函数在被调用时都会自动取得两个特殊变量：this和argument。内部函数在搜索这两个变量时，只会搜索到其活动对象为止，而上面的匿名函数在上一级包含函数中没有搜索到活动对象，所以便搜索到了全局变量"The Window"。

代码片段二中把this对象赋值给了一个名叫that的变量，而在定义了闭包之后，闭包也可以访问这个变量，因为它是我们在包含函数中特意声明的一个变量。

this和argument有同样的问题，如果想访问作用域中的argument或this对象，必须将改对象的引用保存在另一个闭包能够访问的变量中（**`即它的上一级函数`**）。