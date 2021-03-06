
> 前言

之前写JS每句都会加上分号（习惯了很久），但最近好像有点懒，觉得看分号越来越不顺眼，干脆都不写了，所以今天在写async/await函数的时候由于上句没加分号就被坑了，于是认真搜了一番在JS中所有会导致上下行解析出问题的 token，目前一共6个：

**`(`**

**`[`**

**`+`**

**`-`**

**`/`**

**` ` `**

最容易出错的一些例子

###  括号

ES7的async使用已经非常普遍了，这种情况如果不注意的话坑到我们的概率很高
```js
console.log('this has not semicolon')

(async () => {
    await new Promise(res => {
        res('ok')
    })
})()

//ERROR：
//TypeError: console.log(...) is not a function
```

### 方括号
以一个数组作为行首的情况可能不多见，比如：`[1,2,3].map()`类似于这种也是可能会出现的
```js
console.log('this has not semicolon')

[1,2,3].map(item => {
    console.log(item)
})

//ERROR：
//TypeError: Cannot read property '3' of undefined
```
### 反引号
反引号普遍用在ES6的模板字符串里
```js
let str = ''
var a = 5

str = 'string'

`test template string ${a}`

console.log(str)
//ERROR
//TypeError: "string" is not a function
```
初一看报这个错很奇怪，稍稍查一下就知道其实是因为ES6的新语法导致的：tagged template，比如这样
```js
test`test template string ${a}`
```
这么写，模板字符串并不会直接被插值，而是会被拆成各个部分然后把各个部分按一定形式传给这个 test 的函数，而模板字符串的前面的表达式总会被认为是一个函数，这就导致如果模板字符串的左边不是函数的话，就会报 TypeError 错误

### 斜杠

这个出现在行首应该只可能在正则表达式中：
```js
console.log('this has not semicolon')

/1[34578]\d{9}/.test('wx_1345435465')

//ERROR：
//SyntaxError: Invalid or unexpected token
```
这种情况我再vscode中试了下，不加分号编辑器是会直接提示语法错误的，在一般的编辑器中应该也会报错，况且以正则作为行首的情况也基本上见不到

### 加号和减号

这两个试了不会报错，但是下面这个情况有可能不是我们的本意？
```js
console.log('this has not semicolon')

let num = 1

+6

console.log(num)

//RESULT：
//this has not semicolon
//7
```
应该没人会犯这样的错


## 总结

最后总结一下就是如果我们在代码中不想用分号，就得在以括号和方括号为行首的情况下，在行首加上一个分号。

而其他情况基本上很难出现，但真出现并且报错了我们也得知道是什么个情况。

>如有错误，多加指正~~~