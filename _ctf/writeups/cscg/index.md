---
title: CSCG
layout: default
permalink: /ctf/writeups/cscg/
---

{% assign path_parts = page.path | split: '/'| size | minus: 1 | to_integer  %}
{% assign dir_name = page.path | split: '/' | slice: 0, path_parts | last %}

# {{ dir_name | capitalize }}

Welcome to the {{ page.path | split: '/' | slice: 0, -1 | last }} CTF section. Here's a list of individual challenges:


{% for item in site.ctf %}
  {% if item.path contains dir_name and item.path != page.path %}
- [{{ item.title }}]({{ item.url }})
  {% endif %}
{% endfor %}
