---
layout: post
title: "seajs & grunt"
description: ""
category: 
tags: [seajs,grunt,javascript]
---
{% include JB/setup %}

在前端开发的过程中，我们需要面对这么几个问题：

- 相同或相似功能和模块的重复开发
- 静态资源的合并和上线优化
- 不同模块之间的依赖关系处理
- 开发效率的提升和bug数量的减少

这里说一下我比较推荐的开发模式：

#####使用`seajs`作为js代码组织工具
- 首先可以解决模块之间的依赖关系处理；
- 其次，因为采用模块化开发的形式，所以相同或相似功能的模块，可以避免重复开发（不过这要求模块的职能要单一，粒度也足够细）；
- 再次，可以避免全局变量随便定义导致的错误。

#####使用`grunt`作为工程化流程的工具
- 首先，可以有大量的第三方task可用
- 其次，使用grunt已经经过了很多公司工程化的考验，比较可靠
- 再次，可以节约人肉合并和优化的工作，减轻不必要重复工作量


#####`seajs`使用说明：

详细内容可以参考[seajs.org](http://seajs.org/docs/)，这里只说明一下几个需要注意的点：

- 使用`seajs`作为加载器，意味着你的页面中只用通过`script`标签载入一个`seajs`即可，其余模块都可以通过`seajs`加载到页面中。具体配置方法如下：

		seajs.config({
			'alias' : {
				'path/to/file' : 'path/to/file'
			}
		});
	
	在`alias`对象中，冒号左侧的为`模块Id`，右侧为`模块真实路径`，为了避免歧义，所以在合并模块的task中，将`模块Id`定义为`模块的路径`.
	
	但是要明确这点：*这两个值之间没有必然关系，id是可以随便定义的，只不过我们为了简化规则，将其定义成了路径*而已。
	

- 所谓模块化开发，就是指要将不同的功能从大段的代码中抽离出来，通过`对外方法`的形式，使得模块调用者可以方便的调用，而不用担心其他问题。初期使用`seajs`进行模块化开发，只需要记住如下的形式即可：

		define(function (requires,exports,module) {
			// 通过require，加载你所依赖的模块
			var Mod = require(ModId);
			// your code
			
			// 通过exports提供对外的方法
			exports.yourFun = your function
		});
		
#####`Grunt`使用说明：

`Grunt`不是一个代码层级的管理工具，代码层级的管理工具是`seajs`；`Grunt`是一个工程化的流程管理工具。

想了解更多，可以看看：[关于Grunt的一切](http://www.gruntjs.net/docs/getting-started/)

`Grunt`需要安装在你本地的机器上，安装步骤如下：

1. 需要安装`nodejs`，可以直接去官方网站下载安装包[download](http://nodejs.org/download/)，选择一个适合自己电脑的版本即可。
2. 运行`npm install -g grunt-cli`，将`grunt-cli`模块装到你的电脑中。
3. 运行`npm install -g grunt-init`，将`grunt-init`装到你电脑中。
3. 运行`grunt init cell-grunt-template`,安装`cell-grunt-template`，一个`grunt`的脚手架工具，可以省去你自定义`package.json`和`Gruntfile.js`的工作。
4. 安装自己喜欢得方式写代码
5. 在项目的根目录下，执行`grunt`，程序会自动压缩合并你的代码，这时可以看一下压缩合并的过程是否出错，以及合并之后的效果。
6. 如果没问题的话，可以提交测试或者上线。



