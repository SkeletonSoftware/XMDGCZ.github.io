---
layout: page
title: Blog
category_type: 'Blog'
---
{% assign category_list = site.custom-categories %}

{% include list-categories.html lang=page.lang %}
{% include list-posts.html lang=page.lang %}