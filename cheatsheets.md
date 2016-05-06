---
layout: page
title: Cheatsheets
subtitle: Stuff I always forget.
---
* Test
{:toc}

Add some of these:

https://raymii.org/s/articles/virt-install_introduction_and_copy_paste_distro_install_commands.html


# Virtualization

## Domains

### List domains

~~~
virsh list                # List running
virsh list --all          # List all
~~~

### Control domains

~~~
virsh start <domain>
virsh shutdown <domain>
virsh destroy <domain>
virsh suspend <domain>
virsh resume <domain>
~~~

### Define domains

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

## Snapshots

### list Snapshots

~~~
virsh snapshot-list
~~~

### Restore Snapshot

~~~
virsh snapshot-revert --domain undercloud <snapshot-name>
~~~

### Delete Snapshots

~~~
virsh snapshot-delete domain snapshot
~~~

## Networks

### List Networks

~~~
virsh net-list
~~~

### Inspect Networks (XML)

~~~
virsh net-dumpxml NETWORK
~~~

### Define Network

~~~
host# cat > /tmp/provisioning.xml <<EOF
<network>
  <name>provisioning</name>
  <ip address="172.16.0.254" netmask="255.255.255.0"/>
</network>
EOF
~~~

### Edit Network

~~~
virsh  net-edit NETWORK
~~~

Note: after changing network xml schema, you need to recreate the network:

~~~
virsh net-destroy NETWORK
virsh net-start NETWORK
~~~

### Get IP Address of Domains

~~~
virsh domifaddr <domain> --full
~~~

~~~
virsh net-dhcp-leases --network default
~~~

## virt-customize

### Change password

~~~
virt-customize -a shibboleth.qcow2 --root-password password:p6ssw0rd
~~~

### Run Command

~~~
virt-customize -a <domain>.qcow2 --run-command '<command>'
~~~

#### Examples:

~~~
virt-customize -a shibboleth.qcow2 --run-command 'yum remove cloud-init* -y'
~~~

~~~
virt-customize -a shibboleth.qcow2 --run-command 'sed -i s/^PermitRootLogin.*/PermitRootLogin\ yes/ /etc/ssh/sshd_config'
~~~

~~~
virt-customize -a undercloud.qcow2 --run-command 'cp /etc/sysconfig/network-scripts/ifcfg-eth{0,1} && sed -i s/DEVICE=.*/DEVICE=eth1/g /etc/sysconfig/network-scripts/ifcfg-eth1'
~~~

## qemu-img

### Get Image Info

~~~
qemu-img info image.qcow2
~~~

### Create new image

~~~
qemu-img create -f qcow2 image.qcow2 40G
~~~


## virt-install

~~~
virt-install --ram 16384 --vcpus 4 --os-variant rhel7 \
    --disk path=/var/lib/libvirt/images/image.qcow2,device=disk,bus=virtio,format=qcow2 \
    --import --noautoconsole --vnc --network network:provisioning \
    --network network:default --name myvirtualmachine
~~~


## virt-filesystems

### Inspect partition size / info

~~~
virt-filesystems --long -h --all -a image.qcow2
~~~

# OpenStack

## Inspect endpoints

~~~
openstack catalog show <project>
~~~

*example:*

~~~
$ openstack catalog show neutron
+-----------+----------------------------------------+
| Field     | Value                                  |
+-----------+----------------------------------------+
| endpoints | regionOne                              |
|           |   publicURL: http://172.16.0.1:9696/   |
|           |   internalURL: http://172.16.0.1:9696/ |
|           |   adminURL: http://172.16.0.1:9696/    |
|           |                                        |
| name      | neutron                                |
| type      | network                                |
+-----------+----------------------------------------+
~~~

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

## Keystone

### API Examples

~~~
curl -i \
  -H "Content-Type: application/json" \
  -d '
{ "auth": {
    "identity": {
      "methods": ["password"],
      "password": {
        "user": {
          "name": "admin",
          "domain": { "id": "default" },
          "password": "secret"
        }
      }
    }
  }
}' http://localhost:35357/v3/auth/tokens ; echo
~~~

~~~
curl -i \
  -H "Content-Type: application/json" \
  -d '
{ "auth": {
    "identity": {
      "methods": ["password"],
      "password": {
        "user": {
          "name": "user",
          "domain": { "id": "default" },
          "password": "secret"
        }
      }
    }
  }
}' http://localhost:5000/v3/auth/tokens ; echo
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

*Use the -1 modifier; to see boot messages from two boots ago, use -2; and so on:*

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

## Facter

Collect and display facts about the system, can make returns formatted in json.

### install Facter

~~~
yum / dnf install facter - y
~~~

### Examples:

~~~
$ facter
           architecture => amd64
           blockdevices => sda,sr0
           domain => example.com
           fqdn => puppet.example.com
           hardwaremodel => x86_64
           [...]
~~~


~~~
$ facter --json architecture kernel hardwaremodel
{
  "architecture": "amd64",
  "kernel": "Linux",
  "hardwaremodel": "x86_64"
}
~~~

~~~
$ ipaddr=$(facter ipaddress_eth1)
$ echo $ipaddr
192.168.122.253
~~~

## gpg

~~~
gpg --list-keys
gpg --list-secret-keys
~~~

## Handy CLI

### Show config file entries, minus comments ('#')

~~~
egrep -v '^#|^$' file.conf
~~~

# Misc Stuff

## NCMCPP shotcuts

~~~
Up k : Move Cursor up
Down j : Move Cursor down
Page Up : Page up
Page Down : Page down
Home : Home
End : End
Tab : Switch between playlist and browser
1 F1 : Help screen
2 F2 : Playlist screen
3 F3 : Browse screen
4 F4 : Search engine
5 F5 : Media library
6 F6 : Playlist editor
7 F7 : Tag editor
0 F10 : Clock screen

Keys - Global
-----------------------------------------
s : Stop
P : Pause
> : Next track
< : Previous track
f : Seek forward
b : Seek backward
Left - : Decrease volume
Right + : Increase volume
t : Toggle space mode (select/add)
T : Toggle add mode
| : Toggle mouse support
v : Reverse selection
V : Deselect all items
A : Add selected items to playlist/m3u file
r : Toggle repeat mode
Z : Shuffle playlist
i : Show song's info
I : Show artist's info
L : Toggle lyrics database
l : Show/hide song's lyrics
q Q : Quit
~~~
