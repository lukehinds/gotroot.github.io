---
layout: post
title: Convert qcow2 to VDI
subtitle: Using qemu-img
---
It's actually quite simple to convert a VDI to QCOW2.

There is no need for a virtualbox install, you can do it directly using qemu-img.

~~~
qemu-img convert -f vdi -O qcow2 source-image.vdi dest-image.qcow2
~~~
