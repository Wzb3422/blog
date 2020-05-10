---
title: JavaScript 之 「垃圾回收机制」
keywords: JavaScript
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510102418.png'
tags: 垃圾回收
categories:
  - JavaScript
date: 2020-05-09 17:43:56
---

## 垃圾回收 Garbage Collection

JavaScript 具有自动垃圾回收机制，也就是说，执行环境会负责管理代码执行过程中使用的内存。而在 C/C++ 之类的语言中，开发人员的一项基本任务就是手动跟踪内存的使用情况，这是造成众多问题的一个根源。在编写 JavaScript 时，开发人员不用再关心内存使用问题，所需内存的分配与无用内存回收完全实现了自动管理。垃圾收集器会周期性的进行垃圾回收。

<!-- MORE -->

## 可达性 Reachability

在了解具体的算法之前，我们先弄清楚一个内存管理的一个重要概念——可达性。

简单来说，「可达性的值」指的就是可以以某种方式访问或者使用的变量。

有一组基本的固有的可达值，由于显而易见的原因无法删除。如：

+ 当前执行函数的局部变量和参数
+ 当前调用链上的其他函数的变量和参数
+ 全局变量
+ 其他内部变量

这些值称为**根 (roots)**

如果变量能够从根或根开始的引用链被访问，那么该值是可达的。

### 简单的例子

```js
let user = {
  name: "John"
};
```

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510005236.png" alt="20200510005236.png" style="zoom:50%;" />

全局变量 `user` 引用了对象 `{name: "John"}`。这个对象是可达的。

接下来，我们将 `user` 变量解除引用。对于原先对象的引用丢失了。

```js
user = null;
```

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510010343.png" style="zoom:50%;" />

此时 `{name: "John"}` 对象变成了不可达的。无法访问到它，也没有对它的引用。垃圾收集器将会对它进行清除。

### 两个引用

来看看下面这个例子

```js
let user = {
  name: "John"
};
let admin = user;
```

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510010727.png" style="zoom:50%;" />

同样的，我们对 `user` 进行解除引用。

```js
user = null;
```

此时，对象仍能通过全局变量 `admin` 被访问，它还是可达的。当我们再将变量 `admin` 设为 null 时。它就会被垃圾清除了。

### 相互关联对象

现在看一个复杂的例子。

```js 
function marry(man, woman) {
  woman.husband = man;
  man.wife = woman;
  return {
    father: man,
    mother: woman
  }
}

let family = marry({name: "John"}, {name: "Ann"});
```

Marry 函数让两个对象「成为夫妻🎎」，新增一个引用对方的属性，并且返回一个对象。

在内存中的结构是这样的。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510011346.png" alt="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510011346.png" style="zoom:50%;" />

如图，所有的对象都是可达的。

现在，我们来删去两个引用。

```js
delete famliy.father;
delete family.mother.husband;
```

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510011721.png" style="zoom:50%;" />

如果只删除一个引用，显然是不够的。但我们这里删除了两个引用，没有对 `{name: "John"}`对象的引用。这个对象变成了不可达的。

垃圾回收之后。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510012038.png" style="zoom:50%;" />

### 无法访问的「孤岛」

若整个关联对象变成了不可达的，它全部都会被垃圾回收。

如上例，我们执行。

```js
family = null;
```

内存中就会是这样。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510012251.png" style="zoom:50%;" />

## 内部算法

### 标记 - 清除算法 mark-and-sweep

JavaScript 的基本垃圾回收算法为 「标记 - 清除」。

该算法一般由以下几个步骤：

1. The garbage collector takes roots and “marks” (remembers) them.
2. Then it visits and “marks” all references from them.
3. Then it visits marked objects and marks *their* references. All visited objects are remembered, so as not to visit the same object twice in the future.
4. …And so on until every reachable (from the roots) references are visited.
5. All objects except marked ones are removed.

> 直接 copy 英文原文，因为这一部分翻译成中文之后不太好理解，英文更加易懂。

举个例子，我们的内存中结构是这样的。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510013021.png" style="zoom:50%;" />

首先，标记根 (roots)。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510013453.png" style="zoom:50%;" />

下一步，标记它们对外的引用。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510013533.png" style="zoom:50%;" />

然后，继续标记对外的引用，重复。（如果有的话）

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510013632.png" style="zoom:50%;" />

现在，在标记过程中没有被访问到的变量，视为不可达的，并且被垃圾清除。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510013743.png" style="zoom:50%;" />

这就是 JS 垃圾清除的工作原理。JavaScript 引用做了许多优化，使其运行得快，并且不应行代码执行。

## 参考资料 Reference

+ [Garbage Collection - The JavaScript language](https://javascript.info/garbage-collection)
+ 《JavaScript 高级程序设计 第三版》
