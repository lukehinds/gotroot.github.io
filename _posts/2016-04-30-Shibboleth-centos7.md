---
layout: post
title: Shibboleth on CentOS7
subtitle: How to Deploy Shibboleth for SAML
bigimg: /img/path.jpg
---

For the following, we will be deploying this to as a virtual machine runnning on top of KVM / QEMU, but this could  just as easily be applied to a VirtualBox / OpenStack VM or even metal. You just need to follow the section titled 'Installing Shibboleth'

# Deploying CentOS7 under KVM

###  Grab a qcow2 image lets create a new domain:

~~~
qemu-img create -b centos7.0-base.qcow2 -f qcow2 shibboleth.qcow2
~~~

### Remove all the cloud-init which we don't need.

~~~
virt-customize -a shibboleth.qcow2 --run-command 'yum remove cloud-init* -y'
~~~

### Let's set a root password:

~~~
virt-customize -a shibboleth.qcow2 --root-password password:<password>

e.g...

virt-customize -a shibboleth.qcow2 --root-password password:dA98uH&^%38!9s
~~~

### Install our VM:

virt-install --ram 1024 --vcpus 1 --os-variant rhel7 \
    --disk path=/var/lib/libvirt/images/shibboleth.qcow2,device=disk,bus=virtio,format=qcow2 \
    --import --noautoconsole --vnc --network network:default --name shibboleth

### Get the IP address:

~~~
virsh domifaddr shibboleth --full
~~~

You should now be able to SSH into the new VM

# Install Shibboleth

## Get repo

* The OpenSuse repo works fine ...

~~~
curl -o /etc/yum.repos.d/security:shibboleth.repo http://download.opensuse.org/repositories/security://shibboleth/CentOS_7/security:shibboleth.repo
~~~

## Install NTP and httpd

~~~
yum install ntp httpd y
~~~

~~~
systemctl start ntpd httpd
~~~

## And finailly Shibboleth

~~~
yum install shibboleth
~~~

## Set permissive

Note: I know, i know. This is not ideal, but I personally have not had time to write the correct types as yet. I will get onto this, but dumping these notes on github pages while flying back from the Austin Summit.

~~~
sed -i s/^SELINUX=.*/SELINUX=permissive/ /etc/selinux/config
~~~

Reboot the VM.

# Test out Shibboleth with a GET of the Status

~~~
curl -k https://127.0.0.1/Shibboleth.sso/Status
~~~

Later on, I will try to show Keystone Integration.
