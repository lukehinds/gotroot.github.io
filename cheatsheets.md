---
layout: page
title: Cheatsheets
subtitle: Stuff I always forget.
---

# Table of Contents
[virsh](#virsh)  
[neutron](#neutron)


## virsh

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

### I want out

~~~
virsh quit   # Leave CLI
~~~


# neutron

### Crete Network

~~~
neutron net-create NAME
~~~

### Create a subnetwork

~~~
neutron subnet-create NETWORK_NAME CIDR
neutron subnet-create my-network 10.0.0.0/29
~~~
