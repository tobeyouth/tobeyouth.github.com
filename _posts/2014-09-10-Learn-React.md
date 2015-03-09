---
layout: post
title: "Learn React"
modified: 2014-09-10 17:46:00 +0800
tags: ['javascript','react']
---

`React`是*facebook*开源的一个组件化框架。与众不同采用了`jsx`方式作为组件的组成方式。

所谓`jsx`是一种类似*xml*的文件格式，可以理解成为*带有js的xml格式*。所有的`React`组件都是基于`jsx`这种格式进行编写。至于这种格式究竟是好是坏，也是因人而异吧。

个人认为，出于`jsx`这种格式，所以限定了每个组件的粒度需要很细，因为如果写太多的话，跟html相比，还是有点别扭啊o(╯□╰)o。当然，如果想继续用html格式的话，`angular`其实也是个很不错的选择。

再说回`React`，`React`的主旨就是*一切皆组件*，与之思想相同的还有`angular`和`polymer`等，甚至你也可以不用任何框架，通过自己的一套规范是实现这个思路。落实到实现层面，`React`还提供了令一套机制，就是*数据循环*，所谓*循环*，自然是只能有一个方向，即数据只能正向流动，而不能方向流动，这样的机制可以保证数据接口的统一，避免了数据被污染，在`React`的官方网站上有这样一个图，可以清晰的表示*数据循环*的方向：
	
![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/data-flow.png)



	
