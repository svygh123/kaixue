---
layout: page
title: 文章分类
---

{% for category in site.categories %}

<div id="{{ category | first }}"> <h3> {{ category | first | capitalize }} </h3> </div>
{% for posts in category %}
{% for post in posts %}
{% if post.url %}
  <span style="margin-left: 18px;color:#9a9a9a">{{ post.date | date: "%Y.%-m.%-d" }}</span> &raquo; [ {{ post.title }} ]({{ post.url }})
{% endif %}
{% endfor %}
{% endfor %}
{% endfor %}
