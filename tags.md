---
title: 标签归档
layout: default
permalink: /tags
---

## 标签归档

{% assign all_tags = "" | split: "" %}
{% for page in site.pages %}
  {% if page.tags %}
    {% for tag in page.tags %}
      {% unless all_tags contains tag %}
        {% assign all_tags = all_tags | push: tag %}
      {% endunless %}
    {% endfor %}
  {% endif %}
{% endfor %}
{% assign all_tags = all_tags | sort %}

**所有标签：** {% for tag in all_tags %}[{{ tag }}](#{{ tag | slugify }}) {% endfor %}

---

{% for tag in all_tags %}
### {{ tag }}
{: #{{ tag | slugify }}}

{% for page in site.pages %}
  {% if page.tags contains tag %}
- [{{ page.title }}]({{ page.url | relative_url }})
  {% endif %}
{% endfor %}

{% endfor %}
