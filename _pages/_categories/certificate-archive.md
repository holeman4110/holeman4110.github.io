---
title: "certificate"
layout: archive
permalink: /categories/certificate/
author_profile: true
taxonomy: "certificate"
sidebar_main: true
sidebar:
    nav: "docs"
---




{% assign posts = site.categories.certificate %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}