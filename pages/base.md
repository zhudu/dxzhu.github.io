---
layout: categories
title: base
description: 基础筑起高楼
keywords: 基础, base
comments: false
menu: 基础
permalink: /base/
---

> 万丈高楼平地起！

<ul class="bases">
{% for base in site.base %}
{% if base.title != "base Template" %}
<li class="bases-item"><a href="{{ site.url }}{{ base.url }}">{{ base.title }}</a></li>
{% endif %}
{% endfor %}
</ul>

