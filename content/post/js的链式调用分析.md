---
# 文章标题
title: "js的链式调用分析"
# 文章副标题
subtitle: ""
# 文章日期
date: 2016-01-17T21:40:32+08:00
# 标签
tags: ["JavaScript"]
# 分类
categories: ["JavaScript"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

> 前言

在做一个组件化开发的案例时在主逻辑中用到了链式调用的方法，目的是为了让代码更简洁易读，减少代码量，但是对各个构造函数末尾的" return this "很是疑惑，未明其意。查了书才明白这句话的作用。
### 什么是链式调用
其实jquery里面的自带函数都可以链式调用，比如：
```js
point.width(100px).height(100px);
$(this).setStyle('color', 'red').show();
```
上面两个语句都是链式调用
### 分析
链式调用其实是两个部分：

1.操作对象(也就是被操作的DOM元素，如上例的$(this))

2.操作方法(具体要做什么事情，如上例的setStyle和show)

这是jquery中$函数的设计方法：
```js
function $(){
  var elements = [];
  for(var i= 0,len=arguments.length; i<len; i++){
    var element = arguments[i];
    if(typeof element==='string'){
      element = document.getElementById(element);
    }
    if(arguments.length==1){
      return element;
    }
    elements.push(element);
  }
  return elements;
}
```
现在可以将这个函数改造为下面这样：
```js
(function(){
  function _$(els){
    this.elements = [];//把那些元素作为数组保存在一个实例属性中，
    for(var i= 0, len=els.length; i<len; i++){
      var element = els[i];
      if(typeof element==='string'){
        element = document.getElementById(element);
      }
      this.elements.push(element);
    }
  }
 
  _$.prototype = {
    each: function(fn){
      for(var i= 0,len=this.elements.length; i<len; i++){
        fn.call(this, this.elements[i]);
        return this; //在每个方法的最后return this;
      }
    },
    setStyle: function(prop, val){
      this.each(function(el){
        el.style[prop] = val;
      });
      return this; //在每个方法的最后return this;
    },
    show: function(){
      var that = this;
      this.each(function(el){
        that.setStyle('display', 'block');
      });
      return this; //在每个方法的最后return this;
    },
    addEvent: function(type, fn){
      var add = function(el){
        if(window.addEventListener){
          el.addEventListener(type, fn, false);
        }else if(window.attachEvent){
          el.addEvent('on'+type, fn);
        }
      };
      this.each(function(el){
        add(el);
      });
      return this; //在每个方法的最后return this;
    }
  }
  window.$ = function(){
    return new _$(arguments);
  }
})();
```
上面的代码是让定义在构造函数的prototype属性中的对象都返回用来调用方法的那个实例的引用（在上例中也就是返回" -$() "函数的实例），这样的话，如果每次都return this，那么链式调用就可以不断进行下去。
### 实际使用
比如在我这次组件式开发中用的是下面这样的形式：

构造函数：
```js
/*JS主逻辑*/
var H5 = function(){
    this.addPage(){//新增一页
        /*代码和方法*/
        return this;
    }
    
    this.addComponent(){//新增一个组件
        /*代码和方法*/
        return this;
    }
 
    this.loader(){
        /*代码和方法*/
        return this;
    }
}
```
调用：
```js
$(function(){
    var h5 = new H5();
    h5.addPage()
          .addComponent()
          .addComponnet()
          .addComponent()
          .addComponent()
      .addPage()
          .addComponent()
          .addComponnet()
          .addComponent()
          .addComponent()
      ...//后续调用
})
```
这样使用可以避免多次重复使用一个变量对象，减少代码量，使逻辑更清晰。