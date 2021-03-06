---
layout: post
title: "start-at-bitcoin"
modified: 2018-08-22 13:22:46 +0800
tags: [bitcoin]
category: blog
---


### 诞生

比特币诞生于 2008 年，一个网名为**中本聪**的人，提出了一个设想：

> 创造一种不受政府或任何组织控制的货币

比特币的本质就是一串数字，没有任何资产支持（现行货币背后都是国家或银行提供资产支持）。也就是说，比特币是一个完全出于**社区共识**的货币。


### 货币是什么

wikipedia 上对货币的定义为：

> 货币，称钱财，是人们为提高交易效益，对一种媒介达成的共识。

我们现在正在使用的纸币，其实可以看成是**共识**的实体代表。如果没有纸币，货币仍然是成立的，比如银行账户里的钱。

基于这个定义，任何一个事物，不管是实体的还是虚拟的，只要在大家的共识中，它是有价值的，那么就可以当做货币来使用，比特币正是这样一种事物。

而且相较于银行账户里的货币，比特币的数据全部存放在区块链上，更不容易被破坏。

### 比特币的运行

货币的运行主要是通过交易，也就是货币从 A 到 B 这样一个过程。比特币也不例外，不过比特币的交易并不是手递手的传递，而是通过从 A 的比特币地址到 B 的比特币地址来完成的。

对于比特币来说，并不存在**人**这个概念，比特币都是从**地址**到**地址**的。另外，我们上面说过，比特币的所有数据都是存在区块链上的，所有数据都是公开的，所以想要查到某个地址有多少比特币，是非常容易的。

看起来好像是所有的信息都被暴露了，但是有一点非常关键的是，没有人知道地址背后的人到底是谁，这为比特币提供了非常好的匿名性。

#### 公钥/私钥

上面一直在强调**地址**这个东西，比特币地址的基础是`非对称加密算法`。

那么什么又是非对称加密算法呢？

举个简单的例子，在谍战剧中，我们常看到有特工截获敌人的电报信号，但是出于安全性考虑，这些电报的内容其实都是加密过的。通常情况下是有个密码本，发送方和接收方各执一份，这就叫做`对称加密`。

内容先加密再发送，接收方收到消息后，先解密再阅读。

但是这样的方式有个严重的缺点，就是加密规则（密码本）一旦泄露或者被破译，那么发送的内容就跟不加密没有区别了。

为了解决这个问题，1970年，有两个美国科学家提出了一种被称作`Diffie–Hellman key exchange`的算法。

![Diffie–Hellman](https://image.ibb.co/b4GKXe/Diffie_Hellman.png)

使加密和解密可以使用不同的规则，从而也就避免了密码本失窃之后导致的安全问题，这种加密方式，就被称为`非对称加密`。

在非对称加密中，基本就是执行了这样几个步骤：

1. A 生成了两把密钥（公钥/私钥），A 把公钥发送给 B
2. B 获得了 A 的公钥，同时用公钥对信息进行加密，并把加密后的信息发送给 A
3. A 收到了加密后的信息，然后用私钥解密，获得真正的内容

在这个过程中，如果 A 的私钥不被泄露的话，那么用 A 的公钥加密的信息，只有 A 才能够解开，从而保证了安全性。


#### 比特币钱包/地址

说了半天加密，那么加密是为了什么呢？答案就是为了钱包/地址。

比特币钱包其实就是一个存储公钥/私钥的地方。

通常，会生成一个 256 位的私钥；和一个 512 位的公钥（与其他加密系统不同的是，比特币钱包中的公钥，直到第一次交易时，才会被公开）。

因为 512 位的公钥实在是太长了，所以会再生成一个比特币钱包的地址，用来做交易。首先会通过 `SHA-256`和 `RIPEMD` 算法将公钥变成一个 160 位的 hash 值，这个 hash 值也被称为指纹；但是 160 位还是太长，接下来会通过一个 `Base58Check` 的方法，将这个 160 位的 hash 编码成一个长度为 26 ~ 35 的 base58 字符串，这个字符串就是比特币钱包的地址。

接着，会根据私钥，再通过 `Base58Check`方法，生成一个较短的 base58 格式的字符串，这个字符串被称作 `Wallet Interchange Format(WIF)`。

到最后，我们一共得到了三种不同的 key，分别是

1. 两个密钥
	- 256 位的私钥
	- 512 位的公钥
2. 一个 160 位的公钥的 hash
3. 较短的 base58 格式的两个字符串
	- 钱包地址
	- 钱包交换格式(WIF)

流程和关系可以看下面这个图，会更清晰一些：

![bitcoin keys](http://static.righto.com/images/bitcoin/bitcoinkeys.png)



#### 交易

比特币的交易流程整体看起来非常简单，一共分为以下几个步骤

1. A 向 B 发起转账
2. 根据比特币协议, 验证这笔交易是否可行
3. 如果可行, 由矿工将交易信息打包, 写到区块链上
4. 写入成功, 即代表交易成功

但是跟通常情况下，银行账号间的转账交易不同，比特币的交易本身不存在**账户**这样一个概念，A 向 B 转账，比特币并不真正落入 B 的某个账户中，而只是标记了 B **拥有将这部分比特币转出的权利**。

下面这个图记录了 3 个比特币交易的数据：

![transactions](http://static.righto.com/images/bitcoin/transaction_diagram-s800.png)

可以看到的 `Transaction A` 和 `Transaction B` 分别转了 0.005btc 和 0.003btc 到了 `Transaction C`, 之后 `Transaction C` 又分别转出了 0.003btc 和 0.004btc。剩下的 0.001 btc，就作为矿工的费用，归矿工所有了。

在比特币交易的过程中，因为记录的是交易本身，所以有一个原则就是每笔交易的 `input` 总和 一定要等于 `output`总和。

假设一笔交易收到了 100btc，但是真正想转出去的只有 1btc，但是根据上面的原则，这笔交易的 `output` 总和一定要是 100btc，解决办法就是将剩下的 99btc 再做一个 `output` 转回到自己的钱包地址。

我们随便找个比特币[交易记录](https://www.blockchain.com/zh-cn/btc/tx/c18e5cdb816262d2ebc47c35c77bdb32a6324f4ac507dff310159aa448054a62)看下

![看这里](https://image.ibb.co/eNhMf9/image.png)

输入和输入是一致的。

#### 找零
###
交易过程中，会存在**找零**的问题，比如上面我们讲到：

> 假设一笔交易收到了 100btc，但是真正想转出去的只有 1btc，但是根据上面的原则，这笔交易的 `output` 总和一定要是 100btc，解决办法就是将剩下的 99btc 再做一个 `output` 转回到自己的钱包地址。

这个动作就称为找零。

在真实的比特币世界中，通常情况下，找零是会返回到一个新的地址上，而不是原地址。这么做的主要原因有两个：

1. 首先，如果一笔交易存在找零，那么在查看交易记录时，并不会标记出哪个是目标地址，哪个是找零地址，所以也就是说难以猜到交易的真正对象。
2. 其次，因为在交易过程中，转出方公布了自己的公钥，虽然以现在的计算机的算力很难通过公钥反推出私钥，但是并不代表未来也不能反推。所以每次找零都返回到一个新的地址，相当于把剩下的资产做了转移，从而隐藏了公钥。


#### UTXO

所谓 UTXO 全称为 `Unspent Transaction Output`，它是比特币交易的基本单位。

> UTXO 是不能再分割、被所有者锁住的或记录与区块链中的并被整个网络识别成或单位的一定量的比特币。([原文见这里](https://bitcoin.stackexchange.com/questions/64504/what-does-unable-to-decode-output-address-mean))


比较通俗的讲，一次交易中的 `output`，其实就是一个 UTXO，假设我手里有一张 10 块钱，一张 5 块钱，但是我想买一个 12 块钱的东西；那么我一次性要 `output` 两个人民币，分别是 10 块和 5 块，然后又找回给我 3 块钱，那么这个 3 块钱，也是 `UTXO`。


#### 交易过程中的验证

因为是完全去中心且匿名的交易，所以在交易的过程中，会需要一些验证，来证明该交易是否是符合规则的。

每一次的交易，都会要求转出方提供以下信息，以验证当前的交易是否合法：

- 上一笔交易的 hash
- 转出方的公钥
- 转出方私钥生成的数字签名

为了验证交易是否可以正常进行，需要以下三个步骤

1. 找到上一笔交易，以确认支付方确实有能力支付本次交易
2. 算出支付方公钥的指纹，确保与支付方地址一致
3. 使用公钥解开数字签名，以验证私钥属实

具体的步骤，可以参加这个图：

![bitcoin_transaction_chain](http://static.righto.com/images/bitcoin/bitcoin_transaction_chain-s512.png)

经过验证之后，当前的交易将会被矿工打包，并写到区块链中。具体的写入过程可以参考[这篇文章](https://tobeyouth.github.io/blog/2018/08/21/start-at-blockchain/)中的介绍。根据比特币的协议，一个区块的大小是 `1MB`, 而一笔交易大概是 `500b`，因此一个区块大概可以包含 2000 笔交易。矿工负责将这些交易打包，算出 hash，再写入到链上。

因为打包写入到区块链上，是一个会消耗大量计算的过程，所以矿工的工作并不是义务劳动。根据比特币的协议规定，成功添加新区块的矿工将获得奖励，一开始是 50btc，然后每 4 年减半；另一方面，因为比特币最多只支持到小数点后面 8 位，所以可以得出到了 2140 年，矿工将得不到任何奖励。到时候，矿工所能获得到的收益就只能是交易手续费了。

所谓的交易手续费，是由交易方付给矿工的，具体的金额随意。但是矿工会优先选择手续费较高的交易进行打包，以预期获得更高的收益。所以也就是付给矿工的手续费越高，那么交易也就会被越早确认。

#### 交易中产生的冲突

因为是非同步的，而且节点间的同步有可能会出现延迟。所以，就有可能存在同一笔交易被写到了两个区块中的情况，为了解决这种情况所造成的冲突。

比特币的链上规则中做了规定，因为区块是前后链接的，所以一旦出现分叉，那么哪个区块后面跟着的区块多，就判定哪个分叉上的链是正确的，另一条链将被抛弃，其中打包的交易也将因为没有被写入到主链上而判定为失败。

#### 交易和账户

上面说了半天，其实一直在说的都是交易。但是在主流的支付体系中，都是有**账户**这一概念的，也就说，你收入多少，支出多少，构成了你账户中余额的多少。

但是在比特币中，并不存在**账户**这一概念，区块链上记录的就是交易，如果想知道一个地址上一共有多少比特币，那么需要遍历所有的交易，才能计算得出。

### Reference

- [比特币入门教程](http://www.ruanyifeng.com/blog/2018/01/bitcoin-tutorial.html)
- [RSA算法原理](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)
- [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)
- [菲迪-赫尔曼密钥交换算法](https://zh.wikipedia.org/wiki/%E8%BF%AA%E8%8F%B2-%E8%B5%AB%E7%88%BE%E6%9B%BC%E5%AF%86%E9%91%B0%E4%BA%A4%E6%8F%9B)
- [RIPEMD](https://zh.wikipedia.org/wiki/RIPEMD)
- [Base58](https://zh.wikipedia.org/wiki/Base58)
- [bitcoin-paper](https://bitcoin.org/en/bitcoin-paper)
- [Protocol_documentation](Protocol_documentation)
- [how-the-bitcoin-protocol-actually-works](http://www.michaelnielsen.org/ddi/how-the-bitcoin-protocol-actually-works/)
- [比特币交易查询](https://www.blockchain.com/zh-cn/explorer)