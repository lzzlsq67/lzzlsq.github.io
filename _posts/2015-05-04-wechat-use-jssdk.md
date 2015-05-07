---
layout: post
title:  "微信JS-SDK的使用Java版"
keywords: "wechat"
description: "使用最新的JS-SDK进行微信接口的开发"
category: wechat
tags: wechat java
---

微信JS-SDK是微信公众平台面向网页开发者提供的基于微信内的网页开发工具包。
通过使用微信JS-SDK，网页开发者可借助微信高效地使用拍照、选图、语音、位置等手机系统的能力，同时可以直接使用微信分享、扫一扫、卡券、支付等微信特有的能力，为微信用户提供更优质的网页体验。
##1.绑定域名
  先登录微信公众平台进入`公众号设置`的`功能设置`里填写`JS接口安全域名`。需要注意的是安全域名需要的是`ICP备案`的一级域名，二级域名无效。


##2.引入JS文件

在需要调用JS接口的页面引入如下JS文件，（支持https）：http://res.wx.qq.com/open/js/jweixin-1.0.0.js

备注：支持使用 AMD/CMD 标准模块加载方法加载
##3.通过config接口注入权限验证配置

所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次）

```javascript
	wx.config({
	    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
	    appId: '', // 必填，公众号的唯一标识
	    timestamp: , // 必填，生成签名的时间戳
	    nonceStr: '', // 必填，生成签名的随机串
	    signature: '',// 必填，签名，见附录1
	    jsApiList: [] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
	});

