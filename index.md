---
title: 首页
layout: default
permalink: /
---

## 欢迎来到我的博客

这里是我的个人学习笔记和技术博客，主要记录 Java 相关的学习内容和日常思考。

### 最近文章

{% assign sorted_pages = site.pages | sort: "date" | reverse %}
{% for page in sorted_pages %}
  {% if page.title and page.url != "/" and page.url != "/categories" and page.url != "/tags" %}
- [{{ page.title }}]({{ page.url | relative_url }}) {% if page.categories %}<small>({{ page.categories }})</small>{% endif %}
  {% endif %}
{% endfor %}
