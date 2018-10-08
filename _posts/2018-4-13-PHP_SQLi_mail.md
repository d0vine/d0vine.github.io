---
layout: post
title: misc 0x01 / SQL Injection with a valid e-mail address
category: misc
tags: [rozwal, sql-injection, php, filter, security, misc]
---


There is no doubt some specifications (*cough cough* looking at you, Bluetooth!) are overly complicated. Not only is this a hindrance to those implementing it, but it can also cause security issues due to the many ways of bypassing the possible security mechanisms.

Quite some time ago there was a challenge published on <a href="https://rozwal.to" target="_blank">rozwal.to</a> which involved an SQL injection with e-mail validation in PHP. If you were unfortunate enough to see <a href="http://www.ex-parrot.com/~pdw/Mail-RFC822-Address.html">the e-mail address validation regex</a>, you know the drill: e-mail address specification isn't exactly the most complicated one, but neither is it easy. ;)

The code was roughly like this:

```php
<?php
require 'db.php';
if (isset($_GET['mail']))
{
	$mail = $_GET['mail'];

	if (filter_var($mail, FILTER_VALIDATE_EMAIL) === false)
		die("Incorrect email\n");

	$query = "SELECT name FROM users WHERE email='$mail'";

	$q = mysql_query($query);
	$row = mysql_fetch_array($q);
	if ($row) {
		$name = $row['name'];
		echo "Your name is <strong>$name</strong>!<br><br>";
	}
}
?>
```

The validation was performed via the *filter_var* function with *FILTER_VALIDATE_EMAIL* as an argument. Let's have a quick look at <a href="http://php.net/manual/en/filter.filters.validate.php" target="_blank">the manual</a>:

> Validates whether the value is a valid e-mail address.
> 
> In general, this validates e-mail addresses against the syntax in RFC 822, with the exceptions that comments and whitespace folding and dotless domain names are not supported.

So, let's check the RFC! It can be found at <a href="https://www.w3.org/Protocols/rfc822/" target="_blank">https://www.w3.org/Protocols/rfc822/</a>.

I'll skip a few steps and jump straight on to a string that will, at the same time, be a valid e-mail and perform a very simple SQL injection:

`'/**/and/**/1=0/**/union/**/select/**/'SQL+injection'/**/#@test.lul.topkek`

Whoa! Why would that be possible? Well, the answer is obvious: because that is a valid e-mail! There is a nice little <a href="http://jkorpela.fi/rfc/822addr.html">page</a> that explains the RFC in a more human-friendly way:

> addr-spec   =  local-part "@" domain
> (...)
> local-part  =  word *("." word)
> 
> 
> A word is either an atom or a quoted string.
> 
> An atom is a sequence of printable ASCII characters except the space or any of the following:
>     ()<>@,;:\".[]
> 
> Positively speaking, this means that the valid constituents of an atom are the following:
>     !"#$%&'*+-/0123456789=?
>     @ABCDEFGHIJKLMNOPQRSTUVWXYZ^_
>     `abcdefghijklmnopqrstuvwxyz{|}~

This means that we can safely use `'` (which is used for injection in this case), `/*` and `*/` (comments used to achieve separation, just like spaces) and `#` (the comment to stop the other part of the query) in the local part. The rest is a matter of creativity ;)

Cheers!

