---
layout: post
title: "start at blockchain"
modified: 2018-08-21 12:56:02 +0800
tags: [blockchain]
category: blog
---

最近了解了一些区块链的东西，动了一些深入了解的心思。

了解的过程中会不断补充和完善这篇文章，理解的不一定对，随意看看得了，每篇文章都会把阅读过的资料列出来，也方便大家去源头看看。


### 区块链（blockchain）

区块链是什么？

> 一句话，它是一种特殊的分布式数据库

这句定义中有两个关键词**分布式**和**数据库**.

首先，我们可以把区块链看成是数据库，但是这个数据库并没有管理员，也没有中心，每个应用链的人，都具有平等的权利。

### 区块

区块链是由一个个的**区块(block)**组成的, 每个区块可以看做是一个存储数据的单元，每写入一次数据，就会新建一个区块，一个一个区块连接起来，就成了链状。

每个区块分成两个部分：

- 区块头(Block Head)
  
  包含了当前区块的基本信息，包括 `生成时间`, `区块体的 hash 值`, `上一个区块的 hash` 等信息。
  
  区块的 hash 就是根据区块头的内容来生成的。
  
- 区块体(Block Body)

	区块体就是所含数据的真实内容
		
#### hash

区块的根本是源于 hash, 如果把区块链想象成一个真实的链状结构的话，那么 hash 就是每个元素之间的连接点。

##### hash 的特征

关于 hash 的详尽定义，可以在 [wikipedia](https://zh.wikipedia.org/wiki/%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B8)中查看，这里简单总结一下 hash 的特征：

1. 如果两个 hash 是不相同的（根据同一函数），那么这两个 hash 的原始输入也是不相同的
2. hash 的输入和输出不是唯一对应关系的，如果两个 hash 值相同，两个输入值很可能是相同的，但也可能不同，这是由具体的 hash 计算方法决定的

##### hash 在区块链中的应用

区块的内容与 hash 是一一对应的，每个区块的 hash 值都是不同的。同时，每一个区块又都保存着上一个区块的 hash 值。

按照这个逻辑，如果一个区块的 hash 值变了的话，那么它之后的每一个区块的 hash 值都需要进行更新。

进而延伸一下，如果有人想更新某一区块上的内容，那么他必须有能力依次更新这个区块后面所有区块的 hash 值才行，否则的话，被更新的区块就会脱离整个链了。由于更新区块 hash 的计算很复杂且耗时，所以除非掌握了全网 50% 以上的运算能力，否则更新某一区块的行为根本不可能成功。

正是通过这种环环相扣的机制，区块链才可以保证自身数据的不可修改的特性。

### 区块链是如何运行的

#### 挖矿

由于区块链是一条单一的链，每个区块都依赖于上一个区块，所以同一时间，只能向链上添加一个区块。

又由于必须保证各个节点间的同步，所以添加完一个区块，各个节点又必须进行同步。

也就是说，假设节点 A 正忙着生成新的区块，突然收到消息:“B 已经在链上添加了一个区块！”。这个时候 A 之前做的计算就都白做了，又得基于 B  新添加的节点，重新刚才的计算。并且计算的过程中，还会有类似的事情发生。

上面我们说过，在区块链的运行机制中，通过区块自身的 hash 来判断当前区块是否可以被加入到链上。

同时，为了使区块的添加速率趋于平均（比特币的设计是大约10分钟生成一个新的区块），并不是随便一个新建的区块都可以被添加到链上。

通常会设置一个准入条件，只有符合该条件的区块，才可以被添加到链上，设计者通过设置海量的计算，来使计算满足加入条件的区块 hash 值变得非常困难，这个过程就叫做挖矿，也叫（mining）, 继续挖矿计算的机器, 我们就叫做矿机, 操作机器的人(其实是程序), 我们就叫做矿工。


#### 用 js 代码运行一下简单的挖矿机制

首先，每一个区块，我们可以看成是由这几部分组成：

![看这里](https://cdn-images-1.medium.com/max/1600/1*Y3c_hIqCuiDH4x-8dObVyg.png)


这是我们这个链上的第一个区块，这个区块包括了这样几个信息

- index: 区块的 index 值, 这里应该是 0
- previous hash: 前一个区块的 hash
- timestamp: 区块创建时的时间戳
- data: 区块包含的数据内容
- hash: 区块本身的 hash
- nonce: 计算区块 hash 的难度系数


我们先假定这个区块的名字是 `A`, 接下来,我们在它后面再添加个区块 `B`.

那么我的这个 `B` 将包含以下这些信息：

- index: 1
- previous hash: 也就是 `A` 的 hash
- timestamp
- data: I love Blockchain
- hash: ?
- nonce: ?

如果要将 `B` 添加到链上，那么我们接下来的工作就是计算出 `hash` 和 `nonce` 的值，

那么就假设我们计算 hash 的算法是这样的：

```
CryptoJS.SHA256(index + previousHash + timestamp + data + nonce)
```

接下来我们来计算这个 hash 值是否是一个准入的 hash, 代码如下：

```
function isValidHashDifficulty(hash, difficulty) {
  for (var i = 0, b = hash.length; i < b; i ++) {
      if (hash[i] !== '0') {
          break;
      }
  }
  return i >= difficulty;
}
```
我们这里的准入条件为 `hash` 中连续不为 `0` 的值必须不小于 `difficulty`.

参数中的 `difficulty`, 也可以称为`难度系数`, 通过这个系数的动态调整，可以维持新增区块的平均速率。

另外，我们上面计算 hash 的算法中 `index`, `previousHash`, `timestamp` 和 `data` 都是固定的，所以重复计算出来的 hash 值也应该是一样的，为了使每次计算出的 hash 不同, `nonce`被设计成了一个变化的值, 通过 `nonce`值的变化, 计算出不同的 hash, 从而判断  hash 值是否符合准入条件, 用代码表示的话. 就是这样：

```
let nonce = 0;
let hash;
let input;
while(!isValidHashDifficulty(hash, difficulty)) {     
  nonce = nonce + 1;
  input = index + previousHash + timestamp + data + nonce;
  hash = CryptoJS.SHA256(input)
}
```

看到这里, 我们可以发现, 其实矿工的工作就是去猜`nonce`的值, 因为这个值很大, 所以计算难道也就比较大, 挖矿也就成了一件费时费力的事情。

相应的, 既然挖矿是如此费时费力的一件事, 那么矿工也不会是义务劳动的, 矿工获得的相应收益, 会在下一部分 **比特币** 中, 以比特币的运行规则来描述一下。



### Reference
- [区块链入门教程](http://www.ruanyifeng.com/blog/2017/12/blockchain-tutorial.html)
- [How does blockchain really work? I built an app to show you](https://medium.freecodecamp.org/how-does-blockchain-really-work-i-built-an-app-to-show-you-6b70cd4caf7d)
- [Ethereum for web developers](https://medium.com/@mvmurthy/ethereum-for-web-developers-890be23d1d0c)