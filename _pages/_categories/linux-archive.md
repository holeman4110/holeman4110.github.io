---
title: "Linux"
layout: category
permalink: /categories/linux/
author_profile: true
taxonomy: "linux"
sidebar:
    nav: "docs"
---




{% assign posts = site.categories.linux %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}