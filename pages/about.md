---
layout: page
title: About
description: 打码改变世界
keywords: Yan Kun, 严锟
comments: true
menu: 关于
permalink: /about/
---

我是严锟，世界的解析。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

## 论文

{% for paper in site.data.paper %}
- [{{ paper.papername }}]({{ paper.url }})
{% endfor %}

