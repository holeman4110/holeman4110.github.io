---
title: "os"
layout: category
permalink: /categories/os/
author_profile: true
taxonomy: "os"
sidebar:
    nav: "docs"
---




{% assign posts = site.categories.os %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}