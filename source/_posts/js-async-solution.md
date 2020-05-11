---
title: JavaScript 之 「异步解决方案」
keywords: JavaScript
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510145459.png'
tags: 对象
categories:
  - JavaScript
date: 2020-05-10 14:20:56
---

我们知道 JavaScript 语言是单线程的。也就是一次只能执行一个任务。如果有多个任务，就必须排队，前一个完成， 再执行下一个。但是，若是有一个耗时较长，后面所有的任务都必须排队等待，会拖慢整个程序的运行。常见的是浏览器的假死。
为了解决这个问题，JavaScript 将任务的执行模式分为同步任务与异步任务。本文主要聊聊在 JavaScript 中的异步解决方案。

<!-- MORE -->

## 回调函数 Callback

回调函数是异步操作最基本的方法。

```js
ajax(url, res => {
  console.log(res)
})
```
### 回调地狱 Callback Hell

但是回调函数有一个致命的弱点——回调地狱。假设多个请求有依赖性，你可能会写出这样的代码。

```js
ajax(url1, res1 => {
    // 处理逻辑
    ajax(url2, res2 => {
        // 处理逻辑
        ajax(url3, res3 => {
            // 处理逻辑
        })
    })
})
```

回调函数的优点是，简单、易理解和实现，但是不利于代码的阅读与维护。各个部分之间耦合混乱，流程难以追踪。而且，每个任务只能指定一个回调函数。它也不能使用 try catch 捕获错误，不能直接 return 。

## 事件监听

在此模式下，异步任务的执行不取决于代码的顺序，而取决于某个事件是否发生。

下面是两个函数，意图是想让 fn2 在 fn1 执行结束后执行。我们为它绑定了一个事件。（这里是 JQuery 写法）

```js
fn1.on('done', fn2);
```

fn1 需要触发 `done` 事件，从而执行 fn2

```js
function f1() {
  setTimeout(function () {
    // ...
    f1.trigger('done');
  }, 1000);
}
```

优点：

+ 容易理解
+ 可以绑定多个事件，
+ 每个事件可以指定多个回调函数
+ 减少了耦合度。

缺点：

+ 整个程序变成了事件驱动型，运行流程很不清晰
+ 代码可读性降低

## Promise

关于 Promise 的更多看这里。

## Generator

Generator 函数是 ES 6 提供的一种异步解决方案，行为与传统函数不提供。它的最大特点就是可以控制函数的执行。

+ 语法上，首先可以把它理解成一个状态机，封装了多种状态。
+ Generator 还是一个 Iterator 对象生成函数。
+ 可暂停函数，yield 可暂停，next 方法可启动。每次返回的是 yield 后表达式的结果。
+ yield 函数本身没有返回值。next 方法可以带一个参数，该参数就会被当作上一个 yield 表达式的返回值。

 我们现在有三个文件内容都只有一句话，下一个请求依赖上一个请求的结果。

```js
const fs = require('fs')
function read() {
  return new Promise((resolve, reject) => {
    fs.readFile(file, 'utf8', (err, data) => {
      if (err) reject(err)
      resolve(data)
    })
  })
}
function *r() {
  let r1 = yield read('./1.txt')
  let r2 = yield read(r1)
  let r3 = yield (r2)
}

let it = r()
let { value, done } = it.next()
value.then(function(data) { // value是个promise
  console.log(data) //data=>2.txt
  let { value, done } = it.next(data)
  value.then(function(data) {
    console.log(data) //data=>3.txt
    let { value, done } = it.next(data)
    value.then(function(data) {
      console.log(data) //data=>结束
    })
  })
})
```

从上例看出，手动迭代 Generator 很麻烦，逻辑也优点绕，实际开发会配合 `co` 库去使用。

```js
function* r() {
  let r1 = yield read('./1.txt')
  let r2 = yield read(r1)
  let r3 = yield read(r2)
}
let co = require('co')
co(r()).then(function(data) {
  console.log(data)
})
```

上例如此就实现了。

我们可以将前文回调地狱的代码，改写为如下代码。

```js
function *fetch() {
    yield ajax(url, () => {})
    yield ajax(url1, () => {})
    yield ajax(url2, () => {})
}
let it = fetch()
let result1 = it.next()
let result2 = it.next()
let result3 = it.next()
```

## async / await

使用 async/await 它有以下特点：

+ 基于 Promise 实现
+ 非阻塞的
+ 使异步代码看起来像同步代码

我们将上例用 async / await 改写。

```js
async function readAll(params) {
  try {
    let p1 = await read(params, 'utf8')
    let p2 = await read(p1, 'utf8')
    let p3 = await read(p2, 'utf8')
  } catch (e) {
    throw new Error(e)
  }
}
```

## 总结

JavaScript 异步编程进化史：callback => promise => generator => async / await
