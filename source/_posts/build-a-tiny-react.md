---
title: 造一个简单的基于 Fiber Reconcile 的 React
date: 2020-11-30 20:34:04
tags: React
keywords: React
thumbnail: https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/0daee0d28b327a07cd64aa12657e86da6e9e0264.png
categories: React
---

对于很多开发者来说，框架就像是一个黑盒。现我们从 React 的运行机制谈起，到自己实现一个简单的 React 。打开这个黑盒子。

<!-- MORE -->

## React 背后的机制

### 什么是 React

在探讨 React 背后的机制之前，先谈谈我所理解的 React 。从语义上来说「React」，即因为某些因素而对变化作出「反应」，这个「反应」要尽可能的快，并且不影响用户体验。一句话概括，React 的核心是**体验良好的快速响应**。

### React 运行架构

React 从 v15 到 v16 重构了它的整个运行架构，由 stack reconciliation 升级到了 fiber reconciliation 。来看看背后的原因。

#### React 15 - Stack Reconciliation

V15 的架构分为两层。

+ Reconciler - 负责找出变化的组件
+ Renderer - 将变化的组件渲染到页面

我们可以通过调用 `this.setState()` `this.forceUpdate()` `React.render()` 等 API 触发更新。触发更新时，React 会调用 `mountComponent` / `updateComponent` 方法。这两个方法都是通过递归来更新组件的。递归信息都是保存在 call stack 中的，因此这种运行架构称为 Stack Reconciliation。

递归一旦开始就无法中断，假如组件层级非常深，递归更新时间超过了 `16ms` （实际上阈值小于这个数值），用户交互就会卡顿。为解决这个问题，我们可以用可中断的异步更新代替同步的更新。可是递归不能中断，如果我们采用某种手段中断了递归，可这样的异步更新需要一直保留着完整的 call stack，这显然是低效的。

#### React 16 - Fiber Reconciliation

为了解决递归更新不可中断，并且优化用户体验的问题。React 16 采用了新的运行架构。它分为三层。

+ Scheduler - 调度任务的优先级，高优先级的任务先进入 Reconciler
+ Reconciler - 负责找出变化的组件
+ Renderer - 将变化的组件渲染到页面上

通过获知浏览器当前周期内剩余时间是否足够以作为任务中断的标准，并且通知我们。你可能会想到 `window.requestIdleCallback` ，但是由于一些原因，React 放弃使用了这个 API。React 实现了更为完备的 `requestIdleCallback polyfill` ，这就是 Scheduler 。除了触发空闲回调之外，它还提供任务优先级设置。

在 React 15 的时候，是通过递归处理虚拟 DOM 的。为了实现可中断的任务，并且更好的处理任务的优先级。React 16 引入了 Fiber ，并且将递归转变成了可中断的循环。

参考源码 [这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1673)

```js
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
```

每次循环都会通过 `shouldYield()` 判断当前是否有剩余时间。

那么，什么是 Fiber 呢？我们先看看源码中 `FiberNode` 的定义。

参考源码 [这里](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactFiber.new.js#L117)

```js
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // 对应的 React Element
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // Fiber Node 间连接的引用，构成 Fiber Tree
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;
	// 工作单元
  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  
  this.effectTag = NoEffect;
  this.subtreeTag = NoSubtreeEffect;
  this.deletions = null;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;
	// 调度优先级
  this.lanes = NoLanes;
  this.childLanes = NoLanes;
	// 指向另一 Fiber Tree 中对应节点
  this.alternate = null;
}
```

每一个 Fiber 都对应着一个 React Element，FiberNode 储存了 React Element 上的状态，并标记了变化，最终与其它 FiberNode 连接成为 Fiber Tree。此时，在 Reconcile 时操作的对象是 FiberTree，FiberTree 上记录了本次更新变化的节点与状态，Reconciler 负责循环地将其与上一次的 FiberTree 比较即可找出变更的节点，最终把变更交给 Renderer。

在 React 中会存在两颗 FiberTree。对应着屏幕上显示内容的称为 currentFiberTree，正在内存中记录变化的未被渲染的称为 workInProgressTree。两棵树通过 `alternative` 属性连接。

```js
currentFiber.alternative === workInProgressFiber;
workInProgressFiber.alternative === currentFiber;
```

## 简单实现

了解了基本原理之后，我们来简单实现一个 React 。最终的成品在[这里](https://github.com/Wzb3422/build-my-own-react/)。下面只讲关键思路。

我们从我们最熟悉的地方开始。

```js
import React from './react';
const container = document.getElementById("root")
function App() {
  const [state, setState] = React.useState(0);
  return (
    <div>
      <h1>Hello Function Components</h1>
      <p>Click me, count: {state}</p>
      <button onClick={() => setState(s => s + 1)}>Plus one</button>
    </div>
  )
}
React.render(<App />, container);
```

JSX 会被 Babel 编译成 `React.createElement`，由此得到虚拟 DOM。我们先来实现它。

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object" ? child : createTextElement(child)
      )
    }
  };
}
```

对于文本节点，会处理为 TextNode。然后这些 React Element 对象会交给 `React.render()`。

```js
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element]
    },
    alternate: currentRoot
  };
  deletions = [];
  nextUnitOfWork = wipRoot;
}
```

Render 接收到的是顶层元素与 container，将 wipRoot 指向了 `<App />` 的 Fiber 节点。

然后，我们需要创建 `workLoop` 。

```js
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }

  if (!nextUnitOfWork && wipRoot) {
    commitRoot();
  }

  requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);
```

这里不断的调用 workLoop 并且判断是否有剩余时间，确定何时开始工作。当 React.render 调用之后。`nextUnitOfWork` 指向了 `wipRoot` 。一旦剩余时间充足，就进入 `performUnitOfWork`。这里的 work 都指的是 Fiber 节点，Fiber 节点是 Fiber Reconciler 的工作单元。

```js
function performUnitOfWork(fiber) {
  const isFunctionComponent = fiber.type instanceof Function;
  // 根据 React Element 类型来创建
  if (isFunctionComponent) {
    updateFunctionComponent(fiber);
  } else {
    updateHostComponent(fiber);
  }
  if (fiber.child) {
    return fiber.child;
  }
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.parent;
  }
}
```

performUnitOfWork 首先根据 React Element 类型来调用 `updateXXXComponent`，其中进行 Fiber 节点之间的比对与变更标记。然后进行深度优先访问。

```js
function reconcileChildren(wipFiber, elements) {
  let index = 0;
  let oldFiber = wipFiber.alternate && wipFiber.alternate.child;
  let prevSibling = null;

  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;

    const sameType = oldFiber && element && element.type == oldFiber.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: "UPDATE"
      };
    }
    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: "PLACEMENT"
      };
    }
    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index === 0) {
      wipFiber.child = newFiber;
    } else if (element) {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}
```

reconcileChildren 会对新旧 Fiber 节点进行比对，如果 sameType 即会复用 DOM 节点，并只更新属性，并且打上更新标记。打 effectTag 来标记更新的好处是，任务都分布在节点上，这样是可中断的。最后都交给 commit 阶段进行执行。

```js
function commitWork(fiber) {
  if (!fiber) {
    return;
  }

  let domParentFiber = fiber.parent;
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent;
  }
  const domParent = domParentFiber.dom;

  if (fiber.effectTag === "PLACEMENT" && fiber.dom != null) {
    domParent.appendChild(fiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom != null) {
    updateDom(fiber.dom, fiber.alternate.props, fiber.props);
  } else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParent);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```

## 杂谈

自己实现了一遍 React 机制之后，感觉运行过程了然于胸。之后会多尝试实践性的原理学习，这样效率很高也很有成就感～



## Reference

+ [facebook/react - Github](https://github.com/facebook/react)

+ [React 技术揭秘](https://react.iamkasong.com/)

+ [Codebase overview - Reactjs](https://reactjs.org/docs/codebase-overview.html)

