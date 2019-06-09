---
title: "Post about Programming"
layout: archive
permalink: /categories/programming
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.programming | sort:"date" %}

{% for post in posts %}
  {% include archive-single.html type=page.entries_layout %}
{% endfor %}
