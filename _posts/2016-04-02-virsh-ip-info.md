---
layout: post
title: Using domifaddr
subtitle: Using virsh domifaddr, for getting vm IP info
bigimg: /img/path.jpg
---

Two useful IP tools I discovered in virsh, care of [Daniel P Berrange](http://libvirt.org/git/?p=libvirt.git;a=search;h=2f36e6944e6eb56a00e19fcd85ec8513461597c9;s=Daniel+P.+Berrange;st=committer)

~~~
virsh domifaddr <domain> [interface] [–full] [–source lease|agent]
~~~

~~~
 # virsh domifaddr fedora_vm --full
 Name       MAC address          Protocol     Address
 -------------------------------------------------------------------------------
 vnet0      52:54:00:2e:45:ce    ipv6         2001:db8:0:f101::2/64
 vnet1      52:54:00:b1:70:19    ipv4         192.168.105.201/16
 vnet1      52:54:00:b1:70:19    ipv6         2001:db8:ca2:2:1::bd/128
 vnet3      52:54:00:20:70:3d    ipv4         192.168.105.240/16
~~~


~~~
 # virsh domifaddr fedora_vm --full
 Name       MAC address          Protocol     Address
 -------------------------------------------------------------------------------
 vnet0      52:54:00:2e:45:ce    ipv6         2001:db8:0:f101::2/64
 vnet1      52:54:00:b1:70:19    ipv4         192.168.105.201/16
 vnet1      52:54:00:b1:70:19    ipv6         2001:db8:ca2:2:1::bd/128
 vnet3      52:54:00:20:70:3d    ipv4         192.168.105.240/16
~~~
