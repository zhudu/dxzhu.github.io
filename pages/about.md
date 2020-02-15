---
layout: page
title: About
description: 愿得韶华刹那，开得满树芳华。
keywords: duxin Zhu, 朱笃信
comments: true
menu: 关于
permalink: /about/
---

以梦为马，不负韶华。

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
