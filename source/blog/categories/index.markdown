---
layout: page
title: "分类列表"
date: 2013-07-28 23:11
comments: true
sharing: false
footer: true
---
<ul>
{% for item in site.categories %}
    <li><a href="/blog/categories/{{ item[0] }}/">{{ item[0] | capitalize }}</a> [ {{ item[1].size }} ]</li>
{% endfor %}
</ul>
<section>
  <h1>标签云</h1>
  <ul id="tag-cloud">{% category_cumulus bgcolor:#f2f2f2 %}</ul>
</section>
