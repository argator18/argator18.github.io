---
title: Cryptanalysis
layout: default
permalink: /ctf/writeups/cryptana/
---
 
{% assign path_parts = page.path | split: '/'| size | minus: 1 | to_integer  %}
{% assign dir_name = page.path | split: '/' | slice: 0, path_parts | last %}

# {{ dir_name | capitalize }}

These are all my writeups to the challenges I solved during the course "Cryptography and Cryptanalysis [CIT413046]" held in SS25 by [Lorenz Panny](https://yx7.cc) at TUM.

<ol>
{% for item in site.ctf %}
  {% if item.path contains dir_name and item.path != page.path %}
    <li><a href="{{ item.url }}"> {{ item.title }} </a></li>
  {% endif %}
{% endfor %}
</ol>
