---
layout: post
title: Puppet Master / Agent Setup using vagrant and libvirt
---

This guide will provide to vagrant vm's sutiable for quickly testing puppet code.

Ideally these should be ported into `Vagrantfile`. I will try and do that sometime.

This guide assumes you already have kvm/qemu and libvirt installed.

## Install libvirt provider in vagrant on Arch



```
sudo pacman -S vagrant
CONFIGURE_ARGS='with-ldflags=-L/opt/vagrant/embedded/lib with-libvirt-include=/usr/include/libvirt with-libvirt-lib=/usr/lib'

GEM_HOME=~/.vagrant.d/gems GEM_PATH=$GEM_HOME:/opt/vagrant/embedded/gems PATH=/opt/vagrant/embedded/bin:$PATH

vagrant plugin install vagrant-libvirt
```

## Install libvirt provider on Fedora

https://developer.fedoraproject.org/tools/vagrant/vagrant-libvirt.html

## Master Setup

Create a `puppermaster` folder and grab a CentOS 7 Box

`vagrant init centos/7; vagrant up --provider libvirt`

and then `vagrant ssh` into your new machine to peform the following steps:

### Set a hostname

```
sudo hostnamectl set-hostname puppetmaster
```

### Update /etc/hosts

Get the IP address from the puppetmaster `ip a` and make an entry into `/etc/hosts` in the agent

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.121.82     puppetagent
```

### Install the RDO repo

```
sudo yum install -y https://rdoproject.org/repos/rdo-release.rpm
```
* Note: I use the rdo repository, as I need to use the same puppet version as RDO OpenStack

### Update

```
sudo yum update -y
```

### Install Puppet Server

```
sudo yum install -y puppet-server
```

Add the following to `/etc/puppet/puppet.conf`

```
certname = puppetmaster
dns_alt_names = puppetmaster
```

### Generate a new certificate

Start the server to create a puppetmaster certificate

```
sudo puppet master --verbose --no-daemonize
```

## Client Setup

```
sudo yum install -y https://rdoproject.org/repos/rdo-release.rpm
```

```
sudo yum install puppet
```

```
sudo vi /etc/default/puppet
```

And add a single line `START=yes`

### Update /etc/hosts

Get the IP address from the puppetmaster `ip a` and make an entry into `/etc/hosts` in the agent

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.121.126     puppetmaster
```

### Make agent certifcate request


Start the server again

Run puppet agent once to create a certificate request

```
sudo puppet master --verbose --no-daemonize
```

And run the client

```
sudo puppet agent --noop --server=puppet.example.org
```

On the first connection you will see on the agent:

`Exiting; no certificate found and waitforcert is disabled`


On the first connection you will see on the master:

`Notice: puppetagent has a waiting certificate request`

Now run `sudo pupper cert list --all` and you will see the agent certificate:

```
sudo puppet cert list --all
  "puppetagent"  (SHA256) 78:6B:16:AA:64:10:FC:4F:4E:99:98:CF:80:B0:6D:1B:D0:0D:B8:2D:63:13:FB:22:87:1D:C8:06:22:7B:0A:85
    + "puppetmaster" (SHA256) 12:00:63:FF:A9:4D:48:E0:EF:78:A6:EC:75:FC:AD:BC:F5:F0:9A:C4:63:AC:61:B2:AE:39:80:96:57:A6:A6:60 (alt names: "DNS:puppetmaster")
```

Note: The `+` signifies that the certicate is signed (so the server does not need to sign its own certicate)

We do however need to sign the agents certifcate

```
 sudo puppet cert sign  puppetagent
 Notice: Signed certificate request for puppetagent
 Notice: Removing file Puppet::SSL::CertificateRequest puppetagent at '/var/lib/puppet/ssl/ca/requests/puppetagent.pem'
```

You're now ready to start your puppet module development. Just remember to run `puppet agent -t --verbose --server=puppetmaster` each time and if your puppet module changes root owned files, then you will need to run `puppet agent` with `sudo`.

## Some tips

The puppet apply command allows you to execute manifests that are not related to the main manifest, ondemand. It only applies the manifest to the node that you run the apply from. Here is an example:

`sudo puppet apply /etc/puppet/modules/test/init.pp`

## A simple test

sudo vi /etc/puppet/manifests/site.pp

Start your server `sudo puppet master --verbose --no-daemonize` and run the agent `puppet agent -t --server=puppetmaster`

```
puppetagent ~]$ cat /tmp/example-ip
Here is my Public IP Address: 192.168.121.82
```

## Back up

Its worth snapshotting the images, in case you want to roll back to an unused instance

`sudo virsh snapshot-create-as --domain master_default --name "base-puppet-master"`

`sudo virsh snapshot-create-as --domain agent_default --name "base-puppet-agent"`

To restore:

virsh snapshot-revert --domain <vm-name> <snapshot-name>

