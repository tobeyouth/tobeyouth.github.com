---
layout: post
title: "webpack，browserify简介"
modified: 2014-09-15 14:40:07 +0800
tags: [webpack,browserify,nodejs,javascript]
category: blog
---


*webpack*和*browserify*是两个用于编译打包前端资源的工具。于此相同的还包括有百度的*fis*和支付宝的*spm*。主要的工作都是完成*前端工程化*工作中的一环。

##Browserify

*Browserify*是由[substack](https://github.com/substack)开发的，用于实现在浏览器环境中运行*npm*上的*commonjs*模块的编译工具，本质上这并不是一个专门为打包合并前端资源所开发的工具，根据其官网的介绍上，其初衷就是为了让*npm*中的*commonjs*模块可以运行在浏览器中。

使用*Browserify*可以在浏览器所运行的js中，使用同*nodejs*中一样的`require`语法，将*npm*中的*commonjs*模块引入，编译并打包到浏览器运行的js中。

使用*Browserify*的引入特性，就可以实现自动加载依赖，并且打包合并js文件的功能。

但是比较局限的是，*Browserify*只能支持js文件，对于css文件的依赖还没有支持。

###transforms

*Browserify*用于处理文件的模块都统称为`transforms`，例如将`.coffee`格式的文件转换为`.js`文件等。

nodejs社区贡献了很多`transforms`模块，可以在这里看到：[transforms list](https://github.com/substack/node-browserify/wiki/list-of-transforms)。

- 项目地址：[browserify](http://browserify.org/)
- 用法介绍：[handbook](https://github.com/substack/browserify-handbook)
- 浏览器支持情况：

![image](http://browserify.org/images/testling_badge.png)

###配置

*Browserify*的配置都写在*package.json*中，其基本格式如下：

	{
		"name": "mypkg",
		"version": "1.0.0",
		"main": "main.js",
		"browser": "browser.js"
	}
	
当在*node*环境中运行时，执行`require(mypkg)`，将从`main.js`中获取对外接口；在浏览器环境中运行时，执行`require(mypkg)`，将从`browser.js`中获取对外接口。

#####给本地模块命名

除此之外，在`browser`参数中，还可以给本地模块命名，如下所示：

	{
		"browser" : {
			"foo": "./js/foo.js"
		}
	}
	
在其他的模块中，直接通过以下方式就可以调用重命名的模块了：

	var foo = require('foo');
	
#####browser.ingnore

在`browser`配置中，还可以将某一模块的忽略，使其对外接口为一个空的对象，设置方法如下：

	{
		"browser" : {
			"foo": false
		}
	}
	
这时在其他模块中使用`foo`时，将返回一个空对象：

	var foo = require('foo');
	console.log(foo);	// object{}
	
#####browserify.transform

在`browserify`中，可以配置*transforms*，例如：

	{
		"name": "mypkg",
		"version": "1.0.0",
		"main": "main.js",
		"browserify": {
			"transform": ["coffeeify"]
		}
	}
	
#####配置对外全局变量

使用`browserify`命令，会自动将模块引入到一个`package`中，内部定义的方法在`window`作用域下不可用，如果需要想要在页面中可用的话，可以通过`--standalone`命令来指定对外的变量：

	$ browserify foo.js --standalone pageInit > page.js


关于*Browserify*的简介就先到这里，日后有时间的话，可以再来说一下*Browserify*的API和插件开发。

##webpack

与*Browserify*类似（甚至在*webpack*的文档中也多出列出了与*Browserify*的对比），*webpack*也是一个用于解决模块依赖、合并的工具。

与*Browserify*不同的是，*webpack*更接近于前端的使用方式，在*Browserify*中，`require`方法同步并且执行模块，在*webpack*中提供了一种`require.ensure`的方法，只加载模块，而不执行模块；与之类似的，在*webpack*中还提供了一个`require.include`方法，用于在函数内容加载模块，而不执行。

###loader

在*webpack*中*loader*的概念类似于*Browserify*中的*transforms*，也是完成解析编、编译代码工作的部分。

*loader*可以在`require`和配置文件`webpeck.config.js`中设置。

在`require`中设置：

	require("jade!./template/index.jade");
	
在`webpect.config.js`中设置：

	{
		module: {
			loaders: [
				{
					test: /\.jade$/gi,
					loader: "jade"
				}
			]
		}
	}
	
在*loader*中比较牛逼的一点（当然也可以说是比较奇特的一点），就是你可以在*loader*后加入`query`。所谓`query`就像我们经常在url中看到的这种格式：

	http://example.url.com?type=image
	
其中的`type=image`就是`query`，在*loader*中，也可以通过这种方法配置一些定向条件，以`require`为例：

	require("url-loader?mimetype=image/png!./expample.png")

在`webpack.config.js`中，还可以通过`query`字段定义，如下：

	{
		module: {
			loaders: [
				{
					test: /\.jade$/gi,
					loader: "jade",
					query: {
						mimetype: "image/png"
					}
				}
			]
		}
	}
	
###plugins

与*loader*不同的是，*plugins*的主要职责并不是编译文件。*plugins*提供了各种配置，可用于在*webpack*一些*task*。（虽然个人感觉这个功能和*grunt*和*gulp*的*plugins*有很大重合吧）

如果只是将*webpack*作为*grunt*或者*gulp*中的一个*task*的话，其实绝大部分*plugins*都是没有必要使用的，因为这些脚手架工具中也会集成这些*task*




	









