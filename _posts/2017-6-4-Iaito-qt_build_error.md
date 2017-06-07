---
layout: post
title: Iaitō - build error with Qt Creator
---


Today I've decided check out [Iaitō](https://github.com/hteso/iaito) - a frontend for the wonderful *radare2*. It seemed pretty neat (especially watching Hugo's great presentation ["Sweet tools o' mine"](https://www.youtube.com/watch?v=QSUeAJnqHqY) at t2/2016) so I've cloned the repository, installed Qt (5.6, as suggested in the repo), installed r2 from the submodule, opened the project with Qt Creator... and stopped right there.

After running "Build" I saw an error:

`r_core development package not found`

If you have the same problem and did your homework, you know that's due to the `/usr/local/bin` path missing in the PATH variable. This could have happened, for example, when you copied the `Qt Creator.app` directory over to `/Applications` although I haven't done more research on that, TBH.

In order to fix that, go to the `Projects` tab, open `Details` in the `Build environment` section and append `:/usr/local/bin` to PATH (see the image below in case you have trouble finding that setting). Keep in mind that this need to be done for `Debug` and `Release` profiles separately (if I'm not mistaken; feel free to correct me otherwise).

![project settings](/images/iaito_settings.png)

Hopefully now it'll be only easier ;-)

Cheers!

<!--
Why hello there! My name's Chris (can be Kris as well) and this is another attempt of mine at creating a blog-wannabe where I could put my thoughts and ideas.

What will I post here? I'll probably write about CTF challenges, possibly some security stuff and programming experiences. As I don't have that much experience in the security field, for now I'll stick with CTFs and programming.
Please bear in mind that this is also an attempt to boost my writing skills (both in English and overall) so you might find some mistakes. I'd like to apologize for those in advance ;-)

See you soon!
-->
