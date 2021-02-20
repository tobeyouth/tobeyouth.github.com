---
layout: post
title: "prototype And \_\_proto\_\_ in javascript"
modified: 2015-01-30 10:39:03 +0800
tags: [javascript]
category: blog
---

昨天在知乎上看到一个问题很有意思：[javascript一个函数实例有prototype属性吗?](http://www.zhihu.com/question/27862093)。题目大意如下：

	var Dog = function() {};
	Dog.prototype.name="HI";
	
	var Tom = new Dog();
	console.log(Tom.proptotype); // undefinded
	
按照习惯思路，这个`prototype`应该返回定义的那个对象才对，出现这个结果，竟一时不知道问题在哪儿。

按照以往的经验，我们知道其实还可以通过`__proto__`这个属性来访问`Tom`的上层对象，这里我们来访问一下看看：

	console.log(Tom.__proto__); // Object {name: "Hi"}
	
终于有返回结果了。

静下来思考一下这是为什么？

在以往的经验中，我们知道 js 中的对象可以通过`protoype`继承方法和属性，那么这个`__proto__`又是什么？我们来看下 MDN 上的介绍：

*The __proto__ getter function exposes the value of the internal [[Prototype]] of an object.*

这里面出现了另一个新的名词`[[Prototype]]`，这个又是啥？

暂时先看回我们最开始的那段代码，通过`constructor`这个属性，我们可以看看 Tom 的构造函数是什么：

	console.log(Tom.constructor); // function() {};

指向了 Dog ，正确的输出了我们想要看到的构造函数 `Dog`。那么再看看`Tom.constructor.prototype`和`Tom.__proto__`是什么关系：

	console.log(Tom.constructor.prototype === Tom.__proto__); // true
	
结果证明这两个值是一样的！

再回来看 MDN 上的描述，原来这个`[[Prototype]]`就是对象构造函数的`prototype`对象，也就是说 `__proto__` 这个属性是指向 `[[Prototype]]` 的。

那么能不能改变 `__proto__`呢？答案是当然可以：

	Tom.__proto__ = null;
	console.log(Tom.constructor.prototype === Tom.__proto__); // false
	
结果是很合乎预期的，但是这个 `[[Prototype]]` 属性有没有被改变呢？

我没可以通过 `ES5` 中的一个新方法 `Object.getPrototypeOf()` 来查看：

	console.log(Object.getPrototypeOf(Tom)); // Dog {name: "HI"}
	console.log(Tom.__proto__); // null
	
可以看到，虽然 `__proto__` 被修改了，但是实际上 `[[Prototype]]` 是不能被修改的，也就是说这个 `[[Prototype]]` 是对象的内在属性。那么现在，我们来给 `[[Prototype]]` 做个简单的定义：

*[[Prototype]] 是对象的内在属性，指向对象构造函数的原型。在现代浏览器中，\_\_proto\_\_是个指向这个属性的指针；另外，还可以通过 Object.getPrototypeOf() 方法来访问这个属性。*

看到这里是不是有点晕，`prototype` 又是什么呢？

*prototype 实际上是对象构造函数（也就是 constructor 所指向的东西）的一个属性，指向构造函数的原型。它是可设置的。*

这里再来说一下 `constructor`，`prototype.constructor` 属性其实也是一个指针，指向对象原型的构造函数，所以我们是可以修改这个属性的。

最后再来上个图，表示一下这几个属性之间的关系：

![image](http://pic1.zhimg.com/bb6c98abf029dd84488058dbf4fc4692_r.jpg)


ps:常用的关于prototype的方法

1. 获取元素的`[[Prototype]]`:`obj.__proto__` 和 `Object.getPrototypeOf(obj)` 。

	这两个方法的不同之处在于:`obj.__proto__` 是个指针，即可 `get` 也可 `set` 。所以如果有人修改了这个属性，那么这个指针将指向修改后的位置，也就不再是`[[Prototype]]`了。但是`Object.getPrototypeOf(obj)`这个方法，看名字也能看出来，只可以 `set` ，所以取到的一定是`[[Prototype]]`。
	
2. 判断对象间之间的关系：`A.prototype.isPrototype(B)`。

	通过这个方法，我们可以判断得知 `A` 和 `B` 是不是来源于同一份原型。
	
3. 最后再奉献一个判断一个属性是否是继承于原型的方法：

	使用`A.hasOweProperty(method)`判断`method`是否是对象自己定义的。
	使用`Object.getPrototypeOf(A).hasOweProperty(method)` 判断`method`是否是在原型中定义的。
	



	


