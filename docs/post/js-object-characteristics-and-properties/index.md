
## 什么是对象？

在《面向对象分析与设计》书中，作者Grady Booch替我们做了总结，他认为，从人类的认知角度来说，对象应该是下列事物之一：

- 一个可以触摸或者可以看见的东西；
- 人的智力可以理解的东西；
- 可以指导思考或行动（进行想象或施加动作）的东西。

在编程语言中，对象应该具有如下特点：

- 对象具有唯一标识性：即使两个完全相同的对象，也并非同一个对象；
- 对象有状态：对象具有状态，同一对象可能处于不同状态之下；
- 对象具有行为：即对象的状态，可能因为它的行为而产生变迁。

而`面向对象编程`则是使用编程语言来抽象描述对象，早年的JavaScript采用了`原型`来描述对象，但大多数成功且流行的方式是使用`类`的方式，比如Java、C++。

早期JavaScript在创造的时候为了模仿Java，在原型的基础上引入了new、this等特性，让它”看起来更像Java“，可能这也是为什么发展到现在，从ES6之后，JavaScript已经完整的拥有了类的概念，让我们可以基于类来描述对象。



## JavaScript中对象的特征

上面介绍了在编程语言中对象应该具有的特点

其中第一个特征：对象具有唯一标识性，在各个语言中都是用内存地址来体现的，对象具有唯一的内存地址，所以才具有唯一标识性。

C语言中的指针其实可以更好的来帮助理解内存地址：

> 指针是一个变量，其值是另一个变量的地址，即，内存位置的直接地址；
>
> 在C中定义的各种类型的指针其实都是一个代表内存地址的长的十六进制数；

```c
int    *ip;    /* 一个整型的指针 */
double *dp;    /* 一个 double 型的指针 */
float  *fp;    /* 一个浮点型的指针 */
char   *ch;     /* 一个字符型的指针 */
```

> 这个理论在其他编程语言中，包括在JavaScript中都是适用的，只是在C中看起来更底层，可以灵活的运用指针

关于第二个和第三个特征”状态和行为“，不同语言会使用不同的术语来抽象描述它们，比如C++中称它们为“成员变量”和“成员函数”，Java中则称它们为“属性”和“方法”

在 JavaScript中，将状态和行为统一抽象为“属性”，例如下面，d是一个属性，而函数f也是一个属性，尽管写法不太相同，但是对JavaScript来说，d和f就是两个普通属性

```javascript
var o = {
	d: 1,
	f() {
  	console.log(this.d);
	}    
};
```

**在实现了对象基本特征的基础上，JavaScript中对象独有的特色是：对象具有高度的动态性，这是因为JavaScript赋予了使用者在运行时为对象添改状态和行为的能力**，这和其他的基于类的、静态的对象设计的语言是完全不同的



## JavaScript对象的两类属性

第一类属性是数据属性，数据属性具有四个特征：

- value：就是属性的值。
- writable：决定属性能否被赋值。
- enumerable：决定for in能否枚举该属性。
- configurable：决定该属性能否被删除或者改变特征值。

在大多数情况下，我们只关心数据属性的值即可。

第二类属性是访问器（getter/setter）属性，它也有四个特征：

- getter：函数或undefined，在取属性值时被调用。
- setter：函数或undefined，在设置属性值时被调用。
- enumerable：决定for in能否枚举该属性。
- configurable：决定该属性能否被删除或者改变特征值。

访问器属性使得属性在读和写时执行代码，它允许使用者在写和读属性时，得到完全不同的值，它可以视为一种函数的语法糖。

我们通常用于定义属性的代码会产生数据属性，其中的writable、enumerable、configurable都默认为true：

```javascript
var o = { a: 1 };
Object.getOwnPropertyDescriptor(o,"a")
// {value: 1, writable: true, enumerable: true, configurable: true}
```

如果我们要想改变属性的特征：

```javascript
var o = { a: 1 };
Object.defineProperty(o, "b", {
  value: 2,
  writable: false,
  enumerable: false,
  configurable: true
});
//a和b都是数据属性，但特征值变化了
Object.getOwnPropertyDescriptor(o,"a");
// {value: 1, writable: true, enumerable: true, configurable: true} 
Object.getOwnPropertyDescriptor(o,"b");
// {value: 2, writable: false, enumerable: false, configurable: true}
o.b = 3;
console.log(o.b);// 2
```

这里我们使用了`Object.defineProperty`来定义属性，这样来定义属性可以改变属性的writable和enumerable。

如果想创建访问器属性，我们可以使用`get`和`set`关键字：

```javascript
var o = {
  get a() {
    return 1
  }
};
console.log(o.a); // 1
```

访问器属性跟数据属性不同，每次访问属性都会执行getter或者setter函数。这里我们的getter函数返回了1，所以o.a每次都得到1。

## 总结

所以我们可以基本理解为：`JavaScript对象是一个具有高度动态性的属性的索引结构`

> 索引结构是一类常见的数据结构，我们可以把它理解为一个能够以比较快的速度用key来查找value的字典
>
> 以上面的对象o为例，‘a’是key，{writable:true,value:1,configurable:true,enumerable:true}是value



参考自：

重学前端 - Winter

JavaScript高级程序设计