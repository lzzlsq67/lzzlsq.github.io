---
layout: post
title:  "Excel剔除一组数据"
keywords: "Excel"
description: "Excel去除重复数据，或者剔除一组数据"
category: office
tags: excel
---
##Excel剔除重复数据
假如我们现在有两组数据，其中一组数据为主数据A，另一组为子数据B，要将B从A中剔除形成一组新的数据C
**在下图中**

	A列为主列，B列为要从A列中去除的数据，C列为去除重复的新列

![去除Excel重复数据](http://img.hb.aicdn.com/40b027339c6aa850ceaaca343070c4c2a24518f7fc7-WnzkHl_fw658)
在C列第一个单元格内输入`=IF(COUNTIF($B$2:$B$99,$A2)>=1,"",$A2)`，当然，如果有更多的数据，99可以更
改为更大的数据比如`99999`
然后选中C列，按住键盘`ctrl+G` 选择空值
![去除一列数据中多余的空值](http://img.hb.aicdn.com/5150d160eeeb80a4074c2820981a4e8d79dbc1d63c1d-xBT8IK_fw658)