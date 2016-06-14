---
layout: post
title: Record your terminal to gif
subtitle: Using ttyrec & ttygif
---

I recently needed to record some terminal sessions, as back up for a demo I planned to present. I was
already aware of asciinema.org, but found that the lack of full screen mode, meant it came up short for my
needs.

In the end I went for a combination of the very old ttyrec application, with its result recording, converted
into GIF format using ttygif.

An example of the sort of result you can get:

![demogif](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/tty.gif)

* Note: If you zoom your browser in (mouse wheel), you will see the resolution improves, which is good enough
for a full screen presentation.

## Building ttyrec

* Note: ttyrec has not been maintained since 2006, but then again, not a lot has changed to the tty code in
the kernel, so it works fine. I just had to patch the code to work on Fedora. With other dists, your mileage
may vary.

Download the latest (2006!) version of ttyrec

~~~
wget http://0xcc.net/ttyrec/ttyrec-1.0.8.tar.gz
~~~

~~~
tar zxvf ttyrec-1.0.8.tar.gz ; cd ttyrec-1.0.8.tar.gz
~~~

Download the following patch (as of writing this works on Fedora 23 (non wayland) and CentOS / RHEL 7)

~~~
wget https://gist.githubusercontent.com/lukehinds/226452a6c3fbdc1275bdea66c954e8f6/raw/9ce0c7446e7f328e42d24419c31be5d605de428d/ttyrec-1.0.8.patch
~~~

Apply the patch

~~~
patch -i ttyrec-1.0.8.patch
~~~

And make

~~~
make
~~~

## Building ttygif

~~~
git clone https://github.com/icholy/ttygif.git ; cd ttygif
~~~

~~~
make
~~~

~~~
sudo make install
~~~

## Usage

~~~
./ttyrec
~~~

Now type out what it is you wish to record, and then when your done 'exit' out of the terminal session.

Convert to gif:

~~~
ttygif ttyrecord
~~~

All done!
