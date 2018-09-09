---
layout: page
title: About
description: 怂是不可能怂的
keywords: Shuai li
comments: true
menu: 关于
permalink: /about/
---
既然没法选择了，就坚强的走下去吧

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
