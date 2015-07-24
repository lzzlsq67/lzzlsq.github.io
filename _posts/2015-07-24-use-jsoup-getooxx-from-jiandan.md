---
layout: post
title:  "Jsoup使用小结"
keywords: "jsoup"
description: "使用Jsoup解析和遍历一个Html文档，抽取和修改数据"
category: java
tags: jsoup java
---
jsoup 是一款Java 的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery
的操作方法来取出和操作数据。
```
1. 从一个URL，文件或字符串中解析HTML；
2. 使用DOM或CSS选择器来查找、取出数据；
3. 可操作HTML元素、属性、文本；
```

##使用入门
###一、加载文档
####1.文档的对象模型
文档由多个Elements和TextNodes组成
其继承结构如下：Document继承Element继承Node. TextNode继承 Node.
一个Element包含一个子节点集合，并拥有一个父Element。他们还提供了一个唯一的子元素过滤列表。
####2.解析一个html片段

使用Jsoup.parseBodyFragment(String html)方法可以解析一个html片段.

```java
	String html = "<div><p>Lorem ipsum.</p>";
	Document doc = Jsoup.parseBodyFragment(html);
	Element body = doc.body();
```

parseBodyFragment 方法创建一个空壳的文档，并插入解析过的HTML到body元素中。假如你使用正常的`Jsoup.parse(String html)`方法，
通常你也可以得到相同的结果，但是明确将用户输入作为 body片段处理，以确保用户所提供的任何糟糕的HTML都将被解析成body元素。
`Document.body()` 方法能够取得文档body元素的所有子元素，与 `doc.getElementsByTag("body")`相同。
####3.从URL加载DOM对象
使用 Jsoup.connect(String url)方法可以从一个网站加载和获取文档

```java
	Document doc = Jsoup.connect("http://zeusjava.com/").get();
	String title = doc.title();
```

`connect(String url)`方法创建一个新的`Connection`, 和 `get()`取得和解析一个HTML文件。如果从该URL获取HTML时发生错误，便会抛出
`IOException`，应适当处理。

`Connection` 接口还提供一个方法链来解决特殊请求，具体如下：

```java
	Document doc = Jsoup.connect("http://zeusjava.com")
	  .data("query", "Java")
	  .userAgent("Mozilla")
	  .cookie("auth", "token")
	  .timeout(3000)
	  .post();
```
这个方法只支持Web URLs (http和https 协议); 假如你需要从一个文件加载，可以使用 parse(File in, String charsetName) 代替。

####4.从文件加载DOM对象
可以使用静态 Jsoup.parse(File in, String charsetName, String baseUri) 来从文件中加载DOM对象

```java
	File input = new File("/tmp/input.html");
	Document doc = Jsoup.parse(input, "UTF-8", "http://zeusjava.com/");
```
`parse(File in, String charsetName, String baseUri)`这个方法用来加载和解析一个`HTML文件`。如在加载文件的时候发生错误，将抛出
`IOException`，应作适当处理。
`baseUri`参数用于解决文件中URLs是`相对路径`的问题。如果不需要可以传入一个空的字符串。
另外还有一个方法`parse(File in, String charsetName)` ，它使用文件的路径做为 `baseUri`。 这个方法适用于如果被解析文件位于网站的本地
文件系统，且相关链接也指向该文件系统。

###二、提取数据
####1.使用DOM方法遍历一个文档
将HTML解析成一个Document之后，就可以使用类似于DOM的方法进行操作：

```java
	Document doc = Jsoup.connect("http://zeusjava.com/").get();
	Element content = doc.getElementById("content");
	Elements links = content.getElementsByTag("a");
	for (Element link : links) {
	  String linkHref = link.attr("href");
	  String linkText = link.text();
	}
```

Elements这个对象提供了一系列类似于DOM的方法来查找元素，抽取并处理其中的数据。具体如下：

查找元素

	getElementById(String id)
	getElementsByTag(String tag)
	getElementsByClass(String className)
	getElementsByAttribute(String key) (and related methods)
	Element siblings: siblingElements(), firstElementSibling(), lastElementSibling(); nextElementSibling(), previousElementSibling()
	Graph: parent(), children(), child(int index)

元素数据

	attr(String key)获取属性
	attr(String key, String value)设置属性
	attributes()获取所有属性
	id(), className() and classNames()
	text()获取文本内容
	text(String value) 设置文本内容
	html()获取元素内HTMLhtml
	(String value)设置元素内的HTML内容
	outerHtml()获取元素外HTML内容
	data()获取数据内容（例如：script和style标签)
	tag() and tagName()

操作HTML和文本

	append(String html), prepend(String html)
	appendText(String text), prependText(String text)
	appendElement(String tagName), prependElement(String tagName)
	html(String value)

####2.使用选择器语法查找元素
可以使用`Element.select(String selector)`和 `Elements.select(String selector)` 方法实现类似于CSS或jQuery的语法来查找和操作元素。

```java
	Document doc = Jsoup.connect("http://zeusjava.com/").get();
	Elements links = doc.select("a[href]"); //带有href属性的a元素
	Elements pngs = doc.select("img[src$=.png]");
	  //扩展名为.png的图片
	Element masthead = doc.select("div.masthead").first();
	  //class等于masthead的div标签
	Elements resultLinks = doc.select("h3.r > a"); //在h3元素之后的a元素
```

说明
jsoup elements对象支持类似于CSS (或jquery)的选择器语法，来实现非常强大和灵活的查找功能。.
这个select 方法在Document, Element,或Elements对象中都可以使用。且是上下文相关的，因此可实现指定元素的过滤，或者链式选择访问。
Select方法将返回一个Elements集合，并提供一组方法来抽取和处理结果。
####Selector选择器概述
```
	tagname: 通过标签查找元素，比如：a
	ns|tag: 通过标签在命名空间查找元素，比如：可以用 fb|name 语法来查找 <fb:name> 元素
	#id: 通过ID查找元素，比如：#logo
	.class: 通过class名称查找元素，比如：.masthead
	[attribute]: 利用属性查找元素，比如：[href]
	[^attr]: 利用属性名前缀来查找元素，比如：可以用[^data-] 来查找带有HTML5 Dataset属性的元素
	[attr=value]: 利用属性值来查找元素，比如：[width=500]
	[attr^=value], [attr$=value], [attr*=value]: 利用匹配属性值开头、结尾或包含属性值来查找元素，比如：[href*=/path/]
	[attr~=regex]: 利用属性值匹配正则表达式来查找元素，比如： img[src~=(?i)\.(png|jpe?g)]
	*: 这个符号将匹配所有元素
	Selector选择器组合使用
	el#id: 元素+ID，比如： div#logo
	el.class: 元素+class，比如： div.masthead
	el[attr]: 元素+class，比如： a[href]
	任意组合，比如：a[href].highlight
	ancestor child: 查找某个元素下子元素，比如：可以用.body p 查找在"body"元素下的所有 p元素
	parent > child: 查找某个父元素下的直接子元素，比如：可以用div.content > p 查找 p 元素，也可以用body > * 查找body标签下所有直接子元素
	siblingA + siblingB: 查找在A元素之前第一个同级元素B，比如：div.head + div
	siblingA ~ siblingX: 查找A元素之前的同级X元素，比如：h1 ~ p
	el, el, el:多个选择器组合，查找匹配任一选择器的唯一元素，例如：div.masthead, div.logo
```
####伪选择器selectors
```
	:lt(n): 查找哪些元素的同级索引值（它的位置在DOM树中是相对于它的父节点）小于n，比如：td:lt(3) 表示小于三列的元素
	:gt(n):查找哪些元素的同级索引值大于n，比如： div p:gt(2)表示哪些div中有包含2个以上的p元素
	:eq(n): 查找哪些元素的同级索引值与n相等，比如：form input:eq(1)表示包含一个input标签的Form元素
	:has(seletor): 查找匹配选择器包含元素的元素，比如：div:has(p)表示哪些div包含了p元素
	:not(selector): 查找与选择器不匹配的元素，比如： div:not(.logo) 表示不包含 class=logo 元素的所有 div 列表
	:contains(text): 查找包含给定文本的元素，搜索不区分大不写，比如： p:contains(jsoup)
	:containsOwn(text): 查找直接包含给定文本的元素
	:matches(regex): 查找哪些元素的文本匹配指定的正则表达式，比如：div:matches((?i)login)
	:matchesOwn(regex): 查找自身包含文本匹配指定正则表达式的元素
	注意：上述伪选择器索引是从0开始的，也就是说第一个元素索引值为0，第二个元素index为1等
```


####2.使用选择器语法查找元素
从元素抽取属性，文本和HTML
要取得一个属性的值，可以使用Node.attr(String key) 方法
对于一个元素中的文本，可以使用Element.text()方法
对于要取得元素或属性中的HTML内容，可以使用Element.html(), 或 Node.outerHtml()方法

```java
	String html = "<p>An <a href='http://zeusjava.com/'><b>example</b></a> link.</p>";
	Document doc = Jsoup.parse(html);//解析HTML字符串返回一个Document实现
	Element link = doc.select("a").first();//查找第一个a元素
	String text = doc.body().text(); // "An example link"//取得字符串中的文本
	String linkHref = link.attr("href"); // "http://zeusjava.com/"//取得链接地址
	String linkText = link.text(); // "example""//取得链接地址中的文本
	String linkOuterH = link.outerHtml();
	// "<a href="http://zeusjava.com"><b>example</b></a>"
	String linkInnerH = link.html(); // "<b>example</b>"//取得链接内的html内容
```
上述方法是元素数据访问的核心办法。此外还其它一些方法可以使用：
Element.id()
Element.tagName()
Element.className()
Element.hasClass(String className)
这些访问器方法都有相应的setter方法来更改数据.
####3.处理URLs
包含相对URLs路径的HTML文档，需要将这些相对路径转换成绝对路径的URLs.
在你解析文档时确保有指定base URI，然后使用 abs: 属性前缀来取得包含base URI的绝对路径

```java
	Document doc = Jsoup.connect("http://zeusjava.com").get();
	Element link = doc.select("a").first();
	String relHref = link.attr("href"); // == "/"
	String absHref = link.attr("abs:href"); // "http://zeusjava.com/"
```

在HTML元素中，URLs经常写成相对于文档位置的相对路径： <a href="/download">...</a>. 当你使用 `Node.attr(String key)` 方法来取得a
元素的href属性时，它将直接返回在HTML源码中指定定的值。
假如你需要取得一个绝对路径，需要在属性名前加 abs: 前缀。这样就可以返回包含根路径的URL地址`attr("abs:href")`
因此，在解析HTML文档时，定义base URI`非常重要`。
如果你不想使用`abs: 前缀`，还有一个方法能够实现同样的功能 `Node.absUrl(String key)`。

###三、提取数据
####1.设置属性的值
在解析一个Document之后可能想修改其中的某些属性值，然后再保存到磁盘或都输出到前台页面。

可以使用属性设置方法`Element.attr(String key, String value`), 和 `Elements.attr(String key, String value)`.
假如你需要修改一个元素的 class 属性，可以使用 Element.addClass(String className) 和 Element.removeClass(String className) 方法。
Elements 提供了批量操作元素属性和class的方法，比如：要为div中的每一个a元素都添加一个 rel="nofollow" 可以使用如下方法：
`doc.select("div.comments a").attr("rel", "nofollow")`;
与Element中的其它方法一样，attr 方法也是返回当 Element (或在使用选择器是返回 Elements 集合)。这样能够很方便使用`方法连用`的书写方式。比如：
`doc.select("div.masthead").attr("title", "jsoup").addClass("round-box")`;
####2.设置元素的html内容
设置一个元素的html内容可以用以下方式：

```java
	Element div = doc.select("div").first(); // <div></div>
	div.html("<p>lorem ipsum</p>"); // <div><p>lorem ipsum</p></div>
	div.prepend("<p>First</p>");//在div前添加html内容
	div.append("<p>Last</p>");//在div之后添加html内容
	// 添完后的结果: <div><p>First</p><p>lorem ipsum</p><p>Last</p></div>
	Element span = doc.select("span").first(); // <span>One</span>
	span.wrap("<li><a href='http://example.com/'></a></li>");
	// 添完后的结果: <li><a href="http://example.com"><span>One</span></a></li>
```
`Element.html(String html)` 这个方法将先清除元素中的HTML内容，然后用传入的HTML代替。
`Element.prepend(String first)` 和`Element.append(String last)` 方法用于在分别在元素内部HTML的前面和后面添加HTML内容
`Element.wrap(String around)` 对元素包裹一个外部HTML内容。

####3.设置元素的文本内容
设置一个元素的文本内容可以用以下方式：

```java
	Element div = doc.select("div").first(); // <div></div>
	div.text("five > four"); // <div>five &gt; four</div>
	div.prepend("First ");
	div.append(" Last");
	// now: <div>First five &gt; four Last</div>
```

文本设置方法与 HTML setter 方法一样：
`Element.text(String text)` 将清除一个元素中的内部HTML内容，然后提供的文本进行代替
`Element.prepend(String first)`和 `Element.append(String last)` 将分别在元素的内部html前后添加文本节点。
对于传入的文本如果含有像 `<, >` 等这样的字符，将以文本处理，而非HTML。

###三、使用Jsoup实现爬虫留作下一次再写
