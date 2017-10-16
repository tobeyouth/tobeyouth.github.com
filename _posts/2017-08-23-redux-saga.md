---
layout: post
title: "redux-saga"
modified: 2017-08-23 16:00:12 +0800
category: blog
tags: [redux, redux-saga, javascript]
---

最近组里的同事在尝试使用 [redux-saga](https://redux-saga.js.org) 改造一些异步请求比较多的功能。
对 `redux-saga` 早有耳闻，但是出于懒，一直没去关注，这次就着帮同事做 code review 看了一下 `redux-saga`
的文档和代码，以下是学习过程中的一些心得和体会。

按照以往学习的思路，一个被众人推崇的东西，一定有这几方面是值得学习和思考的：

- 设计哲学
- 解决的问题
- 使用后的效果

以下的体会也将从这三个方面来总结一下

### 设计哲学

在使用 `redux-saga` 之前，在 `redux` 中处理异步请求，通常是使用 [redux-thunk](https://github.com/gaearon/redux-thunk)
来解决的。

这里多扯一些关于 `redux`, `redux-thunk` 和 `redux-saga` 之间的关系和差异

#### redux
先来说 `redux`，`redux` 本身是一个完整的控制数据流的框架。在 `redux` 中我们通常关心的是 `action`, `reducer` ,
`middleware`以及
`provider` 中的元素如何响应数据的变化。`redux` 精简了整个流程，使 `action` 和 `reducer` 都只需要 `return`
即可，这样每个部分看起来都可以当做是纯函数，相应的，测试起来也很方便，毕竟是纯函数，固定的输入对应固定的输出。


#### redux-thunk
但是 `redux` 本身的简单，单纯的 `return` 并不能处理异步需求。所以才会使用 `redux-thunk` 这样的插件。

但是需要注意的是，`redux-thunk` 并不是只为了完成异步需求才会被创造出来的。要理解`react-thunk`，首先要了解一下 `thunk`
这个概念

> A thunk is a function that wraps an expression to delay its evaluation.

说白了的话，`thunk` 是一个被延迟执行的函数。

那么`redux-thunk`是怎么解决异步的问题呢？可以直接看下`redux-thunk` 的源码，非常非常非常短

```
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

之所以 `redux-thunk` 可以解决异步的问题，是因为将异步的部分包装成了一个 function，并传入了
`store.dispatch`，这样可以在异步完成之后再调用`dispatch` 触发 `reducer` 的中的响应。

所以`redux-thunk` 的本质是将 `action` 中返回的函数执行。

#### redux-saga

相比于`redux-thunk`，`redux-saga` 更像是在`redux` 的`middlewaras`之外，通过监听 `action` 提供了一个执行`task`
的地方。

另外，`redux-saga` 比较突出的特点是使用了 `generator` 特性，使异步的返回可以用更流畅的语法来表示。


### 解决的问题

在一个多交互的页面中，通常会有多个动作触发同一个 `action` 的时候。比如在购物车中，**加/减数量**, **选择/取消选择商品**,
**删除商品**都会触发重新计算总价的请求。使用 `redux-thunk` 的做法是写一个 `middleware` ，监听所有经过的
`action`，如果发现 `action` 是会触发计算总价的请求中的几种，就会去计算总价，等回调完成后再更新总价。

这个时候就容易出现多个计算请求的情况，因为是异步请求，所以其返回顺序不一定，所以更新总价时会造成错误。

如果用 `radux-saga` 来解决这个问题的话，可以使用`takeLastest` 来执行最后一次被触发的 `task`，之前被触发的 `task`
都会被取消，从而也就解决了这个问题。

当然，这只是一个简单的例子；其实通过其他方法也可以实现在 `redux-thunk` 中实现取消未完成的异步请求的功能。


### 使用后的效果

使用 `redux-saga` 的效果可以从以下几个方法来看：

- 写法上抛弃了 `middlewares` 的概念，因为 `redux-saga`本身就是一个`middleware`，这个 `middleware` 监听了
  `action`，从而可以根据不同的 `action` 处理 `redux` 中的副作用问题。可以简单的理解为将 `middleware` 全都迁移到了`sagas`
  目录中。

- 使用了 `redux-saga` 后，`action` 中的方法可以变得更**纯粹**，不需要再在 `action` 中处理异步问题，每一个 `action`
  只是单纯的返回对应的 `type` 和相关数据即可。
 
- 因为`action` 变得**纯粹**，所以为 `action` 编写测试也就更加方便

- 异步从 `action`中专业到了 `saga` 中，相应单元测试的编写也就从 `action` 转移到了 `saga`。但比较方便的是，`redux-saga`
   使用了 `generator` ，在编写测试用例时，可以基本有同步写法的感觉。


### 一些方法和概念

简单的说完了 `redux-saga` 的一些简介，这里着重列举一下 `redux-saga` 中的方法和概念。


#### Effects

`Effect` 是 `redux-saga` 中的一个简单的对象定义

`Effects` 是 `redux-saga` 中对 `yield` 返回对象的包装。因为`redux-saga` 中每一个 `saga` 都是 `generator` 函数，所以
每个`yield`返回都可以称作是一个 `Effect`。

`redux-saga` 官方推荐使用 `redux-saga/effects` 中的方法来创建一个 `Effect`
对象用于返回。但并不是一定要使用其中的方法才能创建 `Effect`，正如上面所说，每一个`yield` 返回都可以称之为一个 `Effect`。

##### promise, call, apply


最简单的方式是直接 `yield` 一个 `Promise` 对象，也可以通过 `call` 和 `apply` 等方式创建 `Effect`。

但单纯的 `yield` 一个 `Promise` 对象和通过 `redux-saga/effects` 返回一个 `Effect` 对象是不同的。使用 `redux-saga/effects` 提供的方法返回的，将是一个**纯粹**的 javascript 对象，用于描述将要执行的操作，而这个操作，将不会立即执行，而是会到 `redux-saga` 这个 `middleware` 中再去执行。

下面我们通过两个例子来说明这两种方式的异同：

```
// yield a promise

function *getData(url) {
  const data = yield Api.Fetch(url)
  // ...
} 


// yield a effect
import { call } from 'redux-saga/effects'

function *getData(url) {
  const data = yield call(Api.Fetch, url)
  // ...
}
```

在针对返回 `promise` 对象的这个方法写测试时，我们可以这样写：

```
// unit test for yield promise

const iterator = getData(url)
assert.deepEqual(iterator.next().value, another_promise) 
```

因为 `yield` 返回的是个 `promise` 对象，所以在这里我们如果想要测试，也只能创造另一个 `promise` 对象。
但我们这个单元测试的真正目的是为了测试**调用 getData 这个方法，yield 后的方法会不会被正确的执行**，这样创造出另一个 `promise`
对象方法难免有些过于**沉重**。

再来看下，使用 `call` 方法返回了 `Effect` 对象将要怎么测试:

```
// unit text for yield effect

const iterator = getData(url)
assert.deepEqual(iterator.next().value, call(Api.Fetch, url))

```
我们只需要再调用一次 `Api.Fetch` 方法，并不需要关心`call` 调用的是什么方法，写起测试来，感觉可以更**无脑**一些。

##### dispatch, put

在 `saga` 的实际编写中，经常会遇到某个异步请求结束后，需要 `dispatch` 一个新的事件，以让 `store` 和 `reducer` 做出相应
。如果直接通过 `dispath` 方法传递事件和数据的话，单元测试将会无法覆盖到 `dispath` 的动作。

所以，`redux-saga` 提供了`put` 方法，这个方法类似 `call`，将会创建一个标准的可描述对象，将 `dispatch` 的操作放在 `redux-saga` 的 `middleware` 中执行。在 `saga`方法中可以 `yield`
这个对象，则可以在单元测试中覆盖这个操作。


##### takeEvery, takeLast, take

使用 `takeEvery`，可以监听每一次 `action` 的调用，并返回相应的 `Effect`并执行；与之不同的是
`takeLast`，将只执行最新的一次`action` 的调用。

所以，使用 `takeLast` 可以准确的获取多次异步请求中最新的一次，解决上面所说的购物车问题。


