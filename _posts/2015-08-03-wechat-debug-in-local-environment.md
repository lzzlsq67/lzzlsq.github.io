---
layout: post
title:  "搭建微信公众号的本地测试环境"
keywords: "wechat debug"
description: "搭建一个微信公众号的本地测试环境，方便和微信联调，不用重复向测试环境提交代码进行测试"
category: wechat
tags: wechat
---
一般情况下，我们在开发微信公众平台的时候，流程一般是这样的

1. 本地编码
2. 提交测试服务器测试，测试环境一般绑定一个测试号
3. 测试通过后发布生产环境代码

但是这样会非常不方便调试代码，还好有法宝，可以在本地模拟一个测试环境，把微信转发给测试环境的的请求全部都转发给本地
，其实还可以换种思路，只要将本地的ip地址暴露在公网上，就可以和微信服务器进行交互了。


首先你需要一个把本地ip地址暴露在公网上的软件，目前主流而且免费的一般用ngrok
###搭建本地环境的步骤如下:

####一、登录[链接名称](https://ngrok.com/)（可能需要科学上网）注册用户，会得到一个token
![登录](http://i1.tietuku.com/989660f79f923622.png)
####二、下载ngrok，解压并运行ngrok.exe，认证token
![安装配置](http://i1.tietuku.com/f7dafd60e2e92c7a.jpg)

####三、开启指定端口的http服务
![开启服务](http://i1.tietuku.com/985452824f404d3c.png)

####四、把本地端口换成ngrok的域名
![配置接口信息](http://i1.tietuku.com/38cce9a3e7c174a4.jpg)

####五、在微信公众号后台把接口配置信息改为刚才替换掉的ngrok域名
![配置接口信息](http://i1.tietuku.com/98c196a9cd0eaab6.png)

这样一个微信公众平台本地测试环境就搭好了，用户和微信交互时微信服务器发送的消息都将会转发到本地Tomcat服务器，但是还有一点不爽这个域名是随机分配到，就是说如果你按下`ctrl+C`结束了程序，再次启动的话，域名还是会变掉，想得到固定的二级域名，就必须要花钱了
一个月5美元。

