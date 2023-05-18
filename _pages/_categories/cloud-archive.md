---
title: "Cloud"
layout: category
permalink: /categories/cloud/
author_profile: true
taxonomy: "cloud"
sidebar:
  nav: "docs"
---

{% assign posts = site.categories.cloud %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}
