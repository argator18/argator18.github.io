---
layout: default
---
# Task php00: Crazy PHP Router
This week is PHP time! Can you imagine a router with a PHP interface? Probably not the best combination. Find out yourself!

# Overview
This php server is again apart of an apache http server.
The actions a user can do are:
- login
- list users (requires login)
- ping other host on network (requires login)

# Vulnerability
### Leaking Users
The apacha config, has no restrictions at all but sets this:
        `DocumentRoot /var/www/html`
By default every file in this folder is accessible, including the `.csv` file holding the users and their passwords!
This gives us the following users:
```csv
fabianf,e9a4f5c10d078edb9ccd4f14bf822983df280b11
fabiank,076f1c736bf40345d6edcb956185abd9962fe7d1
claudia,0e69738545303274592611682853576306899720
ludwig,4fa50e713e76ed118ccb80e3b10b9b3c47a21ccf
```
As you can see the passwords are hashed (with sha1):
```php
hash("sha1", $_POST['password'])
```

### Login
Next we want to find a way to login into the system.

While sha1 is not considered as a secure hash function. There still don't exist any preimage attacks. This would be neccessary to get a password, that gives the exact result.

Great that PHP is buggy: we can use the concept of [magic hash "collisions"](https://github.com/spaze/hashes).

#### The == and the ===
The problem is how php processes values without a strict type system. In general == and === is not the same in php. === is usually what you expect and == is doing something else.
There are dozen of funny examples how this evaluates to true for the weirdest and most different examples you can think of.
The underlying problem is, that == does type conversion before comparing, while === returns false, if the types differ.

In this case we only need the fact that strings of hex numbers are evaluated to numbers before comparison.
Also php is "smart" enough to *automatically* convert it accoring to its format. That's why it recognizes "5e6" as scientific notation for $5 \cdot 10^6$, exaclty what the user expected, of course.
This is why everthing leading with a "0e" will be converted to 0.
Luckily Claudias password fullfils exactly this characterization and hence we have to find a hash collision for that one byte, which is a 8 bit brute force (or lookup on the gitHub page):).

This way the hashes of the password match according to php even though 
the passwords themself don't.

### Get the flag
The ping is executed like that:
```php
		passthru("ping -c 1 '".$ip."'");
```
Also it is never checked how the ip looks like, so it can contain arbitrary content like "'; /bin/get_flag; echo'", which obviously escapes the quotes, calls the flag binary and makes sure the command doesn't fail because of syntax errors, to get the output.
