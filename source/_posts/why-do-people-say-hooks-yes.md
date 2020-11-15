---
title: Why do people say "hooks yes"?
date: 2020-11-13 19:20:28
tags: React
keywords:
  - React
  - Hooks
  - JavaScript
thumbnail: https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/diveintofreacahjoks.png
categories: React
---

Understanding the origin of hooks.

<!-- MORE -->

## Why introducing Hooks?

The emergence of new tech is generally to solve the problems and defects of the original technology. Let's go over how we reuse stateful logic and problems we often suffer before hooks were born.

### Problems we suffered

#### 1. Hard to reuse stateful logic between components

 `render props`and `HOC` requires you to restructure your components hierachy to fit the data flow. And continuous use of `HOC` results in "wrapper hell" of components, which can be seen in React Devtools. While we could filter them out in Devtools, we need a more primitive way for sharing stateful logic.

#### 2. Complex components become hard to understand

We often maintain a components started out simple but grew into a mess of stateful logic and side effects. Each lifecycle method contains a mix of logics. For example, we often do data fetching in `componentDidMount` and  `componentDidUpdate`. And the same `componentDidUpdate` methods contains event listener set up and other unrelated logic. We finally clean up listeners in `componentWillUnmount`. Mutually related code get split apart. When a component grew into a huge one, it becomes so cumbersome, making it easy to introduce bugs.

## Hooks Yes ! Why ?

Hooks provide a primitive way to manage stateful logic, without changing hierachy of components.

Let's start with `useState`. A possible example like this:

```js
const [age, setAge] = useState(18);
```

A tuple is returned, which contains state itself and the updater function. Mutually related state and logic are returned in pair. Hooks encapsulate state and logic. They're no longer splitted in different lifecycle methods. We can build our own hooks to extract component logic into reusable funtions.(We'll further disscuss this later.

Hooks are closer to the React underlying mechanism. React is about *schedule - render - commit*. Lifecycle methods in class components are upper abstraction for the mechanism, making it easy for developer to opt in. However, hooks are more closer to the mechanism.

Hooks splits your state into primitives. When writing class components, we store all state in `this.state`. State have to be stored in one object, even though most of them are not related.


## Reference

+ [React 技术揭秘 - 卡颂](https://react.iamkasong.com/)
+ [Introducing Hooks - React](https://reactjs.org/docs/hooks-intro.html)
+ [对 React Hooks 的一些思考 - 张立理](https://zhuanlan.zhihu.com/p/48264713)
+ https://www.youtube.com/watch?v=dpw9EHDh2bM&feature=youtu.be

