---
layout: post
title: Epson Wireless Scanner on Fedora
---

This is for the Epson XP-312 wireless scanner / printer, but it should work with most wireless scanners from Epson.

First off, set up an address reservation on your router for the IP assigned to the Epson device. For example, mine is 192.168.1.44

After that go to the [Epson download page](http://download.ebz.epson.net/dsc/search/01/search/?OSC=LX), and search for drivers for your Linux drivers for your particular model.

![epson-page](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/epson-page.jpg)


Unarchive the tar.gz file downloaded

Mine was iscan-bundle-1.0.0.x64.rpm.tar.gz

~~~
$ tar zxvf iscan-bundle-1.0.0.x64.rpm.tar.gz
~~~

Now interestingly, you will find a folder and not an rpm, not a probem though, just cd into the directory and run the install script as root or with sudo.

~~~
$ sudo ./install.sh
~~~
If you prefer to see what the script does first, then run:

~~~
./install.sh --dry-run
~~~

Now a couple of edits to make this work...

Back up you sane.d dll conf

~~~
sudo mv  /etc/sane.d/dll.conf /etc/sane.d/dll.conf-backup
~~~

Create and edit a new file of '/etc/sane.d/dll.conf' (using vim or nano etc) and enter the following values:

~~~
net
epkowa
~~~

* Note, if you have a camera or other device using sane, then of course you will need to keep the relevant entries in place.

Now back up your epkowa.conf file:

~~~
sudo mv /etc/sane.d/epkowa.conf /etc/sane.d/epkowa.conf-bckup
~~~

Create and edit a new file of '/etc/sane.d/epkowa.conf' (using vim or nano etc) and enter the following values:

~~~
usb
scsi
net <printer_ip>
~~~

Where <printer_ip> is your printers IP address, set as an address reservation in your router.

~~~
usb
scsi
net 192.168.0.28
~~~

Now run iscan from the command line or your application starter and scan your images.

![iscan](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/iscan.jpg)
