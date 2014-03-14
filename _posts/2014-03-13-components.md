---
layout: post
title: "Why components?What is components?"
description: ""
category: 
tags: [javascript,components]
---
{% include JB/setup %}

----

###起因

在从事前端开发这份很有前途的职业时，越来越发现其实前端这个行业，尤其是我们这种在互联网行业内的前端工程师，往往面临的问题不是项目的逻辑多么``复杂``，真正的问题反而是如何尽可能``快速``的还原设计稿。

说到设计稿，又要引入``UI``设计师现身说法。其实大多数时候，UI设计师想的也是如何快速的将需求人的想法精美的还原出来。

但是说到这个``快速``的问题，怎样做才能快速？

最简单的方法当然是一些基本元素的快速组合。说到基本元素的快速组合，就要说到主题上，也就是``组件``-粒度足够细的组件。

###解决方案

既然存在问题，那么就一定会有相应的解决方法：

- bootstrap
- YUI
- Google Closure Library 
- jQuery组件(最典型的例子就是jQuery-ui)
- commenJs方式组织的组件(seajs & requirejs)
- 自定义命名空间的组件管理

###一个组件应该是什么样的

- 简单
- 可组合
- 快速更新和维护
- 如果能跟其他模块进行``沟通``就更是极好的了


###个人思路

- 模板和逻辑分离(支持自定义模板)
- 交互逻辑分离
- 对外提供可以操作组件的方法

以一个``dialog``组件为例：

- 模板和逻辑分离：

	支持传入其他tpl作为模板(但是一些功能性的class或者id可以在说明文档中写清楚)
	模板中的一些公用项目，类似于``title``,``footer``一类的可以自定义。
	
- 交互逻辑分离：

	组件自带``bindEvent``方法，但是支持使用者重新定义``bindEvent``方法，新定义的方法会替换组件自带的``bindEvent``方法。
	
- 对外提供可操作组件的方法：

	1.	初始化组件之后，返回组件对象，可以操作组件对象的方法。
	2.	组件绑定全局的方法，当触发这个方法时，则操作组件。

- 使用``seajs``作为代码的组织方式，好处就是迁移到其他基于``commonJs``的框架下比较省力。



###其他的思路（更高深）

以``AngularJs``作为开端，后面跟着``React``，``DarkDom``等组件库。

大致的思路是``结构绑定交互``，有点类似于``flex``的形式。

看一段代码：AngularJs

    		<div ng-controller="TodoCtrl">
      			<span>{{remaining()}} of {{todos.length}} remaining</span>
      			[ <a href="" ng-click="archive()">archive</a> ]
      			<ul class="unstyled">
        			<li ng-repeat="todo in todos">
          				<input type="checkbox" ng-model="todo.done">
          				<span class="done-{{todo.done}}">{{todo.text}}</span>
        			</li>
      			</ul>
      			<form ng-submit="addTodo()">
        			<input type="text" ng-model="todoText"  size="30" placeholder="add new todo here">
        			<input class="btn-primary" type="submit" value="add">
	      		</form>
    		</div>
    		
    		// 定义了结构上的方法
			function TodoCtrl($scope) {
  				$scope.todos = [
    				{text:'learn angular', done:true},
    				{text:'build an angular app', done:false}];
 
  				$scope.addTodo = function() {
    				$scope.todos.push({text:$scope.todoText, done:false});
				    $scope.todoText = '';
				  };
 
  				$scope.remaining = function() {
   					var count = 0;
    				angular.forEach($scope.todos, function(todo) {
      				count += todo.done ? 0 : 1;
    			});
    			return count;
  			};
 
  			$scope.archive = function() {
    			var oldTodos = $scope.todos;
    				$scope.todos = [];
    			angular.forEach(oldTodos, function(todo) {
     				if (!todo.done) $scope.todos.push(todo);
    			});
  			};
		}
	
	

再看一段代码：React

		/** @jsx React.DOM */
		var TodoList = React.createClass({
  			render: function() {
    			var createItem = function(itemText) {
      				return <li>{itemText}</li>;
    			};
    			return <ul>{this.props.items.map(createItem)}</ul>;
  			}
		});
		var TodoApp = React.createClass({
  			getInitialState: function() {
    			return {items: [], text: ''};
  			},
  			onChange: function(e) {
    			this.setState({text: e.target.value});
  			},
  			handleSubmit: function(e) {
    			e.preventDefault();
    			var nextItems = this.state.items.concat([this.state.text]);
    			var nextText = '';
    			this.setState({items: nextItems, text: nextText});
  			},
  			render: function() {
    			return (
      				<div>
        				<h3>TODO</h3>
        				<TodoList items={this.state.items} />
        				<form onSubmit={this.handleSubmit}>
          					<input onChange={this.onChange} value={this.state.text} />
          					<button>{'Add #' + (this.state.items.length + 1)}</button>
        				</form>
      				</div>
    		);
  		}
	});
	React.renderComponent(<TodoApp />, mountNode);

最后再看一段更神奇的代码：DarkDom
	
		<x-postlist source-selector=".src-postlist">
  			<x-hd>Posts</x-hd>
  			<x-ft>Total:<span>3</span></x-ft>
  			<x-item>
    			<x-title>Post A</x-title>
    			<x-tag>js</x-tag>
    			<x-tag>css</x-tag>
    			<p>Paragraph A
      				<x-folder mode="fold">
        				<x-caption>Spoiler</x-caption>
        				<em>Paragraph B</em>
      				</x-folder>
    			</p>
    			<p>Paragraph C</p>
  			</x-item>
  			<x-item source-selector=".box2.src-item"></x-item>
  			<script type="text/darkscript">
  				console.info("DarkDOMReady", 
    				this, this.isMountedDarkDOM);
  			</script>
		</x-postlist>
		
		// js代码太长了，就不贴了











