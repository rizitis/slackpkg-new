## slackpkg-unofficial

This is a **patched** version of slackpkg<br>
- support /etc/slackpkg/whitelist
- keeping support for slackpkgpluss

---

## HowTo

This slackpkg version has in /etc/slackpkg a whitelist file.<br>
Every package name listed there escape from set blacklist. <br>

For example assume for your personal reasons having in blacklist the  `kde/` set. <br>
But you want some packages to included in you installation from this set (or any other) and receive official updates.<br> 
Just open whitelist and name them. example: `cat /etc/slackpkg/whitelist`
```
qca
futuresql
qcoro
digikam
kactivities
kactivities-stats
kstars
```
All these packages will be 

## Terminal Demo

[![Watch the demo](assets/demo.png)](https://asciinema.org/a/zhEjSRZTx23JD52RY5eiS3yTR)

---


## NOTE:

This slackpkg version is a personal fork, **Slackware project or slackpkg has no responsibility**<br>


---

## trademarks

Slackware is a [trademark](http://www.slackware.com/trademark/trademark.php) of Patrick Volkerding. <br>
©[Slackpkg](https://slackpkg.org/) Some Rights Reserved - Github [page](https://github.com/rworkman/slackpkg)

