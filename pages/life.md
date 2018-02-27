---
layout: page
title: 城里的人想出来，城外的人想进去
description: 没什么可说的
keywords: 围城
comments: false
menu: 围城
permalink: /life/
---

> 有一天朋友问我，为什么你总是买一大堆的东西塞进你的冰箱里，就不能等吃完再买吗？我想，如果它总是满的，就不会再有企鹅住进来了吧。尽管我非常想念那只没节操的企鹅。

<ul class="listing">
{% for life in site.life %}
{% if life.title != "Life Template" %}
<li class="listing-item"><a href="{{ site.url }}{{ life.url }}">{{ life.title }}</a></li>
{% endif %}
{% endfor %}
</ul>

<ul class="listing">
{% for wiki in site.wiki %}
{% if wiki.title != "Wiki Template" %}
<li class="listing-item"><a href="{{ site.url }}{{ wiki.url }}">{{ wiki.title }}</a></li>
{% endif %}
{% endfor %}
</ul>
