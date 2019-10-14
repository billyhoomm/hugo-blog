---
# 文章标题
title: "JS的函数定义方式以及对声明的提前"
# 文章副标题
subtitle: ""
# 文章日期
date: 2016-03-08T23:20:32+08:00
# 标签
tags: ["JavaScript"]
# 分类
categories: ["JavaScript"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

> 前言

由于javascript中的函数定义方式比较特殊，并且每种的特点都不相同，所以本文介绍一下JS几种函数定义方式和各自的优缺点，并且对JS中的函数声明和变量声明的hoist（提前）进行简略说明。

### 函数定义方式
#### 函数生命式
这是js中比较常用的方式：
```js
function funcname(a,b){
  //处理代码
}
```
当该函数被调用，函数体才会被得到执行。
#### function构造函数方式
```js
var funcname = new Function ('x', 'y', 'alert(x+y)');
```
该定义方式中前面两个是参数，参数可以是任意多个字符串；第三个是函数体，可以包含任何 JavaScript 语句，语句之间用分号隔开。如果没有参数，传一个函数体即可。由于传递给 Function () 函数中，没有一个字符串是用来声明函数名的，所以它是一个匿名函数。
#### 函数直接量
```js
var funcname = function(a,b){
  //处理代码
}
```
#### 区别
三种方式的区别，可以从作用域、效率以及加载顺序来区分。首先，从作用域上来说，函数声明式和函数直接量使用的是局部变量，而 Function()构造函数却是全局变量:
```js
var y = 'global';
function a(){
  var y = 'local a';
  return y;
}
alert(a());//显示'local a'
var b = function(){
  var y = 'local b';
return y;
}
alert(b()) //显示'local b'
function c(){
  var y = 'local c';
  return new Function('return y');
}
alert(c()());//显示'global'，因为Function()返回的是全局变量y，而不是函数体内的局部变量。
```
再看效率，Function()构造函数的效率要低于其他两种方式，尤其是在循环体中，因为构造函数每执行一次都要重新编译，并且生成新的函数对象。

最后是加载顺序，function 方式(即函数声明式)是在 JavaScript 编译的时候就加载到作用域中,而其他两种方式则是在代码执行的时候加载，如果在定义之前调用它，则会返回 `undefined`。
### 函数声明和变量声明的提前
从上面对三种定义方式的比较来看，只有函数声明方式定义的函数会在JS编译的时候就加载到作用域中，由此想到了JS中对声明的提前

#### 变量声明提前：
```js
(function() {
  console.log(a);//undefined
  var a = "Now it's defined!";
  console.log(a);//"Now it's defined!"
})();
```
在未对a进行定义和声明的时候输出a却提示undefined但没有报错，这其实是 JavaScript 解析器搞的鬼，解析器将当前作用域内声明的所有变量和函数都会放到作用域的开始处，但是，只有变量的声明被提前到作用域的开始处了，而赋值操作被保留在原处。

由于 JavaScript 具有这样的“怪癖”，所以你会看到很多编码指南建议大家将变量声明放在作用域的最上方，这样就能时刻提醒自己注意了。
#### 函数声明提前：
**第一种情况：**
```js
isItHoisted();//"Yes!"
function isItHoisted() {  
    console.log("Yes!");
}
```
如上所示，JavaScript 解释器允许你在函数声明之前使用，也就是说，函数声明并不仅仅是函数名“被提前”了，整个函数的定义也“被提前”了！所以上述代码能够正确执行。

**第二种情况：**
```js
funca();//"Definition hoisted!"
funcb();//抛出错误：undefined is not a function
 
function funca() {  
    console.log("Definition hoisted!");
}
var funcb = function () {  
    console.log("Definition not hoisted!");
};
```
我们做了一个对比，funca函数被妥妥的执行了，符合第一种类型；funcb函数 变量“被提前”了，但是他的赋值（也就是函数）并没有被提前，从这一点上来说，和前面我们所讲的变量“被提前”是完全一致的，并且，由于“被提前”的变量的默认值是 `undefined` ，所以报的错误属于“类型不匹配”，因为 undefined 不是函数，当然不能被调用。

### 总结
通过上面的讲解可以总结如下：

- 变量的声明被提前到作用域顶部，赋值保留在原地
- 函数声明方式定义函数整个“被提前”
- 函数直接量定义函数的方式只有变量“被提前”了，函数没有“被提前”

正好和前面所说的函数定义的三种不同方式的区别是吻合的。

所以提醒大家：作为最佳实践，变量声明一定要放在作用域/函数的最上方（JavaScript 只有函数作用域！）。