---
# 文章标题
title: "JS依次执行多项异步任务"
# 文章副标题
subtitle: ""
# 文章日期
date: 2017-05-19T23:23:50+08:00
# 标签
tags: ["JavaScript", 'Promise']
# 分类
categories: ["JavaScript"]
# 草稿标识，true代表正式环境下不显示
draft: false
---

### 1、JS依次执行多项异步任务

- 有时候，我们希望批量执行一组异步任务，但是不是并行，而是依次执行，这组任务是动态的，在一个数组里，当然我们可以用 for 循环然后一个一个 await 执行，但是还有另外一种方式：

```js
async function taskReducer(promise, action){
  let res = await promise;
  return action(res);
}

function sleep(ms){
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function asyncTask(i){
  await sleep(500);
  console.log(`task ${i} done`);
  return ++i;
}


[asyncTask, asyncTask, asyncTask].reduce(taskReducer, 0);
```
在上面的例子里，我们定义了一个 taskReducer：

```js
async function taskReducer(promise, action){
  let res = await promise;
  return action(res);
}
```
这个 reducer 的两个参数是 promise 和 action，promise 是代表当前任务的 promise，而 action 是下一个要执行的任务。我们可以 await 当前 promise 执行当前任务，然后将执行结果传给下一个 action 就可以了。

这样我们可以调用：

```js
[task1, task2, task3, ...].reduce(taskReducer, init);
```
不管这些任务是同步还是异步都可以被依次执行。需要注意的是，每一个任务的返回值将是下一个任务的输入 promise 或者 value。

### 2、generator 与 async/await 一同使用
将上面的代码进一步扩展，我们发现，它可以支持 generator 与 async/await 一同使用：

```js
async function reducer(promise, action){
  let res = await promise;
  return action(res);
}

function tick(i){
  console.log(i);
  return new Promise(resolve => setTimeout(()=>resolve(++i), 1000));
}

function continuous(...functors){
  return async function(input){
    return await functors.reduce(reducer, input)
  }
}

function * timing(count = 5){
  for(let i = 0; i < count; i++){
    yield tick;
  }
}

continuous(...timing(10))(0);
```
在上面的例子里，我们定义了一个计时 tick 函数，我们通过 timing 来连续调用它，而 timing 是一个 generator，计时器显然是异步函数，然而我们可以：
```js
continuous(...timing(10))(0);
```
而这里的 continuous 其实就是前面的 reduce 的封装。