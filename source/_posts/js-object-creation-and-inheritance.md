---
title: JavaScript 之 「对象的创建与继承」
keywords: JavaScript
thumbnail: 'https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200510104845.png'
tags: 对象
categories:
  - JavaScript
date: 2020-05-09 17:43:56
---

聊聊 JavaScript 的对象。

<!-- More -->

## 创建对象

JavaScript 可以实现多种对象创建模式。

### 字面量创建

```js
let person = {};
person.name = 'bob';
person.getName = function () {
  return this.name;
}
```

注意：`{}`也会从 `Object.prototype` 继承属性和方法。

### Object 构造函数

```js
let person = new Object();
person.name = 'bob';
person.getName = function () {
  return this.name;
}
```

### 工厂模式

工厂模式是一种常见的设计模式，这种模式抽象了创建对象的具体过程。

```js
function createPerson(name, age, job){
  var o = new Object(); // 显式地创建了对象
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function(){
  	alert(this.name);
  };
	return o;
}

let guy = createPerson("Nicholas", 29, "Software Engineer");
```



工厂模式解决了创建多个相似对象的问题，抽象了创建对象的具体过程。

但无法对对象进行识别，判断对象的类型。

### 构造函数模式

```js
function Person(name, age, job){
  this.name = name;
	this.age = age;
	this.job = job;
	this.sayName = function(){
    alert(this.name);
	};
}

let guy = new Person("Nicholas", 29, "Software Engineer");
```

与工厂模式相比，构造函数模式：

+ 没有显式地创建对象
+ 直接将属性与方法赋给了 this 对象
+ 没有 return 语句

按照约定，构造函数始终以大写字母开头。这个做法借鉴于其他 OO 语言，在 ES 中与其他函数区别开来。构造函数本身也是函数，只不过可以用来创建对象而已。

要使用构造函数创建 Person 实例，必须使用 new 操作符。

#### new 的调用过程

1. 创建一个新的对象
2. 将函数的作用域赋给该对象（这样 this 就指向了这个新对象）
3. 执行构造函数中的代码（为新对象添加属性与方法）
4. 返回新对象

例中的实例 `guy`有一个`constructor`属性，该属性指向构造函数 `Person`

```js
console.log(guy.constructor === Person); // true
```

#### 判断对象类型

`constructor`属性最初是用来标识对象类型的。但是，检测对象类型还是 `instanceof` 操作符更为可靠。本例中的 `guy`既是 `Person`的实例，同时也是 `Object`的实例。

```js
console.log(guy instanceof Persion); // true
console.log(guy instanceof Object); // true
```

使用构造函数模式能够将对象实例标识为一种特定的类型。

### 原型模式

每个函数都有一个 `prototype` 属性， 这个属性指向原型对象。

原型对象的用途是包含可以由特定类型的所有实例共享的属性与方法。

```js
function Person(){}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function(){
	alert(this.name);
};
var guy = new Person();
guy.sayName(); //"Nicholas"
var anotherGuy = new Person();
anotherGuy.sayName(); //"Nicholas"
console.log(guy.sayName === anotherGuy.sayName); // true
```

原型对象的属性与方法是所有对象共享的。

原型模式省略了构造函数传递初始化参数的环节，结果所有实例在默认情况下都将取得相同的属性值。它们共享同一个原型对象，这样的属性共享会产生很多问题。因此很少有人会单独使用原型模式。

### 组合使用构造函数模式与原型模式

构造函数模式用于定义实例属性，原型模式用于定义方法与共享的属性。这样，每一个实例都会有自己的一份实例属性的副本，但同时又有对共享方法的引用。

```js
function Person(name, age, job){
this.name = name; 3 this.age = age;
this.job = job;
this.friends = ["Shelby", "Court"];
 2
  }
Person.prototype = { constructor : Person, sayName : function(){
alert(this.name); }
}
var person1 = new Person("Nicholas", 29, "Software Engineer"); var person2 = new Person("Greg", 27, "Doctor");
4
5
6
7
8
  person1.friends.push("Van"); alert(person1.friends); //"Shelby,Count,Van" alert(person2.friends); //"Shelby,Count" alert(person1.friends === person2.friends); alert(person1.sayName === person2.sayName);
//false
//true
```

### 动态原型模式

有 OO 经验的开发人员在看到独立的原型模式与构造函数模式时可能会感到困惑。动态原型模式正是解决此问题的一个方案，它把所有的信息都封装在了构造函数中。而在必要的情况下，初始化原型。

该模式通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型。

```js 
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  if (typeof this.sayName != "function") {
		Person.prototype.sayName = function() {
      console.log(this.name);
    }
  }
}

let guy = new Person("Nicholas", 29, "Software Engineer");
guy.sayName();
```

这里只在 sayName() 方法不存在的情况下，才会把它添加到原型中。这段代码只会在初次调用构造函数时执行。此后，原型已经完成初始化。

### 寄生构造函数模式

寄生构造函数模式的思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后返回新的对象。但从表面上看，这个函数又很像是典型的构造函数。

```js
function Person(name, age, job){
  var o = new Object();
	o.name = name;
	o.age = age;
	o.job = job;
  o.sayName = function(){
		alert(this.name);
  };
	return o;
}
let friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName(); //"Nicholas"
```

本例中，`Person` 创建了一个对象，并以属性和方法初始化该对象，然后返回这个对象。除了使用 `new` 操作符并把包装的函数称为构造函数之外，这个模式和工厂模式其实是一模一样的。构造函数在不返回值的时候，默认返回新对象实例。而通过 `return` 语句，可以重写调用构造函数的返回值。

这个模式可以在特殊的情况下为对象创建构造函数。下面是一个具有特殊方法的数组。

```js
function SpecialArray() {
  let values = new Array();
  values.push.apply(values, arguments);
  values.toPipeString() = function() {
    return this.join("|")
  }
  return values;
}
```

使用场景：该模式下，返回的对象与构造函数或者与构造函数的原型属性之间没有关系。也就是说，构造函数返回的对象与在构造函数外部创建的对象没有什么不同。为此， 不能依赖 `instanceof` 操作符来确定对象类型。由于存在上述问题，我们建议在可以使用其他模式的情况下，不要使用这种模式。

### 稳妥构造函数模式

Douglas CrockField 发明了 稳妥对象 durable object 这个概念。指的是，没有公共属性，而且其方法也不引用 `this` 的对象。稳妥对象适合使用在一些安全的环境中，这些环境会禁用 `new` 和 `this` , 或者防止数据被其他程序篡改时使用。稳妥构造函数模式与寄生构造函数模式类似，但有两点不同，一是创建对象的实例方法不使用 `this` 。二是使用 `new` 构造函数。

```js
function Person(name, age, job) {
  var o = new Object();
  o.sayName = function () {
    console.log(name)
  }
	return o;
}
```

稳妥构造函数与寄生构造函数模式类似，也不能用 instanceof 判断对象类型。

## 对象继承

继承是 OO 语言中最为重要的一个概念。许多 OO 语言都支持两种继承方式：接口继承和实现继承。接口继承只继承方法签名，实现继承则继承实际的方法。ECMAScript 没有方法签名，只进行实现继承。而其实现继承主要是依靠原型链实现的。

### 原型链

我们首先来回忆一下构造函数、原型和实例的关系。每个构造函数都有一个原型对象，原型对象中包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。如果我们让原型对象指向另一个类型的实例，而实例都包含一个指向原型对象的内部指针。如果我们又让原型对象等于另一个类型的实例。如此递进，就构成了原型和实例的链条。这就是所谓的原型链。

```js
function SuperType() {
  this.property = true
}
SuperType.prototype.getSuperValue = function() {
  return this.property
}
function SubType() {
	this.subTypeProperty = false
}
SubType.prototype = new SuperType()
SubType.prototype.getSubValue = function() {
  return this.subProperty
}
var instance = new SubType()
console.log(instance.getSuperValue()) // true
```

在上面的代码中，我们没有使用 SubType 默认提供的原型， 而是给它换了一个新原型。这个新原型就是 SuperType 实例。新原型不仅仅有作为 SuperType 而有的所有属性与方法，而且内部还有一个指针，指向了 SuperType 的原型。

通过实现原型链，本质是扩展了原型搜索机制。

如上例，调用 `instance.getSuperValue()` 会经历三个步骤：

1. 搜索实例
2. 搜索 SubType.prototype (也就是 SuperType 实例)

3. 搜索 SuperType.prototype

最后一步找到该方法。若是找不到属性或者方法，搜索过程会一直查询到原型链的末端。

原型继承也有它存在的问题：

1. 原型对象上的引用类型的值会被所有实例共享
2. 创建自子类型时，不能向超类型的构造函数中传递参数

### 借用构造函数

 在解决原型包含引用类型值的问题过程中，开发人员使用一种叫做借用构造函数的技术（有时也称经典继承）。其思想十分简单，即在子类型的构造函数中调用超类型的构造函数。

```js
function SuperType() {
  this.colors = ['red', 'yellow', 'green']
}
function SubType() {
  SuperType.call(this)
}
```

这样，在每个实例上。都会有自己的 `colors` 属性副本了。

#### 向超类传递参数

相对于原型链而言，借用构造函数可以在子类的构造函数中向超类的构造函数传递参数。

```js
function SuperType() {
  this.name = name
}
function SubType() {
  SuperType.call(this, 'Bob')
  this.age = 29
}
```

### 组合继承

有时也称伪经典继承，指的是将原型链和借用构造函数组合到一，发挥二者之长。实现思路就是使用原型链实现对原型属性和方法的继承，使用借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例有它自己的属性。

```js
function SuperType(name) {
  this.colors = ['red', 'yellow', 'green']
  this.age = name
}
SuperType.prototype.sayName = function() {
  console.log(this.name)
}
function SubType(name, age) {
  SuperType.call(this, name)
  this.age = age
}
SubType.prototype = new SuperType()
SubType.prototype.constructor = SuperType
SubType.prototype.sayAge = function() {
  console.log(this.age)
}
```

组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点。而且，`instanceof` 和 `isPrototypeOf` 也能够识别基于组合继承创建的对象。

### 原型式继承

借助原型可以基于已有的对象创建新的对象。

```js
function object(o) {
  function F() {}
  F.prototype = o
  return new F()
}
```

本质上讲，该函数对传入的对象进行了一次浅拷贝。

ES5 通过 `Object.create()` 方法规范了原型式继承。

> Object.create(proto[, propertiesObject])

使用原型式继承也要注意，引用类型的值始终会共享相应的值，

### 寄生式继承

思路与工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数内部以某种方式来增强对象，最后返回对象。

```js
function createAnother(original) {
  let clone = object(original)
  clone.sayHi = function() {
    console.log('Hi')
  }
  return clone
}
```

### 寄生组合式继承

即通过