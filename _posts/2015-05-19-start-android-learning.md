---
layout: post
title:  "安卓项目INSTALL_FAILED_UID_CHANGED的解决"
keywords: "android"
description: "安卓项目在模拟器上可以运行，但是在手机上却不能运行的问题"
category: android
tags: android java
---

从今天起，开始打算学习android了，结果按照教程，一步一步来，到最后helloworld已经成功了，在模拟器上也运行成功了，但是却不能在自己的手机上运行，
`INSTALL_FAILED_UID_CHANGED`错误,在overstackflow上的大神说的是android API版本不对，然后我更改了API版本之后依旧不能运行，查了好长时间资料才发现
是手机中缓存的原因。
###解决方法
1. 删除/data/app/<自己项目的包名>文件夹下的apk包
2. 删除/system/app/<自己项目的包名>文件夹下的apk包
3. 删除/data/data/<自己项目的包名> 文件夹下相关信息
