---
layout: post
title: Building the lastest version of OVS and mininet within CentOS 7
subtitle: Quick run through
bigimg: /img/path.jpg
---

Set to SElinux to Permissive (obviously this is only for a test machine, if you're doing anything in production	, leave it enforcing and use the OVS in the CentOS openstack repos)

~~~
sudo setenforce Permissive
~~~

Grab yourself git:

~~~
sudo yum -y install git
~~~

Clone mininet:

~~~
git clone git://github.com/mininet/mininet.git
~~~

Edit  the install script:

~~~
vim mininet/util/install.sh
~~~

Add a section for CentOS (you will see the existing sections for Rhel, Debian and Fedora)

~~~
test -e /etc/centos-release && DIST="CentOS"
if [ "$DIST" = "CentOS" ]; then
    install='sudo yum -y install'
    remove='sudo yum -y erase'
    pkginst='sudo rpm -ivh'
    # Prereqs for this script
    if ! which lsb_release &> /dev/null; then
        $install redhat-lsb-core
    fi
fi
~~~

~~~
if ! echo $DIST | egrep 'Ubuntu|Debian|Fedora|CentOS'; then
    echo "Install.sh currently only supports Ubuntu, Debian and Fedora."
    exit 1
fi
~~~

~~~
mininet/util/install.sh -nf
~~~

# Install OVS

Get the depencies needed to build OVS

~~~
sudo yum -y install gcc make python-devel openssl-devel kernel-devel graphviz \
kernel-debug-devel autoconf automake rpm-build redhat-rpm-config \
libtool wget
~~~

~~~
mkdir -p ~/rpmbuild/SOURCES/
cd ~/rpmbuild/SOURCES/
wget http://openvswitch.org/releases/openvswitch-2.3.0.tar.gz
tar zxvf openvswitch-2.3.0.tar.gz
cd openvswitch-2.3.0
rpmbuild -bb --without check rhel/openvswitch.spec
sudo rpm -ivh --nodeps ~/rpmbuild/RPMS/x86_64/openvswitch*.rpm
~~~

Example Run:

~~~
[luke@localhost openvswitch-2.5.0]$ sudo rpm -ivh --nodeps ~/rpmbuild/RPMS/x86_64/openvswitch*.rpm
[sudo] password for luke:
Preparing...                          ################################# [100%]
Updating / installing...
   1:openvswitch-debuginfo-2.5.0-1    ################################# [ 50%]
   2:openvswitch-2.5.0-1              ################################# [100%]
~~~

Start OVS:

~~~
sudo /etc/init.d/openvswitch start
~~~

Check its working OK

~~~
[luke@localhost openvswitch-2.5.0]$ sudo ovs-vsctl show
0e034fda-7d8d-4f95-827a-22678c865254
    ovs_version: "2.5.0"
~~~

# Run mininet

And now lets test mininet:

~~~
[luke@localhost ~]$ sudo mn --test pingall
*** Creating network
*** Adding controller
*** Adding hosts:
h1 h2
*** Adding switches:
s1
*** Adding links:
(h1, s1) (h2, s1)
*** Configuring hosts
h1 h2
*** Starting controller
c0
*** Starting 1 switches
s1 ...
*** Waiting for switches to connect
s1
*** Ping: testing ping reachability
h1 -> h2
h2 -> h1
*** Results: 0% dropped (2/2 received)
*** Stopping 1 controllers
c0
*** Stopping 2 links
..
*** Stopping 1 switches
s1
*** Stopping 2 hosts
h1 h2
*** Done
completed in 5.489 seconds
~~~
