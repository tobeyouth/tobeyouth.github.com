---
layout: post
title: "react benchmarks"
modified: 2017-05-16 15:12:35 +0800
tags: [react, benchmarks]
---

简单的说一下如何在 react 开发中，进行 benchmark，并且会简单的说一些可以提升性能的小技巧。

## Performance

既然想要进行 benchmark，那么就需要一个指标来衡量。react 官方已经提供了这样的工具供开发人员使用。

官方的文档可以参考这里[Performance Tools](https://facebook.github.io/react/docs/perf.html)，在这里我再做个简单的介绍。

#### 引入

```
import Perf from 'react-addons-perf'; 
```

就可以将`Performance Tool`引入到当前的 react app 中使用了。

 
#### 使用

`Performance Tool`的使用十分方便，在上一步引入的 `Perf` 本身是个对象，可以直接调用其方法，来进行性能测试。

##### 获取指标的方法

- Perf.start(): 开始记录
- Perf.stop(): 停止记录
- Perf.getLastMeasurements(): 获取最后一条记录

##### 输出指标的方法

以下这些方法，如果不传入参数的话，都会调用`Perf.getLastMeasurements()`方法，获取最新的 measurements

- Perf.printInclusive(measurements): 输出组件 lifecirlifecyclecle 全部执行完的时间
- Perf.printExclusive(measurements): 只输出组件渲染所用的时间，不包括 `componentWillMount`和`componentDidMount`的执行时间
- Perf.printWasted(measurements): `Wasted`的是指页面中的 dom 实际并没有发生变化，但是组件仍然被渲染的操作，`printWasted`就是输出这些`无意义`操作所用的时间
- Perf.printOperations(measurements): 详细输出每一个组件dom 操作(包括 innerHTML 和 remove)所用的时间和相关信息

##### 一个实例截图

![一个截图](/pics/react-benchmark-pic-1.png)


## 一些优化技巧

### functional

举个简单的例子，假设我们需要渲染一个并没有交互的组件，例如一句话，那么这个组件其实也不存在 `lifecycle`，那么可以直接使用函数式的方法输出这个组件

我做了个简单的 [demo](https://github.com/tobeyouth/react-benchmark)，可以 clone 下来自己看下



```
// 使用 component
class Text extends React.Component {
	render () {
		return (
			<div>{ this.props.text }</div>
		)
	}
}
export default Text


// 使用匿名函数
export default (text) => {
	return (
		<div>{ text }</div>
	)
}

```

看下 `Performance Tool`输出的结果

![一个截图](/pics/react-benchmark-pic-2.png)

可以明显的看到`Benchmark > FunctionWrap`的总时间要小于`Benchmark > ComponentWrap`和`Benchmark > PureComponentWrap`所用的时间

这是因为使用匿名函数，省掉了 `lifecycle` 的一系列函数调用的时间，`Benchmark > PureCompnent`耗时最长是因为`React.PureComponent`会在`shouldComponentUpdate`中默认进行`shallowEqual`的操作，所以初始化渲染会比较慢。

### 使用 PureComponent

`React.PureComponent`和`React.Component`的区别就在于`PureComponent`会默认带一个`shouldComponentUpdate`的方法，通过`shallowEqual`对比当前的 `component` 是否需要进行重新渲染。

有了这样的一个简单的判断，在不手动写`shouldComponentUpdate`方法时，也可以获得一定的性能提升。

具体的测试，同样可以看这个[demo](https://github.com/tobeyouth/react-benchmark)，里面有相关的测试。

#### 一个隐形的坑

通常在可交互的组件上，我们会绑定一些事件，例如下面的例子

```
class User extends React.Component {
  render () {
    console.log('render user')
    return (
      <div className='user'>
        <p>is a component</p>
        <p>name: { this.props.data.name }</p>
        <p>id: { this.props.data.id }</p>
        <div className='buttons'>
          <button onClick={ this.props.onClick }>Change Data</button>
        </div>
      </div>
    )
  }
}

class PureUser extends React.PureComponent {

  render () {
    console.log('render pure user')
    return (
      <div className='user'>
        <p>is a pure component</p>
        <p>name: { this.props.data.name }</p>
        <p>id: { this.props.data.id }</p>
        <div className='buttons'>
          <button onClick={ this.props.onClick }>Change Data</button>
        </div>
      </div>
    )
  }
}

class Anonymous extends React.Component {

  constructor (props) {
    super(props)
    this.state = {
      data: Foo
    }
  }

  render () {

    return (
      <div>
        <div className='wrap'>
          { this.renderUser() }
          { this.renderPureUser() }
        </div>        
      </div>
    )
  }

  changeData () {
    Pref.start()
    this.setState({
      data: Foo
    })
  }

  renderUser () {
    return <User data={ this.state.data } 
            onClick={ this.changeData.bind(this) } />
  }
  
  renderPureUser () {
    return <PureUser data={ this.state.data } 
            onClick={ this.changeData.bind(this) } />
  }
}

```

在这个例子中`<Anonymous />`会将`state.data`传递给自组件，同时会传递一个 `onClick` 的事件回调给子组件。

在运行这个例子时，我们会发现即使我们使用`React.PureComponent`，并且并没有实际改变 `state.data`的值，但是 `<PureUser />`这个组件还是会跟`<User />` 组件一样，会重复被渲染。


究其原因，在于`onClick`这里使用了`.bind`方法，将`changeData`绑定到了当前的作用域内，但是`.bind`方法返回的是个**匿名函数**，所以事实上每次传入到子组件内的`props`都是不同的，`PureComponent`也会被重新渲染。

为了避免这种情况，可以将`.bind`方法前置，改在`constructor`中预先绑定，这样`onClick`将指向一个固定的函数，例子:

```
class PublicClassFields extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      data: Foo
    }
    this.changeData = this.changeData.bind(this)
  }
  ...
  ...
}
```

这样的话，`PureUser`在执行 `changeData`后就不会被重新渲染了。


## ps

后续还会有一些关于 react 性能相关的内容补充进来，同时也会不断的更新这个 [repo](https://github.com/tobeyouth/react-benchmark)中的实例。


