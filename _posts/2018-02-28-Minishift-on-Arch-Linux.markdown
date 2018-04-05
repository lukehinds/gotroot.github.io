---
layout: post
title: Running Minishift on Arch Linux
subtitle: How
---

Install Dependencies

```
sudo pacman -S libvirt qemu dnsmasq ebtables
```

Grab the latest `docker-machine-kvm` release (as of time of writing 0.10.0)

```
sudo curl -L https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-centos7 > /usr/local/bin/docker-machine-driver-kvm
```

Add the KVM group to `qemu.conf`

```
sudo sed -r 's/group=".+"/group="kvm"/1' /etc/libvirt/qemu.conf > /etc/libvirt/qemu.conf
```

Add yourself to the kvm, libvirt groups

```
sudo usermod -a -G kvm,libvirt <user>
```

Implement the above for your current session

```
newgrp libvirt
```

Grab the latest minishift (as of time of writing 1.13.1)

```
wget https://github.com/minishift/minishift/releases/download/v1.13.1/minishift-1.13.1-linux-amd64.tgz ; tar zxvf minishift-1.13.1-linux-amd64.tgz
```

Add execution permissons to the minishift CLI.

```
chmod +x minishift-1.13.1-linux-amd64/minshift
```

If your default network is not running, start it (No idea why, but the arch maintainer for libvirt always stops all networks on update and leaves them inactive post updgrade)

```
sudo virsh net-start default
```

Start minishift

```
minishift start
```
