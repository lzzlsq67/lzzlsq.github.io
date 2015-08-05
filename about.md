---
title: About
layout: page
group: navigation
comment: true
---

#### 我是赵小轩，请叫我程序员

目前从事微信开发，开发语言呢是java，坐标上海，熟悉微信开发相关的相关API,以及公众号搭建，
有志同道合的筒子，可以联系我，一起探讨编程问题。本人同样爱好骑行，喜欢在周末时，驰骋在
大路上，有喜欢的骑行的朋友也可以联系我呐。
#### 联系我吧
{% if site.author.email %}
Email：{{ site.author.email }}
{% endif %}

{% if site.author.weibo %}
Weibo：<http://weibo.com/{{ site.author.weibo }}>
{% endif %}

{% if site.author.github %}
Github：<https://github.com/{{ site.author.github }}>
{% endif %}

{% if site.author.twitter %}
Twitter：<https://twitter.com/{{ site.author.twitter }}>
{% endif %}

{% if site.url %}
RSS：[{{ site.url }}{{ '/rss.xml' }}](/rss.xml)
{% endif %}

{% include support.html %}
