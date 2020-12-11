---
title: Redux 源码阅读（一）createStore
date: 2020-12-08 20:36:42
tags: Redux
keywords: Redux
thumbnail: https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/1607431171098.png
categories: Redux
---

Redux 提供了可预测的状态管理方案。来看看的源码。

<!-- MORE -->

## OverView

本系列从日常使用的 API 自顶向下地阅读其源码的实现。首先查看 Redux 暴露出来的 API。

源码[链接](https://github.com/Wzb3422/redux/blob/master/src/index.ts#L61-L68)

```ts
export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```

看，都是熟悉的 API。本篇从 `createStore` 开始。

## 阅读源码之前

在阅读 `createStore` 之前，我们回顾一下 [Redux 文档中对 `createStore` 讲解](https://redux.js.org/api/createstore)。

官方示例

```ts
import { createStore } from 'redux'

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([action.text])
    default:
      return state
  }
}

const store = createStore(todos, ['Use Redux'])

store.dispatch({
  type: 'ADD_TODO',
  text: 'Read the docs'
})

console.log(store.getState())
// [ 'Use Redux', 'Read the docs' ]
```

store 能维护一个状态树，只能通过 `dispatch()` action 来改变状态。并且能订阅状态的改变以更新视图。这些功能都是怎么实现的？`store` 是通过什么方式维护状态？想要了解其背后的实现。带着问题阅读源码，有目的性，效率更高。

## 开始阅读

### 闭包

打开 `src/createStore.ts` ，首先看[声明的这几个变量](https://github.com/Wzb3422/redux/blob/master/src/createStore.ts#L101-L105)

```ts
function createStore() {
  // 一些错误处理
	let currentReducer = reducer // 当前使用的 reducer
  let currentState = preloadedState as S
  let currentListeners: (() => void)[] | null = []
  let nextListeners = currentListeners
  let isDispatching = false
  // ...
  
  return store;
}
```

没错，redux 维护状态的方式很简单，就是一个闭包，维护在变量 `currentState` 中。

### getState

从 `store` 中获取 State 也同样很简单。查看[源码](https://github.com/Wzb3422/redux/blob/master/src/createStore.ts#L125-L135)

```ts
function getState(): S {
  // 直接返回当前的 State
  return currentState as S
}
```

直接将 `currentState` 返回。这样看来，阅读源码似乎也不是非常困难。我们接着往下看。

### subscribe

[查看源码](https://github.com/Wzb3422/redux/blob/master/src/createStore.ts#L160-L198)

```ts
 function subscribe(listener: () => void) {
   	// 一些错误处理
    let isSubscribed = true // 标记是否注册，保证取消注册只能一次。

    ensureCanMutateNextListeners() // 浅拷贝一份 listener 数组
    nextListeners.push(listener) // 增加新的 listener

    return function unsubscribe() {
      if (!isSubscribed) { // 已经取消注册的不能再次取消。
        return
      }

      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api/store#subscribelistener for more details.'
        )
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1) // 删除该 listener
      currentListeners = null // 手动解除引用，释放
    }
  }
```

说明见注释。在这里我们需要注意 `ensureCanMutateNextListeners` 的用途。

```ts
function ensureCanMutateNextListeners() {
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice()
  }
}
```

当 `nextListeners` 与 `currentListeners` 引用同一个数组时，做一份浅拷贝，防止在 dispatch 过程中注册/注销 listener 而引发的 bug。

### dispatch

接下来，看看熟悉的[`dispatch` 函数的实现](https://github.com/Wzb3422/redux/blob/master/src/createStore.ts#L225-L259)。

```ts
function dispatch() {
  if (isDispatching) {
    throw new Error('Reducers may not dispatch actions.')
  }

  try {
    isDispatching = true
    currentState = currentReducer(currentState, action) // 使用 reducer 获取新的 state
  } finally {
    isDispatching = false // 保证会被执行
  }

  // 遍历调用所有注册的 listeners
  const listeners = (currentListeners = nextListeners)
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners[i]
    listener()
  }

  return action
  }
}
```

先通过 reducer 获取新的 state，并更新 currentState 。然后调用所有的 listeners。这里需要注意的是，将 `isDispatching = false` 放在了 `finally` 语句中，这样能够保证 `try` 块中的语句即使出现了异常，`isDispatching = false` 也能被执行。

### ReplaceReducer

查看[源码](https://github.com/Wzb3422/redux/blob/master/src/createStore.ts#L271-L297)

```ts
function replaceReducer<NewState, NewActions extends A>(
nextReducer: Reducer<NewState, NewActions>
): Store<ExtendState<NewState, StateExt>, NewActions, StateExt, Ext> & Ext {
  // TODO: do this more elegantly
  ;((currentReducer as unknown) as Reducer<
    NewState,
    NewActions
  >) = nextReducer

  dispatch({ type: ActionTypes.REPLACE } as A)
  // change the type of the store by casting it to the new store
  return (store as unknown) as Store<
    ExtendState<NewState, StateExt>,
    NewActions,
    StateExt,
    Ext
  > &
    Ext
}
```

将 currentReducer 替换为 nextReducer，并且改变 `store` 的类型。

## Reference

+ [redux - Github](https://github.com/Wzb3422/redux)

+ [createStore | Redux](https://redux.js.org/api/createstore)

+ [try...catch - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)

