---
title: "git"
layout: archive
permalink: /categories/git/
author_profile: true
taxonomy: "git"
sidebar_main: true
sidebar:
    nav: "docs"
---




{% assign posts = site.categories.git %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}