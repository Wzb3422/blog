---
title: JavaScript 之 Promise 原理
keywords: JavaScript
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510135331.png'
tags: Promise
categories:
  - JavaScript
date: 2020-05-10 13:52:56
---

Promise 表示一个异步操作的最终结果，与之进行交互的方式主要是 `then` 方法，该方法注册了两个回调函数，用于接收 `Promise` 的终值或 `Promise` 不能执行的原因。—— Promises/A+ 规范。

那么，它背后的原理是怎样的呢？我们一步一步来。

<!-- MORE -->

## 基础实现

我们假设有这么一个场景，需要获取用户的 id 并进行处理。

```js
// 不使用 Promise
http.get('some_url', result => {
  // do something
  console.log(result.id)
})

// Promise
new Promise(resolve => {
  http.get('url', result => {
    resolve(result.id)
  })
}).then(id => {
  // do something
  console.log(id)
})
```

这么一看，似乎是不用 `Promise` 更加简洁？

但，假设我们有几个前置异步请求。

```js
//不使用Promise        
http.get('some_url', function (id) {
    //do something
    http.get('getNameById', id, function (name) {
        //do something
        http.get('getCourseByName', name, function (course) {
            //dong something
            http.get('getCourseDetailByCourse', function (courseDetail) {
                //do something
            })
        })
    })
});

//使用Promise
function getUserId(url) {
    return new Promise(function (resolve) {
        //异步请求
        http.get(url, function (id) {
            resolve(id)
        })
    })
}
getUserId('some_url').then(function (id) {
    //do something
    return getNameById(id); // getNameById 是和 getUserId 一样的Promise封装。下同
}).then(function (name) {
    //do something
    return getCourseByName(name);
}).then(function (course) {
    //do something
    return getCourseDetailByCourse(course);
}).then(function (courseDetail) {
    //do something
});
```

这样，`Promise` 的链式调用就舒服了很多。

说到底，Promise 也还是使用回调函数，只不过是把它们