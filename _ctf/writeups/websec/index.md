---
layout: default
permalink: /ctf/writeups/websec/
---
 
{% assign path_parts = page.path | split: '/'| size | minus: 1 | to_integer  %}
{% assign dir_name = page.path | split: '/' | slice: 0, path_parts | last %}

# {{ dir_name | capitalize }}

These are all my writeups for the challenges I created for the challenges in the practical course "Web-Application Security" held by Fabian Franzen in SS25 at TUM.

{% for item in site.ctf %}
  {% if item.path contains dir_name and item.path != page.path %}
- [{{ item.title }}]({{ item.url }})
  {% endif %}
{% endfor %}
