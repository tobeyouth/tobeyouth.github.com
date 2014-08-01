---
layout: post
title: "something about touch"
modified: 2014-07-31 11:12:20 +0800
tags: [touch,javascript]

---

在移动前端开发的过程中，我们碰到的最多也是最头痛的问题就是关于`click`和`touch`带来的一系列交互问题。

在PC端，我们只需要监听`click`事件，就可以完成绝大部分的点击等动作，但是在移动端，由于手势操作的多样性(点击，双击，长按，多手指操作等)，在对用户动作的响应上，需要做出很多额外的判断。更由于各个平台的不同，不同的系统，不同的浏览器都会对用户手势进行或多或少的`改进`和`修正`，导致在交互上的问题频发，需要考虑的情况也更多。接下来，就随便谈谈关于`touch`的各种。

###touch与click

如上面所说的，在PC端，用户的鼠标交互通常可以分解为`mousedown`,`mouseup`,`click`,`doubleclick``mousemove`，`mouseover`和`mouseout`，更细致一些的浏览器中还包括`mouseenter`和`mouseleave`。这一系列的各种事件，基本可以cover用户所有的鼠标操作。

但是在移动端，浏览器开放给我们的，就只包括了`touchstart`,`touchend`,`touchmove`,`touchcancel`这一系列的`touch`事件。当然，大多数浏览器其实也给我们预留了`click`等从鼠标时代遗留下来的事件。但是移动端毕竟不同于PC端，更何况各家浏览器对`click`的处理方式也不尽然相同。

在ios中，每次`click`都会延迟大概300ms之后才被触发；但是在新版本的android中，这个延迟被去掉了，也就是说你的`click`会在`touchend`之后被立即触发。不要小瞧这300ms，不算长，但是却会使得用户体验大打折扣。

这个时候，为了统一这300ms的体验，我们通常会舍弃`click`事件，简单粗暴的方法是直接使用`touchend`来绑定本应该绑定在`click`上的事件：

		var el = document.getElementById('button');
		el.addEventListener('touchend',handler);

看上去不错，但是为什么说这种方法简单粗暴呢？

`touchend`顾名思义，是手指在某元素上结束`touch`时触发的，虽然结束时是在该元素上触发，但是`touch`事件开始时，却不一定是在该元素上。所以会造成很多误操作的机会，于是我们可以这样改造：

		var el = document.getElementById('button'),
			target = null;
		
		el.addEventListener('touchstart',function () {
			target = this;
		});
		
		el.addEventListener('touchend',function () {
			if (target !== this) {
				return;
			};
			target = null;
			
			// code
			
			e.preventDefault();
		});
		
这样，通过一个变量，就可以控制`点击`事件的触发元素了。不过，事情远没有达到无懈可击的地步，虽然绑定了`touchstart`和`touchend`，使得看上去可以cover住大部分的`点击`事件了，但是还有一种极端情况，就是用户的手指点在了该元素上，绕了一圈之后又回到了该元素上，这时候放开手指，是不是也要触发`点击`事件呢？

结果当然是，不！

于是我们可以再进行一下改造：
	
		var el = document.getElementById('button'),
			target = null;
		
		el.addEventListener('touchstart',function () {
			target = this;
			setTimeout(function () {
				target = null;
			},200);
		});
		
		el.addEventListener('touchend',function (e) {
			if (target !== this) {
				return;
			};
			target = null;
			
			// code
			
			e.preventDefault();
		});
	
给`target`的生命周期设置为200ms，这样的话，就可以比较精确的判断出用户是否具有`点击`的意图，从而触发回调函数了。

另外在`touchend`中加入`e.preventDefault();`，目的是为了阻止接下来触发的`click`事件，当然，如果在其他地方绑定了`click`事件，或者在多人合作开发的项目中，并不建议这么做。

但是如果每次`点击`都要这么绑定的话，确实是有点太费劲了。这种方法只适用于交互较少的模块，偶尔写一写这样的代码，还是要比引入个第三方事件包装组件要省力得多的。
		
在通常的业务开发中，我们一般使用`zepto`。在`zepto`中`touch.js`文件，针对移动端封装了`tap`,`singleTap`,`doubleTap`和`longTap`来捕捉用户的各种`点击`事件。

在`zepto`中，所有的这些事件都是绑定在`body`上的，原理跟我们上面的方式大致相似。但是`tap`会在很多情况下出现`点透`的现象。也就是说本来我想点一下浮层上关闭的按钮，浮层如愿关闭之后，结果却连按钮下面的元素也被点中了。具体的研究可以看[这篇文章](http://blog.youyo.name/archives/zepto-tap-click-through-research.html)。

简单来说，其实问题还是发生在那300ms的延迟上。也就是说在`touchend`结束300ms之后，元素又触发了`click`事件，但是此时由于弹层已经被关闭，所以浏览器会默认找到点击位置最近的元素，在这个元素上触发`click`事件。

解决方案也是有的，就是自己在`zepto`的`touch.js`中绑定上一个`click`事件，同时加上判断就可以了，这里就不做赘述了。

在`touch`还是`click`这个问题上，还有个第三方的组件[fastclick](https://github.com/ftlabs/fastclick)。

这个组件的思路是重新定义元素的`click`事件，从而避免掉那300ms带来的各种问题。`fastclick`的使用方法对代码的入侵性比较小，如下所示：

		var testB = document.getElementById('test-b');
		FastClick.attach(testB);
		test.addEventListener('click',handler);

这时`click`事件会在`touchend`触发的同时，立即被触发，再也没有延迟，世界一下变得清爽了：）

####长按与右键

在PC端，我们通过鼠标右键来进行浏览器层级的操作，例如复制文字，下载图片等。

在不想用户进行操作的时候，可以通过`click`事件绑定判断keycode同时`e.prenvetDefault()`来禁止右键的默认行为。

在移动端，类似的功能是通过长按来实现的。在`zepto`中封装了`longTap`事件，可以用来完成这个功能。另外，在很多时候，如果不想触发这个交互动作，可以在页面css中加上这些：

		-webkit-user-select:none; // 禁止用户选择文本
		-webkit-touch-callout:none;// 禁止长按出现菜单
		
	





