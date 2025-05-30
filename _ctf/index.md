---
layout: page
title: CTF
permalink: /ctf
---
# [Capture the Flag](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity))

I'm currently having a lot of fun playing with my teammates from H4TUM some CTFs. We will also host our own CTF this year.

We meet biweekly in Garching to present and discuss topics or just spend time together.  
Everyone is welcome to [join](https://matrix.to/#/!xGLCRFVgLZbkyXXjkK:tum.de?via=matrix.org&via=tum.de&via=in.tum.de) if you are interested.


## Recommendations
If you want to learn what all this is about and tackle some challenges, I can "recommend" (at least I started with) [PicoCTF](https://picoctf.org/). 
If you are at TUM, I can "recommend" (it is a lot of fun, but your life will definitely suffer) the practical course Binary Exploitation.

And if you are even more interested into Cryptography, check out [CryptoHack](https://cryptohack.org/).

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
I participated in some on-site ctfs and security congresses. I might share here some impressions on them in future.

---
## Tools and Concepts
This section is expected to be a collection of topics or some weird scripts, I wrote over the time and might be useful for more general or related topics.
Don't expect any great documentation or general usability.

{% for item in site.ctf %}
  {% if item.path contains '/tools/' %}
- [{{ item.title }}]({{ item.url }})
  {% endif %}
{% endfor %}
