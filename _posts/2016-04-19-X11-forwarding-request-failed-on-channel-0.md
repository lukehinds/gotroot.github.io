---
layout: post
title: X11 forwarding request failed on channel 0
subtitle: How to fix
bigimg: /img/path.jpg
---

Ever get the following issue when trying to X-forward?

~~~
ssh -X root@192.168.124.29
root@192.168.124.29's password:
X11 forwarding request failed on channel 0
~~~

Huh?

~~~
ssh -vX root@192.168.124.29
root@192.168.124.29's password:
<snip>
debug1: Remote: No xauth program; cannot forward with spoofing.
X11 forwarding request failed on channel 0
~~~

Duh!

~~~
# yum install xauth
~~~
