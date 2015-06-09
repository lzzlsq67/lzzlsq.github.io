---
layout: post
title:  "Java转换时间格式为2015-06-09T09:13:44.000Z的字符串"
keywords: "java UTC"
description: "将怪异的时间格式转换为java util date"
category: java
tags: java
---
##最近业务json中出现时间格式为 `2015-06-09T09:13:44.000Z`这种格式的时间，在java中处理需要将时间字符串转换为date

T说明后面接的是时间,Z通用标准时间的标识 
其它的时区显示的时间与通用协调时间不同，因此例如你能使用太平洋标准时间2015-06-09T06:00:00:000-8:00来显示2015年6月9日的早上6：00(它比UTC时间滞后8小时)。 

Java Date使用`UTC时间`，如Tue May 09 17:00:00 CST 2015，CST表示China Standard Time UT+8:00

转换代码为：

```java
	String pecificTime = "2015-06-09T17:00:00Z";
	@Test
	public void testParsePMTime() throws Exception {
		SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
		Date time = df.parse(pecificTime);
		System.out.println(time);
	}
	
```

如果时间格式是String pecificTime = "2015-06-09T17:00:00+0800";

```java
	
	String pecificTime = "2015-06-09T17:00:00+0800";
	@Test
	public void testParseTPTime() throws Exception {
		
		SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ssZ");
		Date time = df.parse(pecificTime);
		System.out.println(time);
	}
```

