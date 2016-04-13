---
layout: page
title: Cheatsheets
subtitle: Stuff I always forget.
---
* Test
{:toc}

# virsh

### Domains

### List domains

~~~
virsh list                # List running
virsh list --all          # List all
~~~

### Control instances

~~~
virsh start <instance>
virsh shutdown <instance>
virsh destroy <instance>
virsh suspend <instance>
virsh resume <instance>
~~~

### Define instances

~~~
virsh dumpxml <instance> >dump.xml
virsh create dump.xml   # Create from XML
virsh edit <instance>
virsh undefine <instance>
~~~

### Resize block device

~~~
virsh blockresize <instance> --path vda --size 100G
~~~

### Get Info

~~~
virsh dominfo
virsh vcpuinfo
virsh nodeinfo
~~~

### Snapshots

#### list Snapshots

~~~
virsh snapshot-list
~~~

#### Delete Snapshots

~~~
virsh snapshot-delete domain snapshot
~~~

### Networks

#### List Networks

~~~
virsh net-list
~~~

#### Inspect Networks (XML)

~~~
virsh net-dumpxml NETWORK
~~~

#### Define Network

~~~
host# cat > /tmp/provisioning.xml <<EOF
<network>
  <name>provisioning</name>
  <ip address="172.16.0.254" netmask="255.255.255.0"/>
</network>
EOF
~~~

# OpenStack

## TripleO

### baremetal introspection status (dhcp)

~~~
sudo journalctl -l -u openstack-ironic-discoverd \
    -u openstack-ironic-discoverd-dnsmasq -f
~~~

## Neutron

### Create Network

~~~
neutron net-create NAME
~~~

### Create a subnetwork

~~~
neutron subnet-create NETWORK_NAME CIDR
neutron subnet-create my-network 10.0.0.0/29
~~~

# General Linux Admin

## journalctl

### Boot messages

~~~
journalctl -b
~~~

### List boots

~~~
journalctl --list-boots
~~~

Use the -1 modifier; to see boot messages from two boots ago, use -2; and so on:

~~~
journalctl -b -1
~~~

### Time Ranges

~~~
journalctl --since "1 hour ago"
~~~

~~~
journalctl --since "2 days ago"
~~~

~~~
journalctl --since "2015-06-26 23:15:00" --until "2015-06-26 23:20:00"
~~~

## gpg

~~~
gpg --list-keys
gpg --list-secret-keys
~~~
