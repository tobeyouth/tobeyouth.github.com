---
layout: post
title: "redux, redux-saga and socket.io"
modified: 2018-09-21 15:16:36 +0800
tags: [redux, redux-saga, socket.io]
category: blog
---

最近公司举行 hackthon, 做了个棋牌类的游戏, 前端的主要技术栈就是 `react` + `redux` + `redux-saga` + `socket.io`


`react` 没什么可说的了，基本已经是前端标配了。本来 `redux` 作为 `react` 全家桶的一部分，也没什么可讨论的了。

但是`redux-saga`作为 `redux` 的神奇 `middleware`, 如果不熟悉 `redux` 的话, 很难真正理解 `redux-saga`, 所以顺路再梳理一下 `redux` 的使用方法和哲学。


## redux

上来先是一个官方文档[redux](https://redux.js.org/)

那么 `redux` 到底是什么呢？

先来看官方的一句话简介：

> Redux is a predictable state container for JavaScript apps.

简单明了的说呢, `redux` 就是一个管理 `state` 的东西。

那么 `state` 又是什么呢？

`state` 我们可以理解为`状态`, 那 `redux` 是个状态机吗？

答案是, **可以这么理解**。

除了 `state`,简介里还提到了一个至关重要的词 - `predictable`。

那么什么才算是 `predictable` 呢？

在 `redux` 的实现里, `predictable` 被实现成了 `functional`,  也就是俗称的 `函数式`。函数式编程这里就不多说了, 有兴趣的可以上网搜一搜, 相关的介绍很多。

那么 `redux` 又是如何做到 `predictable` 的呢？简单来说, 就是把所有可能更新 `state` 的操作都放到了 `reducer` 中, 通过 `action` 来驱动 `reducer` 做出不同的操作。并且 `action` 和 `reducer` 都是纯函数, 从而达到 `predictable`目的。

#### three principles

使用 `redux` 只需记住这三条原则, 就基本上可以使你的 app 达到一个 `predictable` 的状态。

1. single source of truth
	
	一部分业务, 只维护一个数据源(也可以理解为,只是用一个 `state`)。
	
	但并不是整个应用都要使用一个 `state`, 而是同一部分业务, 只使用一个数据源。
	

2. state is read-only
  
  永远不要尝试直接修改你的 `state`, 如果要修改 `state`, 请触发 `action`, 让 `reducer` 去修改.


3. changes are made with pure functions

   只使用**纯函数**来更新 `state`。
   
   为什么要使用纯函数？
   
   因为纯函数是完全可预测的。

#### flow & api

`redux` 本身是个很简单的库, 最可贵的是提出了上面的三条原则, 并且在代码层面提供了执行这三条原则的结构和 api。

整个`redux`的执行流程, 可以用伪代码的形式来表达一下

```
creatStore -->
user interface ---> 
store.dispatch(action<Function>) --> 
reducer<Function> to change the state and return new state -->
rerender view
```

接下来再看下在 `redux` 中, 是如何通过几个 api 来实现这个过程的：

- createStore(reducer, preloadedState, enhancer)
 
  源码[在此](https://github.com/reduxjs/redux/blob/master/src/createStore.js)，减去注释, 大概也就 100 多行代码。
  
  顾名思义, 就是创建了个 `store`, 通过看源码, 我们可以知道这个 `store` 有以下几个方法：
  
  - dispatch
  - subscribe
  - getState
  - replaceReducer
  - [$$observable]

- combineReducers(reudcers)

  在真实的开发环境中, 通常我们开发一个完整的应用, 在一些复杂的页面中, 不可能仅仅使用一个 `state` 就能完整的表达整个应用的状态, 这时多个 `state` 可能面对的就是多个不同的 `reducer`, 但是 `createStore` 这个 api 只能接收一个 `reducer`, 要怎么办呢？
  
  这个时候, 我们就需要用到`combineReducers`了, 它返回一个 `combination` 方法, 在这个方法中, 会将传入的 `reducers` 逐个执行, 返回的是根据 `reducer` 的 name, 处理后的不同的 `state`。
  
- bindActionCreators(actionCreators, dispatch):
	
  通常一个应用中, 会触发很多 `action`, 这些 `action` 触发函数的集合, 就叫做 `actionCreators`。

  另外, 在 `createStore` 中, 我们返回的 `store` 是带有 `dispatch` 方法的, 这个`dispatch` 方法就是 `bindActionCreators`的第二个参数。

  这个 api 的主要作用就是连接了 `store.dispatch` 和定义的 `actionCreators`, 使触发的 `action` 可以在 `redux` 的 `container` 中执行。

- compose(...funcs)

  `compose` 返回一个高阶函数, 可以将 functions 从左到右按 `reduce` 的方式执行
  
  代码非常简单, 看这里:
  
  ```
  export default function compose(...funcs) {
    if (funcs.length === 0) {
      return arg => arg
    }

    if (funcs.length === 1) {
      return funcs[0]
    }

    return funcs.reduce((a, b) => (...args) => a(b(...args)))
  }
  ```

- applyMiddleware(...middlewares)

  在 `redux` 的执行流程中, 可能会有一些多个 `reducer` 通用的操作, 这个操作, 可以在 `redux` 的 flow 中, 执行 `reducer` 前执行 `middleware`。`middleware` 将以 `reduce`的方式逐个执行, 在 `middleware` 中可以获取 `state` 和 `dispath` 其他的 `actionCreator`。


通过这几个 api, 即可实现上面所描述的三条原则, 具体怎么实现, 我们将以接下来的 `react-redux` 为例, 展开说说。


## react-redux

之所以把 `react-redux` 放上来, 主要是为了看下 `redux` 的几个 api 在真实的开发环境中, 是如何使用的。

还是先来看看 `react-redux` 的 api 吧：

- Provider

  `Provider`实际上是一个 `React Component`, 它接收一个 `props`, 就是 `redux` 的 `createStore` 这个方法返回的 `store`

- createProvider
- connectAdvanced
- connect




## redux-saga

## socket.io

## mix them all