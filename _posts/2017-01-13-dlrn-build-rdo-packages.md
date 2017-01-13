<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#orgc1b85f8">1. Prepare Environment</a></li>
<li><a href="#org54828c9">2. Add user to mock group</a></li>
<li><a href="#org81b756e">3. Install DLRN</a>
<ul>
<li><a href="#org0e19184">3.1. dlrn.cfg</a></li>
<li><a href="#orgc6a82dc">3.2. Install VirtualEnv and DLRN Dependencies</a></li>
</ul>
</li>
<li><a href="#org9335018">4. Set up project and spec file</a>
<ul>
<li><a href="#orge9d2440">4.1. rdo.yaml</a></li>
<li><a href="#orgd37cadd">4.2. Spec file</a></li>
</ul>
</li>
<li><a href="#org091449f">5. Final Report</a></li>
<li><a href="#orgde81755">6. Submitting a patch</a>
<ul>
<li><a href="#orge84e67a">6.1. Create a BZ</a></li>
<li><a href="#orgefa38e8">6.2. Submit your patch to review.rdoproject.org</a></li>
</ul>
</li>
<li><a href="#org654e889">7. Billy Bonus - using pyp2rpm</a></li>
</ul>
</div>
</div>


<a id="orgc1b85f8"></a>

# Prepare Environment

    dnf install git createrepo python-virtualenv mock gcc \
                  redhat-rpm-config rpmdevtools httpd libffi-devel \
                  openssl-devel mock


<a id="org54828c9"></a>

# Add user to mock group

    usermod -a -G mock [user name]


<a id="org81b756e"></a>

# Install DLRN

-   Create a master directory to keep your tools in `mkdir ~/buildtools`
-   Clone the **DLRN** repository `cd ~/buildtools; git clone https://github.com/openstack-packages/DLRN`
-   Clone **RDOINFO** repository `git clone https://review.rdoproject.org/r/rdoinfo`


<a id="org0e19184"></a>

## dlrn.cfg

Grab a dlrn.cfg (no need for any customizations)

    wget https://gist.githubusercontent.com/lukehinds/856cb55b8f5b480d9e641f8812749178/raw/97f813e30c89090247e5eb4d5bdca2c087b377a1/dlrn.cfg

    sudo mv dlrn.cfg /etc/mock


<a id="orgc6a82dc"></a>

## Install VirtualEnv and DLRN Dependencies

Create a virtualenv (run these commands from the `DLRN` directory:

    virtualenv env

Source your new env:

    source env/bin/activate

**Note:** Everytime you want to run dlrn, do the above first when opening a new shell instance.

Install PIP requirements

    pip install -r requirements.txt

    pip install -r test-requireiments.txt # might not be needed, but does not hurt!

Run setuptools

    python setup.py install


<a id="org9335018"></a>

# Set up project and spec file


<a id="orge9d2440"></a>

## rdo.yaml

Edit the `rdotools/rdo.yml` file in the repo and add a block like this for your project:

    - project: networking-bagpipe
      name: python-networking-bagpipe
      tags:
        under-review:
        #ocata:
        #ocata-uc:
      conf: rpmfactory-lib
      upstream: https://github.com/openstack/networking-bagpipe
      maintainers:
        - lhinds@redhat.com

`rpmfactory-lib` is for python projects e.g. **python-networking-bagpipe**

`rpmfactory-core` is for a main openstack project e.g. **murano**

`rpmfactory-client` is for a CLI client, e.g. **neutronclient**


<a id="orgd37cadd"></a>

## Spec file

Create a directory `DLRN/data/{openstack:python}-networking-{project}_distro` for your spec file.

**For example**

    DLRN/data/python-networking-bagpipe_distro

**Note:** I'ts important that the `_distro` string of your spec files directory contains an underscore.

Copy / Move your spec file into the `_distro` directory referenced above.

[Here](https://raw.githubusercontent.com/lukehinds/rdo-packaging/master/python-networking-bagpipe/SPEC/python3-networking-bagpipe.spec) is an example spec file used for python-networking-bagpipe

Once your spec file is ready, you can test the build with the following command-line:

    dlrn --config-file projects.ini --package-name python-networking-bagpipe --dev --info-repo <path to the rdoinfo directory>

**For example:**

    dlrn --config-file projects.ini --package-name python-networking-bagpipe --dev --info-repo ../rdoinfo

**Note:** The following error can be ignored:

\#+BEGIN<sub>SRC</sub> bash


<a id="org091449f"></a>

# Final Report

Some people set up an Apache `VirtualHost` to view reports etc, but I honestly find your browser handles File Directories perfectly ok.

    file:///home/<USER>/buildtools/DLRN/data/repos/report.html


<a id="orgde81755"></a>

# Submitting a patch

There are two things you have to do to submit a patch


<a id="orge84e67a"></a>

## Create a BZ

You first need to create a bugzilla in which you reference your spec file and source RPM:

[Here is an example](https://bugzilla.redhat.com/show_bug.cgi?id=1379646)


<a id="orgefa38e8"></a>

## Submit your patch to review.rdoproject.org

-   Get your repository set up (same as review.openstack.org) using `cd ~/buildtools/rdoinfo; git review -s`

-   Create a branch `git checkout -b networking-bagpipe`

-   Make you're changes to `rdo.yml`

-   Add them.. `git add rdo.yml`

-   commit the changes.. `git commit -a`

Note, the commit message needs to be of the following format:

    Adding <PACKAGE NAME> package

    <PACKAGE DESCRIPTION>

    BZ Review: <BUGZILLA URL>

    Initially set under-review until distgit import is complete

**For example**

    Adding python-networking-bagpipe package

    Mechanism driver for Neutron ML2 plugin using BGP VPNs

    BZ review: https://bugzilla.redhat.com/show_bug.cgi?id=1379646

    Initially set under-review until distgit import is complete

You can then submit for review `git review`


<a id="org654e889"></a>

# Billy Bonus - using pyp2rpm

Pyprpm can be used to create a spec file from pypi package

For Example:

<https://pypi.python.org/pypi/openstackdocstheme>

    git clone https://github.com/fedora-python/pyp2rpm; cd pyp2rpm

    virtualenv env
    source env/bin/activate
    pip install -r requirements.txt

    mkdir -p ~/buildtools/myspecs/

    pyp2rpm openstackdocstheme > ~/rpmbuild/myspecs/openstackdocstheme.spec

