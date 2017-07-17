---
layout: post
title: "performance in javascript"
modified: 2014-08-18 17:07:27 +0800
tags: [javascript,navigator]
category: blog
---

在web项目中，页面的展现速度是大家一直关注的一个要点。在有关页面展现速度的统计中，我们通常会统计这样一些时间点：

- 请求响应时间
- dom加载时间
- 第一屏显示时间
- js阻塞UI显示的时间
- 等等

根据这些时间点，可以得知页面的性能瓶颈；还可以分析速度对用户体验的影响。

本着事无巨细的态度，在开发web项目时，通常会在前后端对时间做出很多统计。比如想要统计页面响应的时间，那么就要统计从http请求进入到服务器，再到服务器完成http请求的这一段时间。对于服务器来说，所消耗的时间无非就是IO加上计算的时间，但是对于用户来讲，这只是他所触发的一次浏览的整个时间链中的一小段。很多个这种小段的时间，最终组成了一条完整的时间线，也就是用户所等待的时间。

在新版本的*chrome*浏览器里，提供了一个`performace`的全局对象，通过这个对象，可以获取到当前所打开的页面性能的各项指数，可以现在控制台中执行以下`console.log(performace)`试试看，返回的结果将是：

	{
		memory : MemoryInfo,
		navigation : PerformancsNavigation,
		onwebkitresourcetimingbufferfull : null,
		timing : PerformanceTiming
	}
	
可以看出来，这个全局对象中包括了一些页面性能相关的数据。

先来说`memory`，故名思议，这个肯定是跟内存相关的数据，展开`memory`看看：

	{
		jsHeapSizeLimit: 793000000
		totalJSHeapSize: 10600000
		usedJSHeapSize: 10000000
	}
	
根据参数名推断，`jsHeapSizeLimit`应该是当前浏览器中，允许js所占用的最大内存；`totalJSHeapSize`应该是当前页面中，js所占用的内存，`usedJSHeapSize`应该是页面所用到的内存。
根据内存的使用情况，我们可以推算出当前浏览器中的js运行环境是怎么样的，js运行是否过多的占用了内存，所以导致体验不佳，这一点在移动端可能会很有用。

再来看`navigation`这个对象，这个对象比较简单，只有两条数据：
	
	{
		redirectCount ： 0，
		type : 1
	}
	
其中`redirectCount`表示的为是否有`redirect`跳转；`type`表示的是页面载入的类型，`type`的值有4种，分别是：
- 0 : 正常访问
- 1 : reload
- 2 : 后退访问
- 255 : 这是一个不会在任何浏览器上出现的值，它表示的是完全的浏览器端缓存

最后，再来看最复杂的`timing`对象，这个对象非常之长，完整的标记了浏览器中的各个时间点：
	
	{
		connectEnd: 1408436118939
		connectStart: 1408436118939
		domComplete: 1408436119959
		domContentLoadedEventEnd: 1408436119863
		domContentLoadedEventStart: 1408436119863
		domInteractive: 1408436119863
		domLoading: 1408436119012
		domainLookupEnd: 1408436118939
		domainLookupStart: 1408436118939
		fetchStart: 1408436118939
		loadEventEnd: 1408436119959
		loadEventStart: 1408436119959
		navigationStart: 1408436118939
		redirectEnd: 0
		redirectStart: 0
		requestStart: 1408436118941
		responseEnd: 1408436119395
		responseStart: 1408436118995
		secureConnectionStart: 0
		unloadEventEnd: 1408436118997
		unloadEventStart: 1408436118997 
	}

在w3c关于`timing`的[页面](http://www.w3.org/TR/navigation-timing/)中，有这样一张图:

![image](http://www.w3.org/TR/navigation-timing/timing-overview.png)

这张图完整的表示了浏览器中发起一个http请求的各个步骤，期间对应的时间点，都在`timing`对象中有标记。其中，有几个值比较特殊，`redirectStart`,`redirectEnd`和`secureConnectionStart`，从上面的代码里可以看出，这几个值为0，而其他值都是时间戳，那么这几个值表示的是什么？又为什么为0呢？

先看`redirectStart`和`redirectEnd`，这两个值表示的是同源情况下，做`redirect`跳转的时间。

再看`secureConnectionStart`这个值，这个值在通常情况下，是未定义的，所以直接会返回0，在被定义并且是通过`https`协议的情况下，这个值返回的是`https`协议建立前的时间戳。


通过以上这些`performance`中的选项，我们可以更方便的获取到页面性能的各项参数，尤其是`timing`这个参数，使得监控页面各项速度参数更加精确，应该多加以利用。





