---
layout: page
title: base
description: 基础筑起高楼
keywords: 基础, base
comments: false
menu: 基础
permalink: /base/
---

> 万丈高楼平地起！

<!-- <ul class="bases">
{% for base in site.base %}
{% if base.title != "base Template" %}
<li class="bases-item"><a href="{{ site.url }}{{ base.url }}">{{ base.title }}</a></li>
{% endif %}
{% endfor %}
</ul> -->

<section class="container bases-content">
{% assign sorted_categories = site.categories | sort %}
{% for category in sorted_categories %}
<h3>{{ category | first }}</h3>
<ol class="bases-list" id="{{ category[0] }}">
{% for base in category.last %}
<li class="bases-list-item">
<span class="bases-list-meta">{{ base.date | date:"%Y-%m-%d" }}</span>
<a class="bases-list-name" href="{{ site.url }}{{ base.url }}">{{ base.title }}</a>
</li>
{% endfor %}
</ol>
{% endfor %}
</section>
