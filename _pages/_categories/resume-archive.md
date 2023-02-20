---
title: "resume"
layout: archive
permalink: /categories/resume/
author_profile: true
taxonomy: "resume"
sidebar_main: true
sidebar:
    nav: "docs"
---




{% assign posts = site.categories.resume %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}