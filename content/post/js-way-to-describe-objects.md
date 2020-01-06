---
title: "JavaScript中描述对象的方式：基于类和基于原型"
date: 2019-11-09T10:46:01+08:00
tags: ["JavaScript", "Object", "面向对象编程", "class", "原型"]
categories: ["JavaScript"]
draft: true
---

自从ES6中引入了class关键字，基于类的编程方式就成为了JavaScript的官方编程范式，这也解决了之前的使用各种诡异的方式来模拟类的编码方式。

同时自ES6以来，作为最出名的基于原型的编程语言，JavaScript也提供了一系列新的内置函数，来方便我们更为直接的访问和操纵原型，这样我们会有两种选择：

- 基于类的面向对象

- 基于原型的面向对象

概括一下：

- ”基于类“的编程提倡使用一个关注类和类之间关系的开发模型，我们规划代码时，总是先有类，再从类去实例化一个对象。类和类之间又可能会形成继承、组合等关系，类又往往与语言的类型系统整合，形成一定的编译时能力。
- ”基于原型“的编程让我们更关心一系列对象实例的行为，然后才去关心如何将这些对象划分到最近的使用方式相似的原型对象，而不是将它们分成类



## JavaScript的原型

JavaScript原型系统的主要特点是：

- 所有的对象都有一个*私有字段 [[prototype]]* 指向自身的原型对象
- 读一个属性，如果对象本身没有，则会继续访问对象的原型，直到原型为空或者找到为止。

ES6提供给我们一系列内置函数，可以更为方便直接的操纵原型：

- `Object.create`根据指定的原型对象来创建新对象
- `Object.getPrototypeOf`可以获得一个对象的原型
- `Object.setPrototypeOf`可以设置一个对象的原型

利用上述方法，我们可以抛开类的思维，基于原型来实现抽象和复用，抽象🐈和🐯：

```javascript
let cat = {
  say () {
    console.log('meow~')
  }
  jump () {
    console.log('jump~')
  }
}
let tiger = Object.create(cat, {
  say: {
    value: function() {
    	console.log("roar!");
    }
  }
})
let anthorCat = Object.create(cat)
anthorCat.say()
let anthorTiger = Object.create(tiger)
anthorTiger.say()
```

上面先创建了🐈对象，然后根据🐈为原型做修改创建了🐯，之后再利用Object.create创建其他的🐈和🐯。

这样的话，就可以通过原始的🐈和🐯对象来控制所有的🐈和🐯的行为。



## new运算

在ES6之前，根本没有Object.create这样的方法，要想指定的对象的[[prototype]]，只能通过new运算来进行：

```javascript
function Base(){
    this.p1 = 1;
    this.p2 = function(){
        console.log(this.p1);
    }
} 
let obj = new Base;
obj.p2();
```

上面代码的new运算的实际步骤是：

- 以构造器c1的 prototype 属性为原型，创建新对象
- 将this和调用参数传递给构造器，执行
- 如果构造器返回的是对象，则返回，否则返回第一步创建的对象

用代码来详细解释一下`let obj = new Base;`这句

```javascript
// 1. 创建（或者说构造）一个全新的对象；
var _obj = {};

// 2. 我们将这个空对象的__proto__成员指向了Base函数对象prototype成员对象
_obj.__proto__ = Base.prototype;

// 3. 我们将Base函数对象的this指针替换成_obj，然后再调用Base函数
var _return = Base.call(_obj);

// 4 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象
if (typeof _return === "object") {
  obj = _return;
} else {
  obj = _obj;
}
```



## ES6中的类

上面new运算和function搭配来实现类似class的行为对于很多人来说理解起来都很奇怪，也不利于阅读和整理代码。

而在ES6中，任何时候我们都可以使用class关键字来定义类，而让function回归它原本的函数语义（而不是构造器）

基本写法：

```javascript
class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
  // Getter
  get area() {
    return this.calcArea();
  }
  // Method
  calcArea() {
    return this.height * this.width;
  }
}
```

通过get/set关键字来创建getter，通过括号和大括号来创建方法，数据型成员最好写在构造器里面

继承：

```javascript
class Animal { 
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(this.name + ' makes a noise.');
  }
}

class Dog extends Animal {
  constructor(name) {
    super(name); // call the super class constructor and pass in the name parameter
  }

  speak() {
    console.log(this.name + ' barks.');
  }
}

let d = new Dog('Mitzie');
d.speak(); // Mitzie barks.
```

上面通过extends关键字Dog继承了Animal，它会自动设置constructor，并且自动调用父类的构造函数，无需我们手动操作。

## ES6中使用class的注意点

- class 先定义再使用，**不存在变量提升**
- **不存在私有属性和方法**
- 可以定义**静态属性和方法**，静态方法中的 this 指向 class 本身

```typescript
class Foo {
  static bar() {
    this.baz(); // this === Foo
  }
  static baz() {
    console.log("hello");
  }
  baz() {
    console.log("world");
  }
}

Foo.bar(); // hello
```

- 实例属性可以不在`constructor`中定义，直接写在外面

```typescript
class ReactCounter extends React.Component {
  state = {
    count: 0
  };
}
```

- `new.target`的使用技巧：**感知当前类的继承情况**

```typescript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error("本类不能实例化");
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}

var x = new Shape(); // 报错
var y = new Rectangle(3, 4); // 正确
```

- 理解继承类中的`super`函数（这部分有三种情况）

```typescript
class Parent {
  static myMethod(msg) {
    console.log("static", msg);
  }

  myMethod(msg) {
    console.log("instance", msg);
  }
}

class Child extends Parent {
  constructor(name) {
    super(); // 1. super()===Parent.prototype.constructor.call(this)
    this.name = name;
  }
  static myMethod(msg) {
    super.myMethod(msg); // 2. super===Parent
  }

  myMethod(msg) {
    super.myMethod(msg); // 3. super===Parent.prototype, 父类原型上的方法
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```



## 最后

使用类的思想来设计代码时，也应该用class来声明类，而不是使用旧语法（使用function和new来模拟），这样会避免很多坑。

从我们编码层面来看，class关键字和箭头运算符可以完全替代旧的function关键字，它更明确地区分了定义函数和定义类两种意图。

参考自：

>重学前端 - Winter
>
>JavaScript高级程序设计



 	



