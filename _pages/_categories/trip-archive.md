---
title: "Trip"
layout: archive
permalink: /categories/trip/
author_profile: true
taxonomy: "Trip"
sidebar_main: true
sidebar:
    nav: "docs"
---



{% assign posts = site.categories.trip %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}