---
layout: default
---
# Task xss02: Tweeter

## Tl;dr
Use a simple stored XSS in the username to make the admin fetching JS code from a storage.google domain, as the CSP doesn't allow inline js, but is misconfigured and allows user controlled urls to load js from. 
There you can store JS code, that leaks the admins DMs once he accesses the website.

## Description
I found this Twitter (sometimes also known as X) clone. It seems like only a few people are using it to post their (sometimes out-of-date) jokes.

It's not really something for me, but I hope you can get some nice entertainment from it!

Hint 1: The admin uses a recent version of the Chromium browser.

Hint 2: The goal of this challenge is to leak the admin's direct messages.

## Overview
The website consists of a register + login page.
Additionally you can select the feed page to see some random tweets from people you follow (randomly selected) and another page where you can tweet and see (only!) your own tweets.

All tweets can be *reported* anytime. Even if you can't see them in your tweet and you can also report your own tweets. You can also see easily, that a report requires the id of the tweet, which is incremented through all users for all tweets. So the tweet that was created after the tweet with id $n$ is obviously $n+1$.

## Code
The code reveals, that the admin is actually visiting the users profile containing all tweets, once a tweet is reported.

Also we can see, that many api endpoints, are only accessible for the admin. For example looking at others dms is "of course" ( nice privacy ) only possible for admins:
```java
	@RequestMapping(value = "/user/{username}/dms/{other}", method = RequestMethod.GET)
	public List<MessageInfo> dmhistory(@PathVariable(value="username") String username, @PathVariable(value="other") String other, Authentication auth, HttpServletRequest request, HttpServletResponse response) {
		if(auth == null || !auth.isAuthenticated())
			throw new ResponseStatusException(HttpStatus.FORBIDDEN);
		
		/* Only admins can use the API to look at other feeds */
		if(!request.isUserInRole("ADMIN") && !request.getRemoteUser().equals(username))
			throw new ResponseStatusException(HttpStatus.FORBIDDEN);
		
		if(request.isUserInRole("ADMIN"))
			tweeterInitializer.updateFlagDm();
		
		return dmRepository.getDmHistory(username, other).stream().map(DirectMessage::getInfo).collect(Collectors.toList());
	}
```
But this really is, what we want to achieve in the end.
As the only thing we can input in this website are the following three things, we need to check them:
- id of the reported tweet
- username + password
- tweet content

### WebSecurityConfig
Last but not least let's check out this very important file for WebSecurity: WebSecurityConfig.java:
```java
	/* We want to allow users with slashes and our API encodes the usernames in the URI */
	@Bean
    public HttpFirewall allowUrlEncodedSlashHttpFirewall() {
        DefaultHttpFirewall firewall = new DefaultHttpFirewall();
        firewall.setAllowUrlEncodedSlash(true);
        return firewall;
	}
```
Ok, weird: slashes are allowed in usernames.

Here we can see the defined CSP, which is obviously flawed for several reasons.
```java
    /* Allow googleapis so we can use jQuery */
    .and().headers().contentSecurityPolicy("script-src 'self' https://*.googleapis.com");
```
Read more about this [here](https://www.vaadata.com/blog/content-security-policy-bypass-techniques-and-security-best-practices/) and this post, as well as some others proposed some exploit for the missing `object-src` policy, but this attack vector didn't work out for me (I would be grateful, if someone has insights here).
Still wilcards are usually if used unthoughtful a bad idead in those whitelisting cases. So in this case.



## Vulnerability

### Report
Through the fact that usernames can include slashes we are able to let the admin visit any page we want relative to the original and with the condition that it has to end with /tweets.
This vulnerability isn't actually needed in the end, which will turn out to be a common pattern in this challenge

### XSS
We can input Username, Password and Tweet content. As Passwords are naturally never shown but only saved in the database we only have to check username and tweet content for XSS.
As the username is pasted directly as a string into the html code, this should give us an obvious xss:
This is how the username is included:
```html
      <meta name="user" content="${sessionScope.user}"/>
```

And this is a conceptual XSS vector:
`{username2}">{XSS}`

While `username2` will be now interpreted as the username you could use here some other actually existing username to make the page still working, but you don't need that as we will only work with the api.
This closes the current meta tag and everything that comes after will be interpreted as html tags/anchors.

Another XSS vulnerability can also be found in the tweet content. It seems like it is sanitized.
```js
	sanitized_content = DOMPurify.sanitize(tweet.content);
```
But looking into the sanitizer or specifically in its version that is shipped with, we can see that DomPurify 2.2.8 is used, which is very old by may 2025 and several xss attack vectors have been found in this [version](https://security.snyk.io/package/npm/dompurify/2.2.8). 
So we could also use some of these to get xss, if the one with the Username was not here.
For example this creates a working XSS in this version:
```python
def generate_mxss_payload():
    html = ""
    html += "<div>" * 506
    html += """<table>
      <caption>
        <svg>
          <title>
            <table><caption></caption></table>
          </title>
          <style><a id="</style><img src=https://requestbin.kanbanbox.com/14987us1 onerror=alert(1)>"></a></style>
        </svg>
      </caption>
    </table>"""
    return html
```

### CSP
The problem is that even though we have "XSS", we don't. The CSP is effectively forbidding executing inline js and hence we cannot execute anything. So now let's discuss the wildcard mentioned above. In this case every service from googles api is allowed to be used, to load js code from. This includes:
- the storage api
- JSONP (legacy) endpoints 

#### Storage API
The storage api offers to upload arbitrary content and load it from there. So we can just create a so called google bucket upload our xss code there and retrieve it with `<script src={url_to_xss}></script>`. Now this will be executed as the CSP allows laoding js from this source.

#### JSONP
Another solution includes using JSONP endpoints. Basically they allow through the `callback` url parameter arbitrary js code execution. Their existence is outdated and was introduced before CORS header replaced them, to "bypass" (from a developer POV) the same-origin policy.
Unfortunately even google still has some of them open as listed [here](https://github.com/zigoo0/JSONBee/blob/master/jsonp.txt).
