---
title: 🕴🏻单例模式
keywords: Design Pattern
tags: Design Pattern
categories:
  - Design Pattern
date: 2020-01-17 09:37:56
---

## 定义
单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点。
<!-- MORE -->
## 单例模式的实现
实现一个单例模式并不复杂，只需要用一个变量来标示当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象。
### 简单实现
简单实现如下：

```javascript
var Singleton = function(name) {
	this.name = name;
  this.instance = null;
}

Singleton.prototype.getName = function() {
	console.log(this.name);
}
Singleton.getInstance = function() {
	if (!this.instance) {
  	this.instance = new Singleton(name);
  }
  return this.instance;
}

var a = Singleton.getInstance('Bob');
var b = Singleton.getInstance('Alice');

console.log(a === b); // true
```

或者

```javascript
var Singleton = function(name) {
	this.name = name;
}

Singleton.prototype.getName = function() {
	console.log(this.name);
}

Singleton.prototype.getInstance = (function() {
	var instance = null;
  return function(name) {
  	if (!instance) {
    	instance = new Singleton(name);
    }
    return instance;
  }
})()
```

我们通过 Singleton.getInstance 来获取 Singleton 的唯一对象，这种方式相对简单。但有一个问题，这种方法增加了这个类的“不透明性”。使用者必须知道这是一个单例类，跟使用 new 方式获取对象不同，这里需要使用 Singleton.getInstance 来获取对象。
我们需要更好的方法来实现单例模式。
### 透明的单例模式
我们现在的目标是实现一个“透明”的单例类，用户从这个类创建对象的时候，可以像和使用任何其他普通类一样。在下面的日子中，我们将使用 CreateDiv 单例类，它的作用是在页面中创建唯一的 div 节点。

```javascript
var CreateDiv = (function() {
	var instance = null;
  var CreateDiv = functiono(html) {
  	if (instance) {
    	return instance;
    }
    this.html = html;
    this.init();
    return instance = this;
  }
  CreateDiv.prototype.init = function() {
  	var div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
  }
  return CreateDiv;
})()

var a = CreateDiv('A');
var b = CreateDiv('B');

console.log(a === b); // true
```

虽然现在完成了一个透明的单例类的编写，但是它同样存在一些缺点。
为了把 instance 封装起来，我们使用了自执行的匿名函数与闭包，这增加了一点程序的复杂度。
再看看 Singleton 构造函数

```javascript
var CreateDiv = functiono(html) {
  if (instance) {
    return instance;
  }
  this.html = html;
  this.init();
  return instance = this;
}
```

在这段代码中，CreateDiv 实际上负责了两件事情，一是创建对象和执行 init 方法，二是保证只有一个对象，这违背了单一职责原则。

### 用代理实现单例模式

现在我们通过引入代理类的方式，来解决上面提到的问题。

```javascript
var CreateDiv = function(html) {
	this.html = html;
  this.init();
}

CreateDiv.prototype.init = function() {
	var div = document.createElement('div');
  div.innerHTML = this.html;
  document.body.appendChild(div);
}

var ProxySingletonCreateDiv = (function() {
	var instance = null;
  return function(html) {
    if (!instance) {
      instance = new CreateDiv(html);
    }
    return instance;
  }
})()

var a = new ProxySingletonCreateDiv('A');
var b = new ProxySingletonCreateDiv('B');

console.log(A === B); // true
```

通过引入代理类的方式，我们完成了一个单例模式的编写。与之前不同的是，现在我们把负责管理单例的逻辑迁移到了代理类 ProxySingletonCreateDiv 中。由此，CreateDiv 就变成类一个普通的类，它与代理类组合起来可以达到单例模式的效果。

