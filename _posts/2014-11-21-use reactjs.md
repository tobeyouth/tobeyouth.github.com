---
layout: post
title: "use reactjs"
modified: 2014-11-21 13:54:04 +0800
tags: [javascript,react,gulp,browserify]
category: blog
---

最近在一个项目中使用了`React`作为主力开发框架，写篇总结，说说自己的感受和踩过的坑。

跟`Angular`的对比，网上可以搜到各种文章，着重推荐这篇[From AngularJS to React: The Isomorphic Way](http://blog.risingstack.com/from-angularjs-to-react-the-isomorphic-way/?utm_source=feweekly&utm_campaign=issue40&utm_medium=mail)，讲的比较详实，同时还提出了一种名为`The Isomorphic Way`的思路。

首先来说一下我在这个项目中使用`React`的方式，在`React`的官方网站和很多介绍文章里都表示 - `React`不是一个完整的`MVC`或`MVVM`框架，事实也是如此`React`其实应该算做是一个增强型的`view`模板，当然，你说他是`Web Component`其实也是合理的。在这里就不讨论它的分类问题了，毕竟分类都是人为的，怎么在实际中更好的应用，才是广大使用者更关心的话题。

由于`React`除了部分的`Event`能力之外，并不具备完整的`MVC`能力，所以需要其他工具或者框架来辅助其完成整个前端逻辑的开发，在这次的项目中，我选择的是[browserify](http://browserify.org)，其在浏览器端，提供了`nodejs`中的一些模块调用，使开发过程类似于`node`的开发过程。

当然，也可以配合其他框架，比如`jquery`,`zepto`,`requirejs`,`seajs`什么的，使用`browserify`其实主要就是我比较懒...o(╯□╰)o

再说回`React`，其实`React`的核心就是`View`，也就是UI层面，在使用`Backbone`时，我们会将整个页面拆分成不同的组件，然后再组合成不同的`collections`，最后再组成一整个页面，使用`React`也类似，只不过名字在这里一变，变成了各种`component`，最终组成的`Domtree`是类似这样的：


	<Card>
		<Cardheader>This is Header</Cardheader>
		<Cardbody>
			<Userlist data={this.props.list} />
		</Cardbody>
	</Card>
	
这里面的类似于dom元素的`<Card>`等就可以理解为`component`，是不是有点像`xml`？其实在`React`的官方文档里，这种形式被称为`jsx`，可以理解成用js写的xml，所以`React`的文件，也都是以`jsx`作为后缀的。

在开发过程中，如果我们新建一个`UI组件`(或者叫模块也好，这个随便)，这个时候我们总是需要传入一些数据才行，没有数据，就没办法渲染模板；在很多时候，我们用一些模板引擎的`render`方法，将数据作为参数传到`render`函数中，对模板进行渲染。

在`React`中，如你所见，我们的`UI组件`是类似于html标签的，显然没有一个方法入口，可以将数据作为参数传入，但是可以通过类似于html参数的方式，将数据传入，看🌰：

	var Userlist = React.renderCompoent(
		<Userlist data={window.user} />,
		document.body
	);
	
通过在`<Userlist>`上标记`data`属性，我们就可以将所需要的数据传入到`UI组件`中；在`Userlist`的具体方法中，我也可以通过`this.props`这个对象，来获取其标签上写入的属性。

######props and state

说到`props`，这个对象基本可以理解为*组件与外界通讯的唯一方式，而且还是单向的*，也就是说它是外界对组件的输入入口，在组件本身内部，只能对其`get`，而不能进行`set`。

在组件中，对于数据的操作，建议使用`state`来进行，可以跟`props`有个区分，隔离开组件内外的环境。但是这里其实有个坑，就是假如有一份数据，既要A组件内可更新，又要再其他组件内更新，这个时候，无论是用`props`还是`state`其实都是不大合适的。理想的方法是参照`Flux`规范，使数据的流动成为一个闭环，这个时候，可能我们就需要在更新数据的每个阶段添加一个事件，再在外部监听这个事件去更新数据。关于`React`和`Flux`的种种，以后会再单开篇文章讨论，这里就不展开了。

在这里说一个比较`trick`但是很实用的方法，可以让子组件可以更新父级组件的状态，方法也很简单，还是看🌰吧：

	var Parent = React.createClass({
		"getInitialState":function () {
			return {
				"nums": 0
			}
		},
		"render": function() {
			return (
				<div className="wrap">
					<p className="num">
						{this.state.nums}
					</p>
					<Son clickHandle={this.addNum} nums={this.state.nums}>
						Click Me!
					</Son>
				</div>
			)
		},
		"addNum": function(nums) {
			var num = nums;
			this.setState({
				"nums": num
			});
		}
	});
	
	var Son = React.createClass({
		"render": function() {
			return (
				<button onClick={this.clickHandle}>
					Click Me!
				</button>
			)
		},
		"clickHandle": function () {
			var num = this.props.nums + 1;
			if (typeof(this.props.clickHandle)) {
				this.props.clickHandle(num)
			};
		}
	});
	
通过将父元素的方法绑定在子元素的`props`上，就可以实现在子元素中调用父级元素的方法啦。

######方法继承 - mixins

在以往的开发经验中，我们经常会用一些办法实现*方法继承*，无论是通过`prototype`的方式也好，还是通过`extend`的方式也好。在`react`中，同样有方式实现*方法继承*。

在`react`中，可以通过`mixins`来实现*方法继承*，废话少说，举个🌰：

	var modal = {
		"hide": function() {
			console.log("hide")
		},
		"show": function() {
			console.log("show")
		}
	};
	
	var tips = React.createClass({
		"mixins": [modal],
		"render": function() {
			// code
		},
		"componentDidMount":function () {
			// 调用show方法
			this.show();
		}
	});
	
######元素的clone - React.addon.cloneWithProps

在开发过程中，如果我们需要复制某个元素，可以通过`React.addon.cloneWithProps`方法来实现。例如，现在有个`List`列表，但是其中的`Item`模板类型不确定，出于通用性考虑，我们可以在`List`内部`clone`该`Item`元素，这样，无论`Item`元素怎么变，只要`List`的逻辑不变，理论上可以插入任何类型的`Item`，看🌰：

	var List = React.createClass({
		"render": function() {
			var Items = this.props.data.map(function(itemData,index) {
				// 直接clone子元素
				var _item = React.addon.cloneWithProps(this.props.children);
				
				_item.props.data = itemData;
				_item.props.key = index;
			});
			
			return (
				<div className="list">
					{Items}
				</div>
			)
		}
	});
	
	var itemA = React.createClass({
		"render": function () {
			return (
				<div className="itemA">
					{this.props.data.name}
				</div>
			)
		}
	});

	var itemB = React.createClass({
		"render": function() {
			return (
				<div className="itemB">
					{this.props.name}
				</div>
			)
		}
	});
	
	// 下面是调用list和item的实例
	
	var MainFoo = React.createClass({
		"render": function() {
			return (
				<List data={this.props.data}>
					<itemA />
				</List>
			)
		}
	})
	
	var MainBar = React.createClass({
		"render": function() {
			return (
				<List data={this.props.data}>
					<itemB />
				</List>
			)
		}
	});
	
通过这个🌰，可以看出来，使用`clone`可以使`List`更具拓展性。


######根据状态改变样式

涉及到UI，就会涉及到变化。在`react`中，通过高效的`render`函数，可以允许我们根据状态的变化渲染元素的结构。通常情况下可以通过css3中新增的选择器来做，例如：

	.item[data-status="default"] {
		/* code */
	}
	.item[data-status="focus"] {
		/* code */
	}
	.item[data-status="error"] {
		/* code */
	}
	
如果不喜欢这种方式，也可以通过设置`className`的方式改变元素的样式，在`react`中，有个方便的函数，可以根据状态生成不同的`className`列表，举个🌰：

	var foo = React.createClass({
		"getInitialState": function () {
			return {
				"status": this.props.status || "default"
			}
		},
		"render": function () {
			var cx = React.addon.classSet;
			var classes = cx(
				"btn":true,
				"default": this.state.status === "default",
				"focus": this.state.status === "focus",
				"error": this.state.status === "error"
			);
			return (
				<button className={classes}>
					Click!
				</button>
			)
		}
	});
	
	
######last 

以上仅仅列举了一些我在这一段使用`react`的时间内，觉得有点儿意思的点，更多的内容可以访问[react](http://facebook.github.io/react/docs)的官方文档查看。







