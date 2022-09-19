---
title: "programmers"
layout: archive
permalink: categories/programmers
author_profile: true
sidebar_main: true
---


{% assign posts = site.categories.programmers %}
{% for post in posts %} {% include categories.html type=page.entries_layout %} {% endfor %}