---
layout: post
title: "shadow dom & web components(2)"
modified: 2015-03-09 17:56:04 +0800
tags: [shadow dom,web components]
category: blog
---

这里主要补充一下上一篇日志没有讲到的地方。

###使用多个root

在某些情景下，我们可能会将多个`shadow dom tree`的`root`挂载到同一个`host`上，这时就会出现问题：

	<div id="example1">Light DOM</div>
	<script>
		var container = document.querySelector('#example1');
		var root1 = container.createShadowRoot();
		var root2 = container.createShadowRoot();
		root1.innerHTML = '<div>Root 1 FTW</div>';
		root2.innerHTML = '<div>Root 2 FTW</div>';
	</script>

![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-9.png)

这段代码只输出了`root2`，并没有输出`root1`，这是因为整个`shadow dom`的显示，可以看做是一个**栈**，也就是有**后进先出**的原则，在这里的体现就是**只显示最后被创建的shadow dom tree**。

`root`间的这种先后关系，我们可以通过`olderShadowRoot`来获取

	root2.olderShadowRoot === root1; // true
	

那么是不是就没有办法实现将多个`shadow dom tree`挂载到同一个`host`上了呢？

当然不是，我们可以通过使用`<shadow>`作为占位符，使一个`host`上可以挂载多个`shadow dom`。

	<div id="example2">Light DOM</div>
	<script>
		var container = document.querySelector('#example2');
		var root1 = container.createShadowRoot();
		var root2 = container.createShadowRoot();
		root1.innerHTML = '<div>Root 1 FTW</div><content></content>';
		root2.innerHTML = '<div>Root 2 FTW</div><shadow></shadow>';
	</script>


![image](https://raw.githubusercontent.com/tobeyouth/tobeyouth.github.com/master/_postsimage/webcomponents-10.png)

通过在`root2`的内容中添加`<shadow>`占位符，我们实现了将`root1`也挂载到`host`下。显示上顺序是`root2`在`root1`上面，这是因为我们的`<shandow>`占位符的位置决定的，如果将`<shadow>`换个位置，就会看到显示顺序改变了。

如果一段内容中有多个`<shadow>`占位符，那么将只有个第一个占位符起作用，其他的全都无效。


###插入点(insertion points)

上一篇博客说了如何通过`<content>`创建一个可容纳信息的容器，这种容器又被称作**insertion points**，这个过程也被称为**distribute**。通过**distribute**过程，我们可以将对应的内容放到正确的位置上，但是在`shadow dom tree`的`root`上，是无法获取到`<content>`节点的。

	<div><h2>Light DOM</h2></div>
	<script>
		var root = document.querySelector('div').createShadowRoot();
		root.innerHTML = '<content select="h2"></content>';

		var h2 = document.querySelector('h2');
		console.log(root.querySelector('content[select="h2"] h2')); // null;
		console.log(root.querySelector('content').contains(h2)); // false
	</script>
	

虽然我们没有办法获取`<content>`中的内容，但是我们可以使用`getDistributedNodes`方法拿到被**distribute**的元素。

	<div id="example4">
	  <h2>Eric</h2>
	  <h2>Bidelman</h2>
	  <div>Digital Jedi</div>
	  <h4>footer text</h4>
	</div>

	<template id="sdom">
	  <header>
	    <content select="h2"></content>
	  </header>
	  <section>
	    <content select="div"></content>
	  </section>
	  <footer>
	    <content select="h4"></content>
	  </footer>
	</template>

	<script>
		var container = document.querySelector('#example4');

		var root = container.createShadowRoot();
		var t = document.querySelector('#sdom');
		var clone = document.importNode(t.content, true);
		root.appendChild(clone);

		var html = [];
		[].forEach.call(root.querySelectorAll('content'), function(el) {
		 	html.push(el.outerHTML + ': ');
		 	var nodes = el.getDistributedNodes();
		 	[].forEach.call(nodes, function(node) {
				html.push(node.outerHTML);
			});
			html.push('\n');
		});
		console.log(html.join(''));
		/*
		<content select="h2"></content>: <h2>Eric</h2><h2>Bidelman</h2>
		<content select="div"></content>: <div>Digital Jedi</div>
		<content select="h4"></content>: <h4>footer text</h4>
		*/
	</script>
	
通过输出的结果，可以看到`el.getDistributedNodes`方法取到的正是被**distribute**的元素。

通过`getDistributedNodes`我们可以在`root`中拿到`<content>`内部的元素，与之类似，我们还可以通过`getDestinationInsertionPoints`来获取`insertion points`的信息：

	<div id="host">
		<h2>Light DOM</h2>
	</div>
	<script>
		var container = document.querySelector('div');
	 	var root1 = container.createShadowRoot();
		var root2 = container.createShadowRoot();
	 	
	 	root1.innerHTML = '<content select="h2"></content>';
	 	root2.innerHTML = '<shadow></shadow>';

	 	var h2 = document.querySelector('#host h2');
	 	var insertionPoints = h2.getDestinationInsertionPoints();
		[].forEach.call(insertionPoints, function(contentEl) {
			console.log(contentEl);
	 	});
	 	
	 	/*
	 	<content select=​"h2">​</content>​
		<shadow>​</shadow>​
	 	*/
	</script>
	

###事件模型




	