---
layout: post
title: "wechat develop"
modified: 2017-07-31 12:10:06 +0800
tags: [wechat]
---

总结一些微信相关开发踩过的坑

## 微信网页授权

### 相关文档地址: [https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)

### 后台设置
- 在使用网页授权之前, 需要把要授权的域名，在微信公众号的后台设置一下。具体路径是：`开发 - 接口权限 - 网页服务 - 网页帐号 - 网页授权获取用户基本信息`

### 两种授权方式
网页授权有两种授权的方式，分别是`snsapi_base`和`snsapi_userinfo`。
其中
`snsnapi_base`为*静默授权*，翻译成人类能理解的话，就是不会弹让用户点击*授权*按钮的页面。但是相应的，这个授权方式能获得的信息也很有限，只拿到用户的`openid`，无法获取用户的个人信息。
如果需要用户注册/登录的话，需要调用`snsapi_userinfo` 这个授权方式，才可以取到用户的个人信息。

### openid 和 UnionID

`openid` 是用户和公众号之前唯一的识别标志，同一个用户在不同公众号上的 `openid` 是不同的。
为了在多个公众号之间可以识别唯一的用户，需要使用 `UnionID`，这个是用户的唯一标识。

### 网页授权的步骤

#### snsapi_base

1. 用户访问页面
2. 跳转到授权页面*https://open.weixin.qq.com/connect/oauth2/authorize*
3. 微信网关判断其中的 `redirect_url`是否与后台设置的一致
4. 跳转回 `redirect_url` 中设置的地址，同时带上 `code` 参数
5. 开发者通过 `code` 访问`https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code`, 获取到用户的 `access_token`

#### snsapi_userinfo
1. 用户访问页面
2. 弹出授权界面
3. 用户点击*授权*, 跳转到授权页面*https://open.weixin.qq.com/connect/oauth2/authorize*
4. 微信网关判断其中的 `redirect_url`是否与后台设置的一致
5. 跳转回 `redirect_url` 中设置的地址，同时带上 `code` 参数
6. 开发者通过 `code` 访问`https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code`来获取 `access_token` 和 `openid`
7. 开发者成功获取到`openid` 和`access_token` 之后，通过`https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN`，获取用户的相关信息




