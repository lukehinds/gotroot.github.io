---
layout: post
title: DLRN for RDO package builds
---

What follows are the steps to create a spec file, test that spec file in a local
dlrn instance, and then submit the file as a patch to rdo.review.

## Prepare Environment

    dnf install git \
            createrepo python-virtualenv mock gcc \
            redhat-rpm-config rpmdevtools httpd libffi-devel \
            openssl-devel mock

### Add user to mock group

    usermod -a -G mock [user name]


### Install DLRN

-   Create a master directory to keep your tools in `mkdir ~/buildtools`
-   Clone the **DLRN** repository `cd ~/buildtools; git clone https://github.com/openstack-packages/DLRN`
-   Clone **RDOINFO** repository `git clone https://review.rdoproject.org/r/rdoinfo`


### dlrn.cfg

Grab a dlrn.cfg (no need for any customizations)

```
wget https://gist.githubusercontent.com/lukehinds/856cb55b8f5b480d9e641f8812749178/raw/97f813e30c89090247e5eb4d5bdca2c087b377a1/dlrn.cfg

sudo mv dlrn.cfg /etc/mock
```


### Install VirtualEnv and DLRN Dependencies

Create a virtualenv (run these commands from the `DLRN` directory:

```
virtualenv env
```

### Source your new env:

```
source env/bin/activate
```

**Note:** Everytime you want to run dlrn, do the above first when opening a new shell instance.

Install PIP requirements

```
pip install -r requirements.txt

pip install -r test-requireiments.txt # might not be needed, but does not hurt!
```

Run setuptools

    python setup.py install


## Set up project and spec file

### rdo.yaml

Edit the `rdotools/rdo.yml` file in the repo and add a block like this for your project:

{% highlight yaml %}
    - project: myproject
      name: python-myproject
      tags:
        under-review:
        #ocata:
        #ocata-uc:
      conf: rpmfactory-lib
      upstream: https://github.com/openstack/myproject
      maintainers:
        - someguy@domain.com
{% endhighlight %}

`rpmfactory-lib` is for python projects e.g. **python-myproject**

`rpmfactory-core` is for a main openstack project e.g. **murano**

`rpmfactory-client` is for a CLI client, e.g. **neutronclient**


### Spec file

Create a directory `DLRN/data/{openstack:python}-networking-{project}_distro` for your spec file.

**For example**

    DLRN/data/python-myproject_distro

**Note:** I'ts important that the `_distro` string of your spec files directory contains an underscore.

Copy / Move your spec file into the `_distro` directory referenced above.

[Here](https://raw.githubusercontent.com/lukehinds/rdo-packaging/master/python-networking-bagpipe/SPEC/python3-networking-bagpipe.spec) is an example spec file used for project python-networking-bagpipe

Once your spec file is ready, you can test the build with the following command-line:

    dlrn --config-file projects.ini --package-name python-myproject --dev --info-repo <path to the rdoinfo directory>

**For example:**

    dlrn --config-file projects.ini --package-name python-myproject --dev --info-repo ../rdoinfo

## Final Report

Some people set up an Apache `VirtualHost` to view reports etc, but I honestly find your browser handles File Directories perfectly ok.

    file:///home/<USER>/buildtools/DLRN/data/repos/report.html


## Billy Bonus - using pyp2rpm

Pyprpm can be used to create a spec file from pypi package

For Example:

<https://pypi.python.org/pypi/openstackdocstheme>

    git clone https://github.com/fedora-python/pyp2rpm; cd pyp2rpm

    virtualenv env
    source env/bin/activate
    pip install -r requirements.txt

    mkdir -p ~/buildtools/myspecs/

    pyp2rpm openstackdocstheme > ~/rpmbuild/myspecs/openstackdocstheme.spec

