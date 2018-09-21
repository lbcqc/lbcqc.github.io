---
layout: page
title: About
description: 怂是不可能怂的
keywords: Shuai li
comments: true
menu: 关于
permalink: /about/
---
呃呃呃呃呃，记录一下工作与生活。

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
