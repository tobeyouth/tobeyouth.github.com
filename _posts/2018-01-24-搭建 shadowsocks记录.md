---
layout: post
title: "搭建 shadowsocks 记录"
modified: 2018-01-24 11:05:01 +0800
tags: [ exp, shadowsockss, vultr]
category: blog
---

### 前期准备

因为最近使用的第三方 vpn 快到期了，所以打算自己搭个 shadowsocks 用。
找了一些国外的 vps 提供方，最后发现 `vultr` 算是最实惠的，有个 **2.5美金/月** ，相较于 linode 的 **5美金/月**，虽然配置差了一些，但是简单作为梯子用，还是够用了。

搭建的过程非常简单，首先就是注册 `vultr`，然后充值，先充了个10美金，另外，vultr 比 linode 更方便的一点是不需要国际信用卡，可以直接用支付宝付款。

付完款，可以直接挑选节点和主机配置等，离国内比较近的日本和新加坡节点都比较贵，为了追求便宜，所以随便选了个美国节点。

选完节点和配置，接下来是选择系统，因为只用过 centos 和 ubuntu，又想省得折腾，所以直接选了 ubuntu。


### 登录主机

新建完主机，用网站上提供的账号和 ip 先 ssh 到主机上
```
ssh root@主机 ip
```
这时会要求输入密码，可以直接把网站上提供的主机密码 copy 过去。密码验证通过，会进入到主机的目录中。

为了避免以后每次都要输入 ip 和密码，可以把主机 ip 先在本机上加个 host:

```
sudo vi /private/etc/hosts
```
然后用

```
ip 随便起个名
```
的格式给 ip 起个方便好记的别名。

接下来可以参考[这篇](https://tobeyouth.github.io/blog/2014/07/14/ssh-login/)在你的主机上添加个本机的 ssh-key，这样以后就不用输密码了。

### 正式搭建 shadowsocks server

这里我使用了这个项目[shadowsocks_install](https://github.com/teddysun/shadowsocks_install)，直接一行命令搞定

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```
其中会有个步骤需要选择用什么版本的 shadowsocks，我选择的是 

```
shadowsocks-libev
```
看起来这个版本的 github 是比较活跃的，所以也推荐大家使用这个版本。

之后就是需要输入密码和端口号了

```
Please enter password for shadowsocks-libev
(default password: teddysun.com):

password = teddysun.com

Please enter a port for shadowsocks-libev [1-65535]
(default port: 8989):

port = 8989

Press any key to start...or Press Ctrl+C to cancel
```

全部搞定后，你的 shadowsockss 就搭建成功了。


### 客户端

windows, mac, android, ios 上都有相应的开源客户端，可以自行下载安装

- [window](https://github.com/shadowsocks/shadowsocks-windows)
- [mac](https://github.com/shadowsocks/ShadowsocksX-NG)
- [android](https://github.com/shadowsocks/shadowsocks-android)
- [ios](https://github.com/shadowsocks/shadowsocks-iOS)

因为 ios 的相关客户端已经在 github 上被删除，所以建议使用其他客户端代替，我使用的是一个叫`ssrconnectPro`的免费 app。
当然，使用收费的 app 效果可能会更好，例如 `surge` 什么的。

### 使用客户端

- 首先，要选择链接类型为 `shadowsocks` ，有的客户端中简称为 `ss`。
- 接下来填入你的主机的 ip 和前面设置好的端口号和密码。
- 保存、链接

等成功链接之后，就可以正常访问 google 和 facebook 等并不存在的网站啦！

### 其他
一些相关资源和教程的地址
- [shadowsocks.be](https://shadowsocks.be/)
- [shadowsocks in github](https://github.com/shadowsocks)
- [轻松在 VPS 搭建 Shadowsocks 翻墙](https://www.diycode.cc/topics/738)