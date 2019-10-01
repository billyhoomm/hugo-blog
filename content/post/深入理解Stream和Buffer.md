---
# 文章标题
title: "深入理解Stream和Buffer"
# 文章副标题
subtitle: ""
# 文章日期
date: 2017-12-09T16:26:50+08:00
# 标签
tags: ["NodeJS"]
# 分类
categories: ["NodeJS"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

## Stream
stream是Node.js提供的又一个仅在服务区端可用的模块，目的是支持“流”这种数据结构。
流是一种抽象的数据结构。想象水流，当在水管中流动时，就可以从某个地方源源不断地到达另一个地方，所以可以把数据看成是数据流。

有些流用来读取数据，比如从文件读取数据时，可以打开一个文件流，然后从文件流中不断地读取数据。有些流用来写入数据，比如向文件写入数据时，只需要把数据不断地往文件流中写进去就可以了。

### 思考一个场景

在node中拷贝一个文件的内容到另一个文件中差不多是这样：
```js
fs.readFileSync('/path/to/source', {encoding: 'utf8'});
fs.writeFileSync('/path/to/dest', source);
```
也就是先将一个完整文件读到内存，再将内存中数据写入对应文件，小文件这样做是没问题的，但是就像我现在正在写的一个服务，视频和音频转存服务，视频大小基本保持在100M以上，直接读这样的文件，服务内存分分钟被撑爆。

面对这种大文件，我们应该是像上面提到的像水管一样读，不断反复直到文件全部读取完毕，写入也结束。

**Tips：**
> V8引擎默认限制了堆内存的大小，64位系统为1.4G，32位系统为0.7G，这样的默认设计主要是考虑到V8的垃圾回收效率，如果内存超过1.5G，做一次小的垃圾回收，时间就可能在50ms以上


在Node.js中，流是一个对象，我们只需要响应流的事件就可以了：data事件表示流的数据已经可以读取了，end事件表示这个流已经到末尾了，没有数据可以读取了，error事件表示出错了。
### 简单示例：从文件流中读取文本内容
```js
'use strict';

var fs = require('fs');

// 打开一个流:
var rs = fs.createReadStream('sample.txt', 'utf-8');

rs.on('data', function (chunk) {
    console.log('DATA:')
    console.log(chunk);
});

rs.on('end', function () {
    console.log('END');
});

rs.on('error', function (err) {
    console.log('ERROR: ' + err);
});
```
> data事件可能会有多次，每次传递的chunk是流的一部分数据。

### pipe
就像可以把两个水管串成一个更长的水管一样，两个流也可以串起来。一个Readable流和一个Writable流串起来后，所有的数据自动从Readable流进入Writable流，这种操作叫pipe。

用pipe()把一个文件流和另一个文件流串起来，这样源文件的所有数据就自动写入到目标文件里了，所以，这实际上是一个复制文件的程序：
```js
'use strict';

var fs = require('fs');

var rs = fs.createReadStream('sample.txt');
var ws = fs.createWriteStream('copied.txt');

rs.pipe(ws);
```
> 当end事件触发后，将自动关闭Writable流

### Readable和Writeable
> 上面演示了fs.createReadStream和 fs.createWriteStream实现的流的功能，fs模块的这两个接口其实是基于stream模块的Readable和Writeable来实现的。

Readable为可读取流，Writable即可写流，首先Readable它分为流动模式和暂停模式两种，我们大部分时候都会应用到它的流动模式，但初始化一个Readable流的时候，它是暂停模式的，想将它变为流动模式有以下方法：
- 1、添加一个data事件的监听器来监听数据
- 2、 调用resume()方法来明确开启流动模式
- 3、 调用pipe()方法将数据导入一个可写流

第一种也就是上面例子中的`rs.on('data')`，会持续读取，但其实这样会有一些问题：当在回调函数中有写文件操作时，写入的速度低于读取的速度时，会造成数据的丢失

第二种可以控制速度的pause + resume方式可以解决上面问题：
```js
var fs = require('fs');
var rs = fs.createReadStream('source/file');
var ws = fs.createWriteStream('dest/file');

rs.on('data', function(chunk){
  if(ws.write(chunk) === false){ //【write()方法是有返回值的，该方法会返回true或者false，返回true说明已经将数据写入，返回false则说明数据尚未写完】
    rs.pause();// 尚未写完，停止读取
  }
});

ws.on('drain', function(){
  rs.resume(); // 数据已经写完，继续读取
});

rs.on('end', function(){ // 已经没有更多数据，关闭可写流
  ws.end();
});
```
**Tips：**
> drain事件：这是个啥呢？与HighWaterMark参数（高水位线）有点关系，TODO

第三种就是上面介绍过的pipe（管道）**`rs.pipe(ws)`**,只需要一行代码就可以解决上面的所有问题了，这也是Node官方推荐我们使用的
## Buffer
> Buffer代表一个缓冲区，存储二进制数据，是字节流。也是应用Stream的基础支撑，我们在网络传输时，就传输的这种字节流。写文件时，也是写的字节流。

### 独立内存
上面提到的Buffer对象都是JavaScript层面上的，能够被V8的垃圾回收标记回收。但其内部的parent属性指向的SlowBuffer对象却来自于Node自身C++中的定义，是C++层面上的Buffer对象，所以内存不在V8的堆中。

当我们需要在内存中存储一些比较大的字符串时，转换为Buffer存储可以降低Node的内存使用率，但也会带来一些问题，比如多了Buffer与String的互转操作，性能比不上原生的String，在高IO场景下也许会带来性能问题。
### 8KB载体
- 当我们实例化一个新的Buffer类，会根据实例化时的大小去申请内存空间，如果需要的空间小于8KB，则会多一次判定，判定当前的8KB载体剩余容量是否够新的buffer实例，如果够用，则将新的buffer实例保存在当前的8KB载体中，并且更新剩余的空间。如果不够用，则很悲催剩余空间会被浪费掉。

- 如果需要超过8KB的Buffer对象，将会直接分配一个SlowBuffer对象作为slab单元，这个单元将会被这个大Buffer对象所独占。

### 字符串拼接
很多代码是这样写：
```js
let rs = fs.createReadStream('test.md');
let data = '';
rs.on("data", function(chunk) {
    data += chunk; //等价于data = data.toString() + chunk.toString();
});
```

另外一种是使用`Buffer.concat()`来拼接
```js
let chunks = [];
let clen = 0;
res.on('data', function(chunk) {
    chunks.push(chunk);
    clen += chunk.length;
});
res.on('end', function() {
    var buf = Buffer.concat(chunks, clen);
});
```
测试下的结果是使用`data += chunk`方式的拼接耗时基本接近`Buffer.concat()`方式的两倍了，所以性能上考虑明显用后者较好。并且`data += chunk`方式在处理非UTF-8的字符串以及UTF-8宽字节中文的时候很可能出现乱码错误。


### 编码格式
字符串是有编码格式的，比如UTF-8。而Buffer是没有编码格式的。两者可以相互转换。转换时必须指定编码格式。

例如：我们最常用的`http.createServer`的的回调函数的第一个参数`req`就实现了上文中提到的Readable接口：`http.IncomingMessage`，在这个流中读到的数据就是Buffer对象，是字节流，而我们在程序中使用时，经常是要转换为String。反过来，res（类型`http.ServerResponse`，可写的流，实现了Writable接口）有个方法`setDefaultEncoding`，用来设置流的编码格式，在write数据时，会使用指定的编码格式来编码数据，然后发送给客户端。

就是说，网络传输的是Buffer，程序需要处理String，Buffer和String之间可以转换。Buffer有`toString`方法，可以按指定的编码格式将字节流转换为String。`fs.createWriteStream`和`fs.createReadStream`两个方法都有一个可选参数options，可以指定`defaultEncoding`，这里指定的编码格式，也是用于在Buffer和String之间转换的。

** iconv-lite**

我们在处理非UTF-8编码的字符串、二进制格式的时候必须得使用Buffer，然而Buffer支持的编码类型有限（utf8、ascii、utf16le、utf16be、ucs2、base64、hex），比如在windows上常见的ANSI（GB2312）编码在node下处理会直接乱码。[iconv-lite](https://github.com/ashtuchkin/iconv-lite#extend-nodejs-own-encodings)（纯JS实现）可以实现编码转换

**jschardet**

[jschardet](https://www.npmjs.com/package/jschardet)可以检测一段字符串或者读取到的文件的编码方式，并且有准确率的提示。

## 总结

好好耍
各种实现原理有待研究...

其他各种api可以参考文档：https://nodejs.org/dist/latest-v8.x/docs/api/