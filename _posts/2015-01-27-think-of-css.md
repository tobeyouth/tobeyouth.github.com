---
layout: post
title: "think-of-css"
modified: 2015-01-27 17:55:31 +0800
tags: [css,front-end,framework]
---


###css风格

- OOCSS:[Object Oriented Css](https://github.com/stubbornella/oocss/wiki)
1. Separate structure and skin
2. Separate container and content
3. Avoid the descendent selector (i.e. don't use .sidebar h3)
4. Avoid IDs as styling hooks
5. Avoid attaching classes to elements in your stylesheet (i.e. don't do div.header or h1.title)

- SMACSS:[Scalable and Modular Architecture for Css](https://smacss.com/)
	1. Base (element only selector)
	2. Layout (id selector + layout class)
	3. Module (class only selector + descendant selector)
	4. State (class selector)
	5. Theme (override below rules)
	6. Avoid tag selectors for common elements
	7. Use class names as the right-most selector

- ACSS:[Atomic CSS](http://www.smashingmagazine.com/2013/08/02/other-interface-atomic-design-sass/)
	1. use atomic style
	2. don't use concrete classname

- BEM:[Block Element Modifier](http://bem.info)
	1. use `blockname-modulename`
	2. no abstract classname
	
---
	
###会不会有更好的风格呢？

- 期望：

	1. 便于移植
	2. css文件尽量精简
	3. 可以应对频繁改动，而不致使复杂度增高
	4. 和html,js解耦
	5. 表示状态更清晰
	
---

	
###组件化css


听起来很像`BEM`？是有些相似，但`Block`和`Components`的概念还是有所不同。举个例子：
		
		// BEM
		.user-block-hd {
			border:1px solid #333;
		}
		.user-block-button {
			background-color:#ccc;
		}
		.user-block-button-success {
			background-color:#eee;
		}
		
		// 组件化
		.user-block .hd {
			border:1px solid #333;
		}
		.user-block .button {
			background-color:#ccc;
		}
		.user-block .button[data-status="success"] {
			background-color:#eee;
		}

简单的来说：

- 将每个组件进行抽象化，在需要改动的项目中，覆盖样式
- 将交互状态放在形如`[data-status]`这样的选择器中
- 每个组件对应一个默认的基础可用样式，可用直接`import`使用
- 页面布局与组件分开，组件更关注细节

###组件化css面临的问题：

- 容易被覆盖（不知道已经存在该组件，所以覆盖）
- 生成的css文件并不精简（因为有`[data-status]`这样的选择器）
- js挂载事件容易重复（组件的交互逻辑封装在组件js内时）
- 暂时想到这么多...

### 解决方案：

- *容易被覆盖*:组件平台化，推广通用组件
- *css文件不精简*:暂时没有解决方案 囧
- *js挂载事件容易重复*:使用`react`最佳

