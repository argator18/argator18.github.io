---
layout: page
title: CTF
permalink: /ctf
---
# [Capture the Flag](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity))

I'm currently having a lot of fun playing with my teammates from H4TUM some CTFs. We will also host our own CTF this year.


## Recommendations
If you want to learn what all this is about and tackle some challenges, I can "recommend" (at least I started with) [PicoCTF](https://picoctf.org/). 
If you are at TUM, I can "recommend" (it is a lot of fun, but your life will definitely suffer) the practical course Binary Exploitation.

And if you’re more interested in cryptography, check out [CryptoHack](https://cryptohack.org/).

The goto page for ctfs is [CTFTIME](https://ctftime.org/).

I make sure to participate each year in at least:
- [Cyber Security Challenge Germany](https://cscg.de/)

And from now on:
- [FlareOn](https://flare-on.com/)

If you are interested into Security in general, the [CCC](https://www.ccc.de/en/) and specifically their [congress](https://events.ccc.de/congress/) might be interesting to you.

---

## [Writeups](/ctf/writeups)

{% for item in site.ctf %}
  {% assign path = item.path %}
  {% if path contains '/writeups/' and path contains '/index.md'  %}
    {% assign path_parts = path | split: '/'| size | minus: 1 | to_integer  %}
    {% assign dir_name = path | split: '/' | slice: 0, path_parts | last %}
- [{{ dir_name }}]({{ item.url }})
  {% endif %}
{% endfor %}

---
## Experiences and Impressions

---
## Tools and Concepts

{% for item in site.ctf %}
  {% if item.path contains '/tools/' %}
- [{{ item.title }}]({{ item.url }})
  {% endif %}
{% endfor %}
