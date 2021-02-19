---
layout: post
title: "functional programming"
modified: 2017-07-07 10:59:08 +0800
tags: [functional programming]
---

简单的梳理一下函数式编程的相关知识点。

## 定义
首先来看什么才是**函数式编程(functional programming)**。

来自 [wikipedia](https://en.wikipedia.org/wiki/Functional_programming) 的定义

> In computer science, functional programming is a programming paradigm—a style of building the structure and elements of computer programs—that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data...

来自[中文 wikipedia](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80)
>  函数式编程（英语：functional programming）或称函数程序设计，又称泛函编程，是一种编程范型，它将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。函数编程语言最重要的基础是λ演算（lambda calculus）。而且λ演算的函数可以接受函数当作输入（引数）和输出（传出值）


## 特点
根据定义，我们可以得出函数式编程的几个特点：

- First-class and higher-order functions: 函数可以被当做参数，也可以被当做返回
- Pure functions: [纯函数](https://zh.wikipedia.org/wiki/%E7%BA%AF%E5%87%BD%E6%95%B0)，固定的输入一定会有固定的输出；同时不会有[副作用](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E5%89%AF%E4%BD%9C%E7%94%A8)
- Recursion: 通常来讲是尾递归优化，因为如果函数嵌套过多的话，会导致占用内存过高，所以需要进行尾递归优化
- Strict versus non-strict evaluation: 通常被理解为**惰性计算**，也就是说需要用到时，才会通过表达式取值，而不是在一开始就把值都取到
- Referential transparency: 引用透明，在函数式编程中，一个变量一旦被定义，将不会再改变，这也是保证无副作用的前提。对于数值的改变，通常情况下是直接返回一个新的值，而不去改变原有的值。

## 一些好处

函数式编程的很多优势，都是围绕着`纯函数`这个概念的。

因为是`纯函数`，所以输入不变，那么输出肯定是不变的；因为这种前后的一致性，就为测试带来了很大的便利。

另外，因为是`纯函数`，其实也就并不存在`全局变量`的这个概念，所有的数据就是在类似`pipe`的管道中流动，每个函数的输出都是一个全新的数据，并不会修改传入的数据；因为无副作用，所以可以说函数式编程是"安全"的。

## 一些概念

### Currying 

这里的 `Currying` 并不是咖喱，而是传说中的[柯里化](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96)。
简单来说，柯里化就是将多参数的函数变成单参数的函数，例如：

```
// origin:
function add (x, y) {
	return x + y
}
var a = add(1, 2)

// after currying:
function add(x) {
	return function (y) {
		return x + y
	}
}

var b = add(1)(2)
```
这样看起来柯里化似乎没有什么优势，但是再看一下接下来的代码：

```
var a = add(10, 1)
var b = add(10, 2)
var c = add(10, 3)
```
可以看到这三次调用，都是在`10`的基础上再加上一个参数，那么运用柯里化，可以改造成这样：

```
function addNumber(x) {
	return function (y) {
		return x + y
	}
}

var addTen = addNumber(10)

var a = addTen(1)
var b = addTen(2)
var c = addTen(3)
```

每次只传入一个参数，看起来清爽，调用起来也更自由。这里只是个最简单的应用，在更多参数的情景中，柯里化的作用会更加明显。

### Monad
Monad 可以理解为是函数式编程的单元，一个 Monad 


// 未完待续