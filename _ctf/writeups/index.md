---
layout: default
title: CTF Write-ups
permalink: /ctf/writeups/
---

<h1>{{ page.title }}</h1>
<ul>
  {%- assign all_posts = site.ctf
       | sort: "date"
       | reverse -%}
  {%- for post in all_posts -%}
    {%- if post.name != "index.md" -%}
    <li>
      <a href="{{ post.url }}">
        {{ post.title | default: post.basename_without_ext }}
      </a>
      <small>— {{ post.date | date: "%Y-%m-%d" }}</small>
    </li>
  {%- endif -%}
  {%- endfor -%}
</ul>
