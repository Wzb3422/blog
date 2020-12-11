---
title: Redux 源码阅读（二）combineReducers
date: 2020-12-08 20:36:42
tags: Redux
keywords: Redux
thumbnail: https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/1607704737336.png
categories: Redux
---

随着 Web 应用变得越来越复杂，你需要将 redux 状态进行拆分，每个模块使用独立的 reducer。你一定离不开 combineReducers 。

<!-- MORE -->

## 阅读源码之前

和往常一样，在阅读源码之前，回顾一下[官方文档对该 API 的描述](https://redux.js.org/api/combinereducers)。

`combineReducers` 是一个 helper function，接收多个 reducing function，返回一个 resulting reducer。resulting reducer 执行的时候会调用所有的 child reducer，并且将它们的结果以一个对象返回，并给每个 `state` 确定命名空间。命名空间取决于传递给 `combineReducers` 的对象的 `key` 。

`combineReducers` 只有一个参数——reducers 组成的对象。

`combineReducers` 实现了一些检查规则，以减少开发中遇到的问题：

1. 如果遇到预期之外的 `ActionType`，你应该将 `state` 原样返回。

2. 永远不要 `return undefined` 。`combineReducers` 会在这种情况下抛出错误。

3. 如果 `reducers` 接收的 `state` 为 `undefined` ，你应该返回 `initial state` 。

另外，`combineReducers` 会对你的 `reducer` 函数进行检查（传 `undefined` 给它们），即使你定义了 `initial state`。

同样，我们要带着问题去阅读源码。从上文的描述，我产生了以下问题：

1. `combineReducers` 的 namespace 是如何实现的？
2. `combineReducers` 的检查是如何实现的？

带着问题，我们开始阅读源码。

## 开始阅读源码

首先，对用户输入的 reducers 对象进行一次遍历。确保每一项都有对应的 `reducer` ，并做一份拷贝至 `finalReducers` 。之后的操作都将会使用检查过的 `finalReducers` 。

对应源码[地址](https://github.com/Wzb3422/redux/blob/master/src/combineReducers.ts#L139-L155)。

```ts
export default function combineReducers(reducers: ReducersMapObject) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers: ReducersMapObject = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)
```

然后，对 `finalReducers` 进行传递 `undefined` 检查。保存捕获到的错误（如果有的话）。

对应源码[地址](https://github.com/Wzb3422/redux/blob/master/src/combineReducers.ts#L164-L169)

```ts
let shapeAssertionError: Error
try {
  assertReducerShape(finalReducers)
} catch (e) {
  shapeAssertionError = e
}
```

我们来看看 `assertReducerShape` 的实现。

对应源码[地址](https://github.com/Wzb3422/redux/blob/master/src/combineReducers.ts#L77-L107)

```ts
function assertReducerShape(reducers: ReducersMapObject) {
  // 对 reducers 进行遍历
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key]
    // 给 reducers 传 undefined，看它是否会返回 initial state
    const initialState = reducer(undefined, { type: ActionTypes.INIT })

    if (typeof initialState === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined during initialization. ` +
          `If the state passed to the reducer is undefined, you must ` +
          `explicitly return the initial state. The initial state may ` +
          `not be undefined. If you don't want to set a value for this reducer, ` +
          `you can use null instead of undefined.` // 这里说明了，如果要给值设为空，应该设置其为 null 而不是 undefined
      )
    }
		// 给 reducer 未知的 action，看它是否会直接返回 state
    if (
      typeof reducer(undefined, {
        type: ActionTypes.PROBE_UNKNOWN_ACTION()
      }) === 'undefined'
    ) {
      throw new Error(
        `Reducer "${key}" returned undefined when probed with a random type. ` +
          `Don't try to handle ${ActionTypes.INIT} or other actions in "redux/*" ` +
          `namespace. They are considered private. Instead, you must return the ` +
          `current state for any unknown actions, unless it is undefined, ` +
          `in which case you must return the initial state, regardless of the ` +
          `action type. The initial state may not be undefined, but can be null.`
      )
    }
  })
}
```

如代码中注释所写。`assertReducerShape` 对 `reducers` 进行遍历，检查每一个 `reducer` 是否会在 init 时返回 `initial state`，并且处理未知的 `action` 时能够直接返回 `state`。

然后，`combineReducers` 返回一个函数 `combination` 。首先进行一些错误处理。

对应源码[地址](https://github.com/Wzb3422/redux/blob/master/src/combineReducers.ts#L171-L189)

```ts
return function combination(
	state: StateFromReducersMapObject<typeof reducers> = {},
  action: AnyAction
	) {
  // 抛出 shapeAssertionError，如果有的话
  if (shapeAssertionError) {
    throw shapeAssertionError
  }
	// 在非生产环境警告「获取到与预期的 state 结构不一致的问题」
  if (process.env.NODE_ENV !== 'production') {
    const warningMessage = getUnexpectedStateShapeWarningMessage(
      state,
      finalReducers,
      action,
      unexpectedKeyCache
    )
    if (warningMessage) {
      warning(warningMessage)
    }
  }
```

`shapeAssertionError` 不必赘述。至于 `getUnexpectedStateShapeWarningMessage` ，先看看其实现。

对应的源码[地址](https://github.com/Wzb3422/redux/blob/master/src/combineReducers.ts#L25-L75)

```ts
function getUnexpectedStateShapeWarningMessage(
  inputState: object,
  reducers: ReducersMapObject,
  action: Action,
  unexpectedKeyCache: { [key: string]: true }
) {
  const reducerKeys = Object.keys(reducers) // reducer 的 namespace
  // 问题来源，根据 action 的类型判断
  const argumentName =
    action && action.type === ActionTypes.INIT
      ? 'preloadedState argument passed to createStore'
      : 'previous state received by the reducer' 
	// 	如果没有 reducer
  if (reducerKeys.length === 0) {
    return (
      'Store does not have a valid reducer. Make sure the argument passed ' +
      'to combineReducers is an object whose values are reducers.'
    )
  }
	// 判断是否为 「纯对象」
  if (!isPlainObject(inputState)) {
    const match = Object.prototype.toString // 注意这里的 `toString()` 调用方式
      .call(inputState)
      .match(/\s([a-z|A-Z]+)/)
    const matchType = match ? match[1] : ''
    return (
      `The ${argumentName} has unexpected type of "` +
      matchType +
      `". Expected argument to be an object with the following ` +
      `keys: "${reducerKeys.join('", "')}"`
    )
  }
	// 检查 inputState 中是否有额外的属性
  const unexpectedKeys = Object.keys(inputState).filter(
    key => !reducers.hasOwnProperty(key) && !unexpectedKeyCache[key]
  )

  unexpectedKeys.forEach(key => {
    unexpectedKeyCache[key] = true
  })

  if (action && action.type === ActionTypes.REPLACE) return
	// 返回错误
  if (unexpectedKeys.length > 0) {
    return (
      `Unexpected ${unexpectedKeys.length > 1 ? 'keys' : 'key'} ` +
      `"${unexpectedKeys.join('", "')}" found in ${argumentName}. ` +
      `Expected to find one of the known reducer keys instead: ` +
      `"${reducerKeys.join('", "')}". Unexpected keys will be ignored.`
    )
  }
}
```

判断是否为「纯对象」，并且检查是否有额外的属性。输出对应的错误信息。实现细节参考注释。

最后，把 state 交给每一个 `reducer` ，获取到新的 `nextState` 。并且在过程中检查 `nextStateForKey` 不要为 `undefined` 。返回最终的 `state` 。代码比较简单，阅读难度不大。

对应的源码[地址](https://github.com/Wzb3422/redux/blob/master/src/combineReducers.ts#L191-L209)

```ts
    let hasChanged = false // 是否改变的标志位
    const nextState: StateFromReducersMapObject<typeof reducers> = {}
    // 遍历所有的 reducers
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action) // 从 reducer 计算得到 nextState
      // undefined 处理
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      // 看 state 是否改变，同时使用短路优化
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged = // 值没有变，但是非纯对象部分被去除的情况
      hasChanged || finalReducerKeys.length !== Object.keys(state).length
    return hasChanged ? nextState : state
  }
}
```

## Reference

+ [combineReducers | Redux](https://redux.js.org/api/combinereducers)

+ [redux - GitHub](https://github.com/Wzb3422/redux)

