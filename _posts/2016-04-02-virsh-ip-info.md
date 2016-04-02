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

Look at some leased IP's

~~~
# virsh net-dhcp-leases --network default
Expiry Time          MAC address        Protocol  IP address                Hostname        Client ID or DUID
-------------------------------------------------------------------------------------------------------------------
2014-06-16 03:40:14  52:54:00:85:90:e2  ipv4      192.168.150.231/24        fedora20-test   01:52:54:00:85:90:e2
2014-06-16 03:40:17  52:54:00:85:90:e2  ipv6      2001:db8:ca2:2:1::c0/64   fedora20-test   00:04:b1:d8:86:42:e1:6a:aa:cf:d5:86:94:23:6f:94:04:cd
2014-06-16 03:34:42  52:54:00:e8:73:eb  ipv4      192.168.150.181/24        ubuntu14-vm     -
2014-06-16 03:34:46  52:54:00:e8:73:eb  ipv6      2001:db8:ca2:2:1::5b/64   -               00:01:00:01:1b:30:c6:aa:52:54:00:e8:73:eb    
~~~
