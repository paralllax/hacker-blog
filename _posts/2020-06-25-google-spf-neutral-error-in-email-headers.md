---
title: spf=neutral google neither permitted nor denied email header error
published: true
---

* * *

I have been using a free tier slack instance for our family. Once we started running out of space, I started migrating to Rocket.Chat. As part of this I had to set up an email server, which gave me a bit of trouble. When sending an email I would get a message like the following in the headers:

```
 spf=neutral (google.com: 192.168.1.1 is neither permitted nor denied by best guess record for domain of noreply@mydomain.com) smtp.mailfrom=noreply@mydomain.com 
```

This led to the email getting flagged as spam since google was unable to determine if my host was actually allowed to send email from my domain. 

Admittedly, I haven't had much a chance to play around with email and DNS records for it, so a lot of this was a fun learning experience. In any case, my problem was SPF records were deprecated in [2014](https://serverfault.com/a/807543). Turns out you just had to create them as SPF in TXT. This is what my records ended up looking like:

```
;; A Records
mydomain.com.	1	IN	A	192.168.1.1
postfix-01.mydomain.com.	1	IN	A	172.168.1.1

;; MX Records
mydomain.com.	120	IN	MX	10 postfix-01.mydomain.com.

;; TXT Records
mydomain.com.	1	IN	TXT	"v=spf1 mx -all"
```

After I switched the SPF to SPF in TXT, the error went away and it was verified that my host could send mail. The error was very vague and generic so I thought I would post about it in case anyone else runs into the same issue as me. I configured opendkim, and TLS later for further verification of my host to avoid getting tagged as spam. I may come out with a guide on that later. Thanks for reading!
