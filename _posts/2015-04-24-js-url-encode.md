---
layout: post
title:  "JS URL传参中文乱码"
keywords: "js 乱码"
description: "JS URL 传参中文乱码的解决"
category: javascript
tags: js 乱码
---
在接口开发中，中间件和APP交互的时候，遇到URL传参的问题，发现到后台都是乱码。发现URL应该需要编码
在js中对中文参数进行编码，在服务器进行解码

###1.在js中对中文参数进行两次编码
在js中对中文编码的方法这里使用encodeURI
	
```javascript
	var username=$("username").val();
	encodeURI(encodeURI(username))

```

###2.在服务器端，将传来的参数解码为UTF-8编码的中文

```java
String username = request.getParameter("username");
username = java.net.URLDecoder.decode(username,"UTF-8"); 
```
##PS:javaScript中的编码方法 
**encodeURI()** 方法：**

	把URI字符串采用UTF-8编码格式转化成escape格式的字符串。不会被此方法编码的字符：! @ # $& * ( ) = : / ; ? + '

**escape()** 方法：** 

	采用ISO Latin字符集对指定的字符串进行编码。所有的空格符、标点符号、特殊字符以及其他非ASCII字符都将被转化成
	%xx格式的字符编码（xx等于该字符在字符集表里面的编码的16进制数字）。比如，空格符对应的编码是%20。unescape方法与此相反。
	不会被此方法编码的字符： @ * / + 

**encodeURIComponent() 方法：**

	把URI字符串采用UTF-8编码格式转化成escape格式的字符串。与encodeURI()相比，这个方法将对更多的字符进行编码，
	比如 / 等字符。所以如果字符串里面包含了URI的几个部分的话，不能用这个方法来进行编码，否则 / 字符被编码之后URL将显示错误。
	不会被此方法编码的字符：! * ( ) 
