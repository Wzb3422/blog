---
title: 🤔WebAssembly的诞生与原理简述
keywords: WebAssembly
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511170830.png'
tags: WebAssembly
categories:
  - WebAssembly
date: 2020-05-11 17:03:56
---

WebAssembly 是一个可移植、体积小、加载快并且兼容 Web 的全新格式。谈谈我对它的诞生、现状、与发展的看法。

<!-- MORE -->

## 写在前面

本文内容仅代表个人观点，有异议欢迎交流。

## 为何而来

### 10天出生的 JavaScript

1995 年，Brendan Eich 用十天的事件创造了 JavaScript。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511185339.png" style="zoom: 67%;" />

当时 Netscape 遇见到，未来的网页需要变得动态。Netscape 的创始人 Mark Anderson 认为需要为 HTML 设计一种胶水语言，让网页设计师和兼职程序员能够使用它来组装图片和插件之类的组件，并且代码可以直接编写到网页标记中。

它的原本的目标开发人员是**网页设计师和兼职程序员**。

JavaScript 是解释型的，没有真正的类、没有静态类型、操作符重载、最初的时候连异常都没有。（异常后来加上去了）

Brenden Eich 十天做出的项目，完成了需求，市场反响不错。

随着 Web 的迅速发展，JavaScript 当然也跟着日渐蓬勃，各个浏览器都支持，成为了 Web 事实上的官方语言。所以，这个原本是为了「让网页设计师和兼职程序员能够使用它来组装图片和插件之类的组件」的语言，居然变成了互联网世界中不可或缺的语言之一。

到这时，也渐渐暴露出了 JavaScript 的问题。最主要的问题有两个：

+ 语法太灵活导致大型项目开发困难
+ 性能不能满足一些场景的需要

在这里，我们主要讨论性能的问题。

JavaScript 是一门解释型的语言。我们都知道解释型语言的一大特点就是慢。Web 应用越来越复杂，JavaScript 性能问题日益凸显。

### V8 Engine 的诞生

于是，Google 在 2009 年在 V8 中引入了 JIT 技术（Just in Time Compiling），通过各种变异优化直接将 JavaScript 编译成运行在 CPU 上的机器码。JavaScript 的性能提升了 20 - 40 倍。

<img src="https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511193501.png" style="zoom:50%;" />

不过 JIT 也有它的问题。例如：V8 会通过类型推断来减少对弱类型变量的拆装箱。但是 JavaScript 是动态类型的，开发人员很有可能在编写代码的时候改变变量的类型。这会导致 JIT 的重编译，有时候 V8 的性能提升，还没重编译的开销大。还有一些特殊情况，JIT 甚至都无法编译。

V8 提升性能这么大的另一个原因是，JavaScript 原来的执行效率实在太低了。

### 另寻它路

既然 JIT 遇到动态类型和一些语言功能的问题那是不是可以从语言层面上进行约束？

按照这个思路催生了其他的路径：

+ Microsoft 的 [ TypeScript ](http://www.typescriptlang.org/) 通过为 JS 加入静态类型检查来改进 JS 松散的语法，提升代码健壮性
+ Google 的 [ Dart ](https://www.dartlang.org/) 则是为浏览器引入新的虚拟机去直接运行 Dart 程序以提升性能

+ Mozilla 的 [ asm.js ](http://asmjs.org/) 是取 JavaScript 的子集，JavaScript Engine 针对 asm.js 进行性能优化

但是以上方式各有优缺点：

+ TypeScript 只是解决了 JS 语法松散的问题，最后还是需要编译成 JS 去运行，对性能没有提升
+ Dart 只能在 Chrome 预览版中运行，无主流浏览器支持，用 Dart 开发的人不多
+ asm.js 语法太简单、有很大限制，开发效率低

### WebAssembly 诞生

浏览器三大巨头分别提出了自己的解决方案，互不兼容。这违背了 Web 的宗旨。是技术的统一让 Web 走到了今天，因此形成一套新的规范去解决 JS 面临的问题迫在眉睫。

WebAssembly 诞生了，WebAssembly 是一种新的字节码格式，主流浏览器都已支持 WebAssembly。和 JavaScript 解释运行不同的是，WebAssembly 的字节码和底层机器码很相似，可以快速的装载运行，因此性能相对于 JavaScript 的解释有大幅度的提升。也就是说，WebAssembly 并不是一门编程语言，而是一份字节码标准，需要使用高级编程语言编译出字节码放到 WebAssembly 的虚拟机中运行（有点像 Java ）。浏览器厂商需要做的事情就是根据 WebAssembly 的规范实现虚拟机。

## WebAssembly 的原理

要弄清楚 WebAssembly 的原理，首先要知道计算机的原理。计算机由各种电子元件组成，为了方便表示，只存在开和闭两个状态，对应的二进制就是 0 和 1。计算机只认识 0 和 1，数字和逻辑都由二进制表示。也就是直接装载到计算机中运行的机器码，机器码的可读性极差。因此，人们编写高级语言，再编译成机器码。

由于不同的计算机 CPU 架构不同，机器码标准有所差别，常见的架构有 x86、AMD 64、ARM，因此高级语言编译成可执行代码时需要指定目标架构。

WebAssembly 是一种抹平了不同 CPU 架构的机器码，WebAssembly 的机器码不能放在任何一个平台上运行，但由于非常接近机器码，可以被非常快速的翻译为对应架构的机器码。因此，WebAssembly 的运行速度和机器码非常接近。

WebAssembly 的优点有：

+ 体积小。浏览器运行时只需要加载编译成的字节码，一样的逻辑比用字符串描述的 JS 文件小很多
+ 加载快。由于文件体积小，再加上无需解释运行，WebAssembly 能够更快的加载实例化，减少运行等待时间
+ 兼容性问题少。WebAssembly 是非常底层的字节码规范，制订好后很少变动，就算以后发生变化,也只需在从高级语言编译成字节码过程中做兼容。可能出现兼容性问题的地方在于 JS 和 WebAssembly 桥接的 JS 接口。

每个高级语言都去实现源码到不同平台的机器码的转换工作是重复的，高级语言只需要生成底层虚拟机(LLVM)认识的中间语言(LLVM IR)。

LLVM 能实现：

+ LLVM IR 到不同 CPU 架构机器码的生成
+ 机器码编译时性能大小优化

除此之外，LLVM 还是先了 LLVM IR 到 WebAssembly 字节码的编译功能。也就是说，只要高级语言能够转换到 LLVM IR，最终就能编译成 WebAssembly 字节码。目前能编译成 WebAssembly 的语言有：

+ AssemblyScript：语法与 TypeScript 一致，前端开发人员学习成本低。
+ C / C++：官方推荐方式
+ Rust
+ Kotlin
+ Golang

## 使用场景

绝大多数情况下，JavaScript 都能满足需求。当遇到了计算密集型 Web 应用，JavaScript 就显得力不从心了。

一些可能的场景：

+ 图片 / 视频编辑
+ 游戏：Web 运行 3A 游戏
+ 图像识别
+ 视频直播
+ VR / AR
+ CAD
+ 开发者工具：编辑器、调试器、编译器...
+ 加密工具

这么看来，WebAssembly 将会为 Web 带来更多的可能。我对它的未来很期待。

## 一些问题

WebAssembly 标准虽然已经定稿并且得到主流浏览器的实现，但目前还存在一些问题：

+ 浏览器兼容性不好，只有最新版本的浏览器支持，并且不同的浏览器对 JS WebAssembly 互调的 API 支持不一致
+ 生态工具不完善不成熟
+ WebAssembly 开发成本高。如果使用官方推荐的 C / C++ 的方式，来进行 Web 开发，开发周期会加长。C / C++ 不适合应「 三天一变的 Web 需求🤪」

## 参考资料 References

+ [WebAssembly 现状与实战](https://www.ibm.com/developerworks/cn/web/wa-lo-webassembly-status-and-reality/index.html)

+ [V8 与 JIT](http://pelli.ren/gitbooks/other_webperformance/javascript-performance/V8-and-JIT.html)

+ [Asm.js 维基百科](https://zh.wikipedia.org/wiki/Asm.js)

+ [WebAssembly 维基百科](https://zh.wikipedia.org/wiki/Asm.js)

+ [WebAssembly .org](http://webassembly.org/)

