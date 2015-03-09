---
layout: post
title: "shadow dom &amp; web components"
modified: 2015-03-09 13:20:09 +0800
tags: [shadow dom,web components]
---
###what is web components

上面说了`shadow dom`，再来说一下什么是`web components`。根据[w3c](http://www.w3.org/TR/2013/WD-components-intro-20130606/)上一个版本的文档来看，所谓`web components`分为以下几部分：

- Template:定义模板
- Decorators:基于模板定制交互和样式
- Cuntom Element:自定义元素
- Shadow DOM:下面会有详细介绍
- Imports:加载上面所有一切的方式

看上去这几部分貌似功能有些重复，所以最新的[w3c](http://www.w3.org/wiki/WebComponents/)文档重新定义了`web components`所包含的部分：

- Shadow DOM
- Custom Elements
- HTML Imports


###what & why shadow dom?

首先来说说什么是`shadow dom`。从字面意义上解释，就是**藏在阴影里的dom**，但什么又是藏在阴影里？

据我们所知，在网页中，存在一个**根节点**，也就是我们常说的`document`节点，整个网页都基于此渲染，我们称之为`dom tree`。所谓`shadow dom`可以理解为页面的`dom tree`中嵌套的另一个`dom tree`，但却不是`iframe`，而是实实在在的页面元素。

可能这么说还是有点儿抽象，我们来看一个简单的实例：

![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-1.png)

这是facebook右侧的推荐信息。

先从最简单的思路来，如果做这样一个组件，我们需要用 div+css 写出来一个这样的结构，然后嵌入的页面里，再把对应的操作交互的 js 和控制样式的 css 引入到页面，这样页面中才会呈现这个结构。

再进一步，我们发现要引用这样一个模块，需要在 **结构，css，js** 三个方面引入文件才可以，这实在是太麻烦了，所以我们发明了使用 js 来渲染该模板的方法 —— 也就是用 js 拼接模板，同时绑定交互，这样也就说只需要引入 **js，css** 两种文件即可。同时因为这个模块的结构是由 js 来控制的，所以如果发生改动，只需要修改 js 文件即可，不用再逐个模板去更新了。

另一种方式，我们在模板文件中通过形如 `<include>,<import>`等方式引入一整个打包好的模板片段，这个片段中集成了 **结构，css，js**。但是这么做也会存在问题 —— 如果每个小模块都单独引入自己的 css 和 js 文件，那么对于一整个页面来说，引入的外部文件就会过多，拖慢页面加载速度。

同时，以上这三种思路还有个共同的弊端，就是其实所生成的这个模块是插入到整个页面的`dom tree`中的，如果不小心存在了同名的 class ，css 很容易产生覆盖的情况，这个风险从代码角度很难控制。

**这个时候，就需要 `shadow dom` 登场了**。

上面我们说过，`shadow dom`是整个页面的`dom tree`中的另一个`dom tree`。

这是什么意思呢？

也就是说`shadow dom`中的内容具有自己单独的`dom`作用域 —— 简单的说，就是解决了样式命名冲突覆盖的问题。

但仅仅这样一个好处并不足以说明`shadow dom`是个还算不错的东西。再来看同名 class 带来的问题：

使用jquery的时候，我们常用`$('.className')`的方式来为dom节点绑定事件或者保存数据，如果不小心出现了同名 class ，那么本来要给**A**元素绑定的事件，也会给本来不应该绑定该事件的**B**元素绑定上。

使用`shadow dom`同样可以解决这个问题。

再来说更深一层的意义。随着现在前端工程的日渐庞大，前端开发工作也开始逐渐分层，比如有的人就会专门的负责通用组件开发，另一部分人会专门负责页面逻辑的开发。在遇到一些通用组件的时候，显然调用通用组件是最合乎工程思想的做法，但是一些复杂组件需要引入和配置的东西太多，难免会拖慢开发效率，同时也很容易导致bug的出现。这时`shadow dom`的优势才真正的体现出来，使用`shadow dom`的本质是将一部分业务逻辑封装在一个声明化的标签当中，专门负责通用组件开发的人可以只专注于组件内部逻辑的实现，而负责页面逻辑开发的人，可以专注于页面逻辑，彼此之间不会造成任何影响。我个人认为这才是`shadow dom`最有价值的地方。


###how to use shadow dom

写了这么多介绍，来看更具体的使用：

#####定义内容

`shadow dom`遵循`dom tree`的结构，根节点被称为`root`，`shadow dom`的内容会被挂载到定义的`root`下，并且对外公开的也只用`root`而已（也就是说从页面层级上，是获取不到这个`shadow dom tree`的，只能获取到所挂载的`root`节点）。

直接看代码：

	<!-- 所谓的root -->
	<div class="root" id="root">I'm Root</div>
	
	<!-- shadow dom 的结构 -->
	<template id="template">
		<div class="hd">My name is:</div>
		<div class="bd">
			<content>SHADOW</content>
		</div>
		<div class="ft">Let's write a shadow dom</div>
	</template>
	
	<script type="text/javascript">
		var root = document.querySelector('#root').createShadowRoot();
		var template = document.querySelector('#template');
		root.appendChild(template.content);
	</script>
	
以上代码就会见简单的生成一个`shadow dom`，并插入到`root`节点中，可以把代码复制到页面里，看看会发生什么：

![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-2.png)

我们在`<content>`标签里写的内容并没有被显示，显示的是我们在`root`元素中写入的`textContent`;

我们再来写句js，改变一下`root`中元素中的内容：

	document.querySelector('#root').textContent = 'Changed name';

![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-3.png)

这时我们发现，显示中的`<content>`改变了，也就是说**可以通过改变root元素的textContent改变shadow dom中定义的content部分**。

但是真正的开发时，情况不会这么简单，如何定义多个`<content>`呢？

继续看：

	<!-- 所谓的root -->
	<div class="root" id="root">
		<div class="hd">I'm Root hd</div>
		<div class="bd">I'm Root bd</div>
		<div class="ft">I'm Root ft</div>
	</div>
	
	<!-- shadow dom 的结构 -->
	<template id="template">
		<div class="hd">
			<content select=".hd"></content>
		</div>
		<div class="bd">
			<content select=".bd"></content>
		</div>
		<div class="ft">
			<content select=".ft"></content>
		</div>
	</template>
	
	<script type="text/javascript">
		var root = document.querySelector('#root').createShadowRoot();
		var template = document.querySelector('#template');
		root.appendChild(template.content);
	</script>


![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-4.png)


通过在`<content>`中定义`select`参数，可以在`shadow dom tree`中选择性的显示不同内容。除了通过`.selector`这种方式之外，还可以通过其他的选择器方式来选择不同的内容作为`<content>`中的内容。



#####封装样式

上面的部分定义了`shadow dom tree`的结构和内容，但是光有结构是远远不够的，还需要封装样式，才算是一个可用的`shadow dom tree`。

下面来看代码：

	<!-- 所谓的root -->
	<div class="root" id="root">
		<div class="hd">I'm Root hd</div>
		<div class="bd">I'm Root bd</div>
		<div class="ft">I'm Root ft</div>
	</div>
	
	<!-- 其他 -->
	<p>other</p>
	<div class="other">
		<div class="hd">I'm Other hd</div>
		<div class="bd">I'm Other bd</div>
		<div class="ft">I'm Other ft</div>
	</div>
	
	<!-- shadow dom 的结构 -->
	<template id="template">
		<style>
			.hd {color:#999;}
			.bd {color:#666;}
			.ft {color:#333;}
		</style>
		<div class="hd">
			<content selected=".hd"></content>
		</div>
		<div class="bd">
			<content selected=".bd"></content>
		</div>
		<div class="ft">
			<content selected=".ft"></content>
		</div>
	</template>
	
	<script type="text/javascript">
		var root = document.querySelector('#root').createShadowRoot();
		var template = document.querySelector('#template');
		root.appendChild(template.content);
	</script>


![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-5.png)

可以看到我们定义的三个样式只作用于了`shadow tree`中的节点，这就是**样式封装**。

除此之外，在`shadow dom tree`中，我们还可以配置`root`节点的样式：

	<!-- 所谓的root -->
	<div class="root" id="root">
		<div class="hd">I'm Root hd</div>
		<div class="bd">I'm Root bd</div>
		<div class="ft">I'm Root ft</div>
	</div>
	
	<!-- 另一个root -->
	<section class="root" id="rootOther">
		<div class="hd">I'm Other Root hd</div>
		<div class="bd">I'm Other Root bd</div>
		<div class="ft">I'm Other Root ft</div>
	</section>
	
	<!-- shadow dom 的结构 -->
	<template id="template">
		<style>
			:host {padding:3px;border:1px solid #333;display: inline-block;}
			:host(div) {background-color:#fff;}
			:host(section) {background-color:#f50;}
			.hd {color:#999;}
			.bd {color:#666;}
			.ft {color:#333;}
		</style>
		<div class="hd">
			<content selected=".hd"></content>
		</div>
		<div class="bd">
			<content selected=".bd"></content>
		</div>
		<div class="ft">
			<content selected=".ft"></content>
		</div>
	</template>
	<template id="othertemplate">
		<style>
			:host {padding:3px;border:1px solid #333;display: inline-block;}
			:host(div) {background-color:#fff;}
			:host(section) {background-color:#f50;}
			.hd {color:#999;}
			.bd {color:#666;}
			.ft {color:#333;}
		</style>
		<div class="hd">
			<content selected=".hd"></content>
		</div>
		<div class="bd">
			<content selected=".bd"></content>
		</div>
		<div class="ft">
			<content selected=".ft"></content>
		</div>
	</template>
	
	<script type="text/javascript">
		var root1 = document.querySelector('#root').createShadowRoot();
		var root2 = document.querySelector('#rootOther').createShadowRoot();
		var template1 = document.querySelector('#template');
		var template2 = document.querySelector('#othertemplate');
		root1.appendChild(template1.content);
		root2.appendChild(template2.content);
	</script>

![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-6.png)

可以通过`:host`来选中`root`节点，并且可以通过`:host(selecor)`的形式选择不同类型的`root`节点。

#####突破封装




