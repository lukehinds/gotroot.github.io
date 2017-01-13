---
layout: post
title: Mi5 Security challenge
subtitle: How I cracked it.
bigimg: /img/path.jpg
---

OK, before anyone gets the wrong idea. I did not literally hack MI5.

I guess the title is a little click baity, granted.

I was actually on the MI5 website after seeing a documentary about the UK's secret service, and came across a challenge on the 'careers' section.

![puzzle](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/Screenshot-from-2015-07-20-20-06-34.png)

At first I thought it was some interactive flash puzzle where you have to fit the squares to order, but if you click on the image it actually hyperlinks to a pcap file (network trace).

So from there I downloaded the file, and fired up wireshark

![wget](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/Screenshot-from-2015-07-20-19-12-37.png)

I noted some SMTP flows...

![wireshark](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/Screenshot-from-2015-07-20-19-18-31.png)

I then followed the TCP Stream and could see an email conversion.

As you can see, there is a password.

![password](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/Screenshot-from-2015-07-20-19-14-42.png)

Pretty easy stuff I guess, but then there are plenty of people around who would send an email with a password in plain view (have you ever done that?)

So next it was a case of finding what the password was for.

Later down the trace, I found another SMTP stream and lo and behold, a base64 encoded rar file that had been attached to the email....

![smtpstream](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/Screenshot-from-2015-07-20-19-19-07.png)

I dumped this file out and cleared all of the stuff we don't need...

![smtpstream](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/Screenshot-from-2015-07-20-19-54-41.png)

From there it was a case of decoding this raw file from base64.

Python of course is the obvious choice. I  found there is library  for base64 de/encoding,  so I knocked up a very quick rudimentary script:

```
import base64, re, sys

infile = open(sys.argv[1], 'r')
outfile = open(sys.argv[2], 'wb')

for line in infile:
    encoded64 = line.rstrip()
    decoded64 = base64.b64decode(encoded64)
    outfile.write(decoded64);

infile.close()
outfile.close()
```

I then used the script to decode 'backup.rar', by using the password from the first TCP flow and out came four files on the other end...

![smtpstream](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/Screenshot-from-2015-07-20-21-56-21.png)

After this I worked out how to get the 'secret messages' from the files.

I will leave that up to the reader to work out, and in turn leave some quality control in place for anyone going forward for an interview.
