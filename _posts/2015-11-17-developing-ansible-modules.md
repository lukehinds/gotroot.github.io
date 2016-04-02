---
layout: post
title: Ansible Module Development
subtitle: Using the Ansible Python API
bigimg: /img/header_tech_2.jpg
---

So I have recently being writing some middleware code on top of the very powerful Ansible Python API.

Ansible for those that don't know is an automation framework, that facilitates remote machine configuration, via a set of templates called 'playbooks'. To quote Wikipedia;

Ansible is a free software platform for configuring and managing computers. It combines multi-node software deployment, ad hoc task execution, and configuration management. It manages nodes over SSH or PowerShell and requires Python (2.4 or later) to be installed on them'.

# SO WHY IS ANSIBLE SO GOOD?

What made Ansible a winner for me, is the ability to craft elaborate modules (remote scripts) and have ansible execute them, coupled with the fact that ansible puts no requirements on the managed nodes. There is no need to enroll them as minions or install a client of some sort, all remote execution is native to OpenSSH or python's paramiko.

I don't want to go into giving a full blown overview of ansible, but to cast some background, I will round up some key areas. I expect if you came here through google, you know Ansible well already and are seeking to extend Ansible with your own code / modules, still I will do a quick overview for those completely green. Those wanting to skip this, can go direct to 'Module Development'

THE QUICK AND DIRTY ON ANSIBLE

![ansible](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/ansible.png)

Image source http://sysadmincasts.com

# ANSIBLE MANAGEMENT NODE

The management node is essentially any machine, that has python 2.6 or greater and the ansible software. This could be an engineer's laptop, a vagrant image, or docker container, raspberry pi etc. The main requirement, is that the node has the ability to connect to the machines that it is designated to manage. This is the server if you like, the central point that distributes the automation calls to all target nodes.

## INVENTORY

The inventory, is where we list our hosts that you wish to manage via ansible,

~~~
[lab]
node1.acmetest
node2.acmetest
node3.acmetest
node4.acmetest
~~~

## PLAYBOOKS

Playbooks are the recipes written in YAML that are used to inform Ansible of the modules you wish to call, alongside the metadata required. A good example here would be an apache installation, this can be achieved with just the following playbook

~~~
– hosts: all
user: root
sudo: no
tasks:
– name: install apache2
action: apt pkg=apache2 state=latest
notify:
– start apache2
– name: Make sure index.html is present for the default virtual host
action: copy src=files/index.html dest=/var/www/index.html
– name: Make sure index.html is owned by www-data
action: file path=/var/www/index.html owner=www-data group=www-data
handlers:
– name: start apache2
action: service name=apache2 state=started
~~~

*Playbook from lowendbox.com*

Simple enough to follow, but essentially;

1. Apply this to all hosts
2. Run as root
3. Install the apache2 package (using apt in this example, so debian, but yum / dnf is supported as well)
4. Copy over a index.html and chown / chmod the file.
5. Finally bounce apache (systemctl restart apache2)

This playbook would be enacted against all nodes listed with the Inventory host file. If prefered, you could run it against a subset, but using a header:

For example – 'hosts: lab' would only run hosts node1.acmetest to node4.acmetest

~~~
[lab]
node1.acmetest
node2.acmetest
node3.acmetest
node4.acmetest

[production]
node1.acmeprod
node2.acmeprod
node3.acmeprod
node4.acmeprod
~~~

Further to playbooks, you can also create 'roles' - which essentially allow you to group server types, a good example here could be 'webserver' 'databaseservers' . I won't go further into roles, so recommend anyone wanting to learn more ansible does so by visiting the official ansible docs page.

# MODULES

Modules are the low level task masters of ansible, and ansible ships with a number of included modules (called the ‘module library’). Modules can be executed directly on remote hosts via Playbooks or using the Python API (which is what we will do).

Modules are no more than scripts which are remotely piped down an SSH connection to the target node. The script is executed and the stdout of that script, is piped back within some JSON to core ansible. From their the json can be parsed as a python dictionary, allowing Ansible to understand success criteria / response codes.

Most modules are developed in python, but there is nothing to stop you using any language, as long as its on the target system and it supports standard output. For example, you could develop modules in bash or perl.

# MODULE DEVELOPMENT

The Ansible team recently made Module development a little easier by allowing you to inherit from a class  which takes care of the logistics of reporting back to core.

For those that did not already know, the ansible and ansible-playbook commands are actually python scripts sitting on top of a core Ansible. This means that near on all functions available in ansible, there is an associated API that can be harnessed in your own scripts. We will be developing on top of the same API as the aforementioned ansible commands.

This is very useful if you need ansibles remote SSH execution framework, but wish to invoke this process via another system, or embed its workflows within your own application.

For this post, we will create a simple module that will provision an OpenStack instance, using the nova client. This way we will need to handle lots of arguments, wait for the process to complete, and return a status.

## ENVIRONMENT

To get an environment set up, we have two options, use virtualenv to encapsulate or install ansible and its requirements, system wide. If you're prototyping, I recommend you contain everything by creating a virtual environment.

Create a project directory and a module directory.

~~~
mkdir -p ~/ansible-project/ansible-modules && cd $_
~~~

All we need to do, is install ansible (and pip will look after the dependencies)

~~~
pip install ansible
~~~

Alongside ansible, the following dependent libraries will be installed

~~~
MarkupSafe
PyYAML
ecdsa
jinja
paramiko
pycrypto
~~~

Now we need to setup an ansible.cfg, as we want our scripts to not get destroyed after running and we don't want the key trust to nag us. Obviously, this is not good for production, but it's forgivable for now, as this is only for a development environment. Anything more exposed should use ssh keys and have key host checking on.

~~~
[defaults]
host_key_checking = False
library = /home/you/ansible-project/ansible-modules
keep_remote_files = True
~~~

Next we are going to set up a hosts file. The actual official way of doing this is by using an inventory, but I personally don't like this method, as it limits me , as only ansible knows which node it runs on, still if you want the official method, then it will be:

~~~
example_inventory = ansible.inventory.Inventory(hosts)
~~~

You then need a hosts file in the ansible format. I prefer instead to use the following simple format

~~~
computenode1.mydomain
computenode2.mydomain
~~~

I then iterate my hosts using a NamedTemporaryFile and pass this as an inventory value - we will see this latter. Props to billwanjohi for this method.

```
def load_temporary_inventory(content):
    tmpfile = NamedTemporaryFile()
    try:
        tmpfile.write(content)
        tmpfile.seek(0)
        inventory = Inventory(tmpfile.name)
    finally:
        tmpfile.close()
    return inventory
```

So let's get on with making our module.

We are going to use a couple of helper libraries, mainly ConfigParser, as I am too lazy to type out long command line args. We will also use json to parse the return data.

The idea is we will provision a cirros instance and get the build status back. Of course this is not rocket science, and I will be doing a few shortcuts such as hard coding usernames and passwords, but this can easily be changed by using ssh keys or getpass instead. I am more about demonstrating what ansible can do.

So let's look at the full module, and then break down the key parts.

First we have are config.ini

~~~
[config]
username = <your_openstack_tenant_name>
password = <your_password>
projectname = yourproject
name = Ansible Rocks My World!
image = cirros
flavor = m1.tiny
network = web-tier
ip = 192.168.10.20
keyname = luke
~~~

And our module, which we will call openstack_module (note the lack of .py extension!) -  you need to save this into ~/ansible-project/ansible-modules/

```
#!/usr/bin/python

import os
import time
from keystoneclient.auth.identity import v2
from keystoneclient import session
from novaclient import client


def main():
    module = AnsibleModule(
        argument_spec = dict(
            arg1=dict(required=False, default=None),
            arg2=dict(required=False, default=None),
            arg3=dict(required=False, default=None),
            arg4=dict(required=False, default=None),
            arg5=dict(required=False, default=None),
            arg6=dict(required=False, default=None),
            arg7=dict(required=False, default=None),
            arg8=dict(required=False, default=None),
            arg9=dict(required=False, default=None),
        ),
        supports_check_mode = True
    )

    # global vars
    username = module.params['arg1']
    password = module.params['arg2']
    tenantname = module.params['arg3']
    name = module.params['arg4']
    image = module.params['arg5']
    flavor = module.params['arg6']
    network = module.params['arg7']
    ip = module.params['arg8']
    keyname = module.params['arg9']

    auth = v2.Password(auth_url='http://127.0.0.1:5000/v2.0',
                       username=username,
                       password=password,
                       tenant_name=tenantname)

    sess = session.Session(auth=auth)
    nova = client.Client(2, session=sess)

    image = nova.images.find(name=image)
    flavor = nova.flavors.find(name=flavor)
    network = nova.networks.find(label=network)
    nics = [{'net-id': network.id,'v4-fixed-ip':ip}]

    instance = nova.servers.create(name=name, image=image, flavor=flavor, nics=nics, key_name=keyname)

    # Poll at 5 second intervals, until the status is no longer 'BUILD'
    status = instance.status
    while status != 'ACTIVE':
        time.sleep(1)
        # Retrieve the instance again so the status field updates
        instance = nova.servers.get(instance.id)
        status = instance.status
    module.exit_json(status=status)

from ansible.module_utils.basic import *

if __name__ == '__main__':
    main()
```

So essentially we are accepting the arguments passed to this module (we look at the initiating script next) and we then use those args to pass to nova to create the instance. Finally we return the status 'Active' in the json piped back to ansibles core.

module.exit_json(status=status)
Let's now take a look at the python script which calls the ansible API to kick off our new module.

Save this script into ~/ansible-project

```
import os
import json
import ansible.runner
import ansible.playbook
import ansible.inventory
from ansible.inventory import Inventory
from ConfigParser import SafeConfigParser
from tempfile import NamedTemporaryFile

parser = SafeConfigParser()
parser.read('config.ini')

username = parser.get('config', 'username')
password = parser.get('config', 'password')
projectname = parser.get('config', 'projectname')
name = parser.get('config', 'name')
image = parser.get('config', 'image')
flavor = parser.get('config', 'flavor')
network = parser.get('config', 'network')
ip = parser.get('config', 'ip')
keyname = parser.get('config', 'keyname')


def init_script():
    if not os.path.isfile('hosts'):
        print "Host File does not exist"
    with open("hosts", "r") as f:
        lines = f.read().splitlines()
        for hostname in lines:
            ansible_function(hostname)


def load_temporary_inventory(content):
    tmpfile = NamedTemporaryFile()
    try:
        tmpfile.write(content)
        tmpfile.seek(0)
        inventory = Inventory(tmpfile.name)
    finally:
        tmpfile.close()
    return inventory


def ansible_function(hostname):
    runner = ansible.runner.Runner(
        inventory=load_temporary_inventory(hostname),
        module_name='openstack_module',
        module_args='arg1=%s arg2=%s arg3=%s arg4=%s arg5=%s arg6=%s arg7=%s arg8=%s arg9=%s' \
        % (username, password, projectname, name, image, flavor, network, ip, keyname),
        remote_user=username,
        remote_pass=password # - As above
        timeout=5,
    )
    out = runner.run()
    print json.dumps(out, sort_keys=True, indent=4, separators=(',', ': '))

if __name__ == '__main__':
    init_script()
```

The main aspect to our calling script, is the runner class 'ansible.runner' inherited from the ansible API, this allows us to nominate a module arguments, and note how we use the load_temporary_inventory, to workaround the ansible inventory class.

Last of all we see that the 'Active' state returns when the cirros instance is created:

term

And finally our running instance..

openstack

There we have it. Hopefully that gives you some ideas of how powerful and extendable ansible is!
