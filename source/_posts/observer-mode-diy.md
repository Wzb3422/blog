---
title: 👀手动实现一个观察者模式
keywords: JavaScript
tags: 对象
categories:
  - JavaScript
date: 2020-02-11 17:43:56
---

观察者模式在前端中随处可见。也叫 **发布订阅模式 **。接下来看一个面试题，手动来实现一个。

<!-- MORE -->

## 题目
**微信前端面试题：**
编写一个 People 类，使其的实例具有监听事件、触发事件、解除绑定功能。（实例可能监听多个不同的事件，也可以去除监听事件）

```typescript
class People {
  constructor(name) {
    this.name = name
  }

  // TODO: 请在此处完善代码

  sayHi() {
    console.log(`Hi, I am ${this.name}`)
  }
}


/* 以下为测试代码 */
const say1 = (greeting) => {
  console.log(`${greeting}, nice meeting you.`)
}

const say2 = (greeting) => {
  console.log(`${greeting}, nice meeting you, too.`)
}

const jerry = new People('Jerry')
jerry.sayHi()
// => 输出：'Hi, I am Jerry'

jerry.on('greeting', say1)
jerry.on('greeting', say2)

jerry.emit('greeting', 'Hi')
// => 输出：'Hi, nice meeting you.' 和 'Hi, nice meeting you, too'

jerry.off('greeting', say1)
jerry.emit('greeting', 'Hi')
// => 只输出：'Hi, nice meeting you, too'
```

## 实现
题目意图很明确，需要我们手动实现一个观察者，能够监听、发布、取消订阅。下面是我的实现。

```typescript
interface Observer {
  on(type: string, callback: (args: any) => any): void
  emit(type: string, ...args): void
  off(type: string, callback: (args: any) => any): void
}

class People implements Observer {
  constructor(name) {
    this.name = name
  }
  private name = 'John Joe'
  private events: {[props: string]: ((args: any) => any)[]} = {}
  // 订阅事件
  public on(type: string, callback: (args: any) => any) {
    // 已存在该事件
    if (this.events[type]) {
      this.events[type].push(callback)
    } else {
      this.events[type] = [callback]
    }
  }
  public emit(type: string, ...args) {
    if (this.events[type]) {
      this.events[type].forEach(item => {
        item.apply(this, args)
      })
    } else {
      console.log('此事件暂未被任何观察者订阅')
    }
  }
  public off(type, callback) {
    if (this.events[type]) {
      this.events[type].splice(this.events[type].indexOf(callback), 1)
      console.log(`Unsubscribed event (type: ${type})`)
    } else {
      console.log(`not such event (type: ${type}`)
    }
  }

  sayHi() {
    console.log(`Hi, I am ${this.name}`)
  }
}


/* 以下为测试代码 */
const say1 = (greeting) => {
  console.log(`${greeting}, nice meeting you.`)
}

const say2 = (greeting) => {
  console.log(`${greeting}, nice meeting you, too.`)
}

const jerry = new People('Jerry')
jerry.sayHi()
// => 输出：'Hi, I am Jerry'

jerry.on('greeting', say1)
jerry.on('greeting', say2)

jerry.emit('greeting', 'Hi')
// => 输出：'Hi, nice meeting you.' 和 'Hi, nice meeting you, too'

jerry.off('greeting', say1)
jerry.emit('greeting', 'Hi')
// => 只输出：'Hi, nice meeting you, too'

```

