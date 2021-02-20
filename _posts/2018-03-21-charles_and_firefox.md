---
layout: post
title: "解决 firefox 不能使用 charles 的问题"
modified: 2018-03-21 18:01:05 +0800
tags: [charles, firefox, exp]
category: blog
---

基本不用 firefox，今天有同事报了个 firefox 上的问题，下了个 firefox。
想着用 charles 抓下包看下，结果发现并不能抓到任何包。感觉有点儿奇怪，试了一下其他的浏览器，safari 和 chrome 都可以，唯独
firefox 不行。

于是 google 了一下，发现了一个 charles 官方推出的 firefox 插件 [charles-proxy](https://addons.mozilla.org/zh-CN/firefox/addon/charles-proxy/)，但很遗憾，新版本的 firefox 已经不能用了。

在 firefox 的 preferences 里看了下 proxy 的设置，感觉可能是这里的问题，于是就尝试着改了一下 proxy 的设置，改成了`Menu
proxy configuration`，如图：

![firefox proxy configuration]('./firefox-proxy.png')

图中的 `127.0.0.1` 就是当前本机的地址，后面的 `8888`和`8889`是我本机 charles 占用的端口号。

配置完之后，发现还是不行，又想了一下，感觉可能是 https 的问题。于是又在 preferences 里搜了一下
certificates，发现果然可以设置，点击一下`View Certificates` :

![view certificates]('./firefox-certificates.png')

可以看到一堆证书的设置，还有个 `Import` 的按钮，打开本机的 keychain accesss，可以看到之前安装过的 charles
的证书，把这个证书导出来一份，然后再导入到 firefox 中就可以了。

导入完证书，刷了一下网页，果然可以捕获到正确的请求了。

解决了问题，想了一下原理。

charles 本身应该是通过在本地起一个proxy server，然后把系统中的请求都转发到本地的 proxy server 来进行抓包的。firefox
之所以不行，可能是因为 firefox 默认绕过了系统中设置的 proxy，所以需要手动设置一下 proxy 才可以。
