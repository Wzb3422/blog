---
title: 🐣this、call 和 apply
keywords: JavaScript
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/whatisthisdas1319sdvn2.png'
tags: JavaScript
categories:
  - JavaScript
date: 2020-02-08 12:07:56
---

在 JavaScript 中，this 总是指向一个对象，具体指向哪个对象是运行时**基于函数执行环境**而动态绑定的。

<!-- MORE -->

## this 的指向
除去不常用的 with 与 eval 的情况之外，在实际应用中，this 的指向大致可以分为下面4种。
### 1. 作为对象的方法调用（隐式绑定）
当函数作为对象方法被调用时，this 指向该对象。

```javascript
const obj = {
	name: 'Bob',
  getName: function() {
  	console.log(this.name);
  }
};
obj.getName(); // Bob
```

### 2. 作为普通函数调用（默认绑定）
当函数不作为对象方法被调用时，也就是普通函数调用的方式，此时的 this 总是指向全局对象，在浏览器环境中，这个全局对象是 window 对象。

```javascript
window.name = 'I am window';
function getName() {
	return this.name;
}
console.log(getName()); // I am window
```

### 3. 作为构造函数调用（new 绑定）
构造函数和普通函数外表一模一样，它们的区别在于被调用的方式。当使用 new 运算符调用函数时，该函数总会返回一个对象，通常情况下，构造函数里的 this 就指向这个对象。

```javascript
const MyClass = function() {
	this.name = 'Bob';
};
let obj = new MyClass();
console.log(obj.name); // Bob
```

### 4. Function.prototype.call 或 Function.prototype.apply 调用（显式绑定）
跟普通的函数调用相比，用 Function.prototype.call 或 Function.prototype.apply 可以动态地改变传入函数的 this。

```javascript
const person1 = {
	name: 'Bob',
  getName() {
  	return this.name;
  }
}
const person2 = {
	name: 'Alice'
}
console.log(person1.getName); // Bob
console.log(person1.getName.call(person2)); // Alice
```

## 丢失的 this
这是一个经常遇到的问题，先看看这个例子。

```javascript
const obj = {
	name: 'Bob',
  getName() {
  	return this.name;
  }
};
console.log(obj.getName()); // Bob
let getNameFn = obj.getName;
console.log(getNameFn()); // undefined
```

在第七行，函数作为对象方法被调用，this 指向的是 obj ，此时 this.name 为 obj.name。但通过 getNameFn 以普通函数调用时，此时的 this 指向 window，所以结果为 undefined 。

若想使用 getNameFn 来访问 obj 的 name 属性，可以使用 apply/call 。

```javascript
getNameFn.call(obj); // Bob
```

## 不关心 this 的时候
如果把 null 或者是 undefined 作为 this 的绑定对象传入 call、apply 或者 bind ，这些值在调用时会被忽略，实际上应用的是默认绑定规则。当使用 call/apply 借用对象方法的时候，有时我们并不会关心 this ，但仍需要传入一个占位值，这时候我们可能会选择传入 null。然而，使用 null 来忽略 this 的绑定可能会产生一些副作用。如果某个函数确实使用了 this （可能是一个第三方库中的函数），那默认绑定 this 到全局对象，这将可能导致不可预计的后果。
### 安全的 this
对于这个问题，一个“更安全”的做法是传入一个特殊的对象，把 this 绑定到这个对象上不会对你的程序产生副作用。我们可以创建一个 DMZ (demilitarized zone, 非军事区) ，它是一个空的委托对象。这时将 this 绑定到这个 DMZ 对象中，不会对全局对象产生不必要的影响。
# call 和 apply
ECMAScript 3 给 Function 的原型定义了两个方法，它们是 Function.prototype.call 和 Function.
prototype.apply。能够熟练使用这两个方法，是我们成为真正的 JavaScript 程序员的重要一步。
## call 和 apply 的区别
这两个方法的作用一模一样，都可以用来修改 this 的指向。区别在于，传入参数的形式不同。

apply 接收两个参数，第一个参数指定了函数的 this 指向，第二个参数为一个带下标的集合，可以为数组或类数组。apply 方法把这个集合中的元素作为参数传递给被调用的函数。
```javascript
func.apply(thisArg, [argsArray])
```

call 传入的参数数量不固定，第一个参数指定函数的 this 指向，之后的参数依次作为参数传递给被调用的函数。

```javascript
func.call(thisArg, arg1, arg2, arg3, ...);
```

当使用 call / apply 的时候，如果我们传入的第一个参数为 null 函数体内的 this 会指向默认的宿主对象，浏览器中为 window。
## call 和 apply 的用途
### 1. 改变 this 的指向
call 和 apply 最常见的用途就是改变 this 的指向。

```javascript
var obj1 = { 
  name: 'Bob'
};
var obj2 = {
  name: 'Alice'
};
window.name = 'window';
const getName = function() {
	return this.name;
}
getName(); // window
getName.apply(obj1); // Bob
getName.call(obj2); // Alice
```

在实际开发过程中，我们经常会碰到 this 丢失的情况。看下面这个例子。

```javascript
document.getElementById('btn').onclick = function() {
	console.log(this.id);
  const showId = function() {
  	console.log(this.id);
  }
  showId(); // undefined
}
```

第六行的 showId() 函数是以普通函数调用的，其 this 指向 window 。我们原本的目的是对获取到的 btn 进行进一步操作。因此，需要修正 this 的指向。

```javascript
document.getElementById('btn').onclick = function() {
	console.log(this.id);
  const showId = function() {
  	console.log(this.id);
  }
  showId.call(this); // btn
}
```

### 2. Function.prototype.bind
bind 方法创建一个新的函数，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。大部分浏览器都已经实现来这个方法，我们也可以来模拟一个。

```javascript
Function.prototype.bind = function(context) {
	let self = this;
  return function() {
  	return self.apply(context, arguments);
  }
};
```

这是一个简化的 bind 实现，通常我还会把它实现得复杂一点。

```javascript
Function.prototype.bind = function() {
  const self = this; // 保存原函数
	const context = Array.shift.call(arguments); // 需要绑定的 this 上下文
  const args = Array.slice.call(arguments); // 剩余的参数转为数组
  return function() {
  	return self.apply(context, Array.concat.call(args, Array.slice.call(arguments)))
  }
}
```

### 3. 借用其他对象的方法
函数的参数列表 arguments 是一个类数组对象，虽然它也有“下标”，但它并非真正的数组，
所以也不能像数组一样，进行排序操作或者往集合里添加一个新的元素。这种情况下，我们常常
会借用 Array.prototype 对象上的方法。比如想往 arguments 中添加一个新的元素，通常会借用
Array.prototype.push。

```javascript
(function() {
	Array.prototype.push.call(arguments, 3);
  console.log(arguments); // [1, 2, 3]
})(1, 2);
```

在操作 arguments 的时候，我们经常地借用 Array.prototype 上的方法。想把 arguments 转化为真正的数组的时候，我们可以使用 Array.prototype.slice 方法。想截去 arguments 的第一个元素的时候，我们又可以使用 Array.prototype.shift。这种机制的内部实现原理是什么呢？我们来看看 V8 引擎的源码。

```javascript
function ArrayPush() {
	var n = TO_UINT32( this.length );
  var m = %_ArgumentsLength();
  for (var i = 0; i < m; i++) {
  	this[1 + n] = %_Arguments( i );
  }
  this.length = n + m;
  return this.length;
}
```

通过这段代码我们能看到，Array.prototype.push 其实是一个复制属性的过程，把参数按照下标添加到被 push 的对象上面，并修改这个对象的 length 属性。至于被修改的对象是数组还是类数组，并不重要。由代码可知，对象只需要满足两个条件：

1. 对象可以存取属性
1. 对象的 length 属性可读写
