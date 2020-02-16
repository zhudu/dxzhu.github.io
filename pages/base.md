---
layout: page
title: base
description: 基础筑起高楼
keywords: 基础, base
comments: false
menu: 基础
permalink: /basse/
---

> 万丈高楼从这起！

<ul class="listing">
{% for base in site.base %}
{% if base.title != "template page" %}
<li class="listing-item"><a href="{{ site.url }}{{ base.url }}">{{ base.title }}</a></li>
{% endif %}
{% endfor %}
</ul>
