---
title: 跨域方案总结
date: 2020-12-10 19:37:48
tags: Browser
keywords: 跨域
thumbnail: 
categories: Browser
---

在前后端分离开发过程中，经常会遇到前端请求发出去了后端没有接收到请求的情况。一般是由浏览器的同源策略的跨域问题。我们来总结一下跨域问题的解决方案。

<!-- MORE -->

## 同源策略

同源的定义：如果两个 URL 的 `protocol` `port` `host` 都相同的话，则这两个 URL 是同源的。

不同源的 `XHR` 会被浏览器的同源策略拦截。

## 跨域解决方案

### 1. jsonp

由于跨域资源嵌入（*Cross-origin embedding*）一般是被允许的。我们可以使用 `<script>` 标签引入跨域的静态资源。基于此原理，可以通过动态创建 `<script>` ，再请求一个带参网址实现跨域通信。

**浏览器端代码**

```html
<script>
	let script = document.createElement('script');
  script.type = 'text/javascript';
  script.src = 'https://another.example.com/api/login?callback=handleCallback';
  document.head.appendChild(script);
  
  function handleCallback(res) {
    console.log(JSON.stringify(res));
  }
</script>
```

**Node.js 服务端代码**

```js
const querystring = require('querystring');
const http = require('http');
const server = http.createServer();

server.on('request', function(req, res) {
  const params = qs.parse(req.url.split('?')[1]);
  const fn = params.callback;
  
  res.writeHead(200, { 'Content-Type': 'text/javascrpt' });
  res.write(`${fn}(${JSON.stringify(params)})`);
  
  res.end();
});
```

不过，jsonp 只能实现 `GET` 请求。

### 2. document.domain + iframe

场景：主域相同，子域不同。

实现原理：两个页面通过 `document.domain` 设置为基础主域，即可实现同域。

网页（https://example.com）

```html
<iframe src="https://child.example.com"></iframe>
<script>
  var greet = 'hello world';
	console.log(document.domain); // 'example.com'
</script>
```

iframe（https://child.domain.com）

```html
<script>
	document.domain = 'domain.com';
  console.log(window.parent.greet); // hello world
</script>
```

### 3. location.hash + iframe





## Reference

+ [浏览器的同源策略 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

+ [常见的跨域解决方案 - Segment Fault](https://segmentfault.com/a/1190000011145364)

+ 