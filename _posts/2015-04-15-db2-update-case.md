---
layout: post
title:  "查询oracle中clob中的xml的节点数据"
keywords: "oracle"
description: "使用oracle的xmlType函数得到xml文档里的节点的值"
category: oracle
tags: oracle
---
##查询oracle中clob中的xml的节点数据


###查询xmltype字段里面的内容
　　得到id=1的value变脸的值

```sql
　　select i.xmldoc.extract(''//name/a[@id=1]/@value'').getStringVal() as ennames, id from abc i
```
　　得到a节点的值
　　select id, i.xmldoc.extract(''//name/a/text()'').getStringVal() as truename from abc i
　　得到节点id属性的值
　　Select hd.Data_t.extract(''/root/name/@id'').getStringVal() As Name FROM sehr_house_data hd

 
