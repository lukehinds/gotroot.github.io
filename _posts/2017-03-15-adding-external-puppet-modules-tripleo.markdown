---
layout: post
title: Adding External Puppet Modules to TripleO
tags:
  nfvpe
---

Sometimes its useful to use existing puppet-modules, when developing
a new composable service for TripleO.

Using existing modules is often worth while, as many community modules
on puppet-forge are well established and have been tested and used by
countless operators.

## Selecting a module
First thing to do is search https://forge.puppet.com for the area you
need, so for example 'ssh', 'ntp' etc.

When looking for modules, I tend to use the following guidelines to help
me make a selection.

1. Is it supported?

Some modules haver a banner stating 'supported'. These modules are
earmarked by puppet as they have the following attributes:

* Rigorously tested with Puppet Enterprise
* Supported with PE by Puppet, Inc
* Maintained for the same lifecycle as Puppet Enterprise
* Compatible with multiple platforms

A supported module is a good indicator, but if you cannot find one, then
the following checks will help you sort further.

1. Is the module maintained?

Check that the module is actively maintained. For example, when was the
last release made? How many outstanding issues are there on the github
repository? Does the maintainer accept pull requests that improve the
module?  Do they engage with the community in helpful manner (important
as you may need to communicate with them, if issues are found).

2. Do they unit test?

A good puppet module, will have an CI driven unit tests. Typically it
will run using travis, whereby any pull request runs a series of unit
tests against different puppet and ruby versions. For TripleO its
important they test against not only Puppet 3, but also 4 which is
becoming the next supported version in puppet Openstack. There should
also be tests specific to the os.family to make sure a change for one
Linux distribution, does not break others.

3. Quality score and Community Rating.

A decent puppet module should have a 5 score and a good community
rating, be mindful though that the community rating might sometimes not
always be a good indicator. This signifies a good coverage of Puppet
version Compatibility, and decent metadata available to show which
operating systems are supported, package dependencies etc.

4. Does it support Hiera data?

This is key. I found some Puppet Modules are not able to handle Hiera,
which is a key requirement for OpenStack puppet modules.

There is of course other indicators, such as code quality, but thats
a topic going beyond this post, and to be truthful I am still new to
Puppet myself, so I don't feel right acting as an authority here.

5. Test it out!

Fire up a VM / Container and test it out. See how it works, and if you
like the design.

## Integrating into TripleO and RDO Trunk.

This took me a few steps to get right and the information out their is
non existant from what I could find.



Various values have been set to the secure default in Horizon.

### Enforce Password Check

By setting `ENFORCE_PASSWORD_CHECK` to `True` within Horizons
`local_settings.py`, it displays an 'Admin Password' field on the
"Change Password" form to verify that it is the admin logged-in that wants
to perform the password change, and not an opportunist who has found their workstation
unlocked.

The `ENFORCE_PASSWORD_CHECK` value can now be toggled via puppet as seen in the
following commit to [puppet-horizon](https://github.com/openstack/puppet-horizon/commit/14cc9e89ab29b8c8bfc4e7664ff786a35253eee2).

Within TripleO Heat templates, we populate the hiera data fed to puppet-horizon
with the `True` boolean, as seen in [this
patch](https://github.com/openstack/tripleo-heat-templates/commit/ca122325ddb9b17c75bc3050ff17ed71fd273e7e).

By utlising TripleO Heat templates, it then becomes possible to toggle this
value to false (should someone have a reason to do that) using an enviroment
file and passing in a value of `horizon::enforce_password_check: false`

### Disallow Iframe Embed

DISALLOW_IFRAME_EMBED can be used to prevent Horizon from being embedded
within an iframe. Legacy browsers are still vulnerable to a Cross-Frame
Scripting (XFS) vulnerability, so this option allows extra security hardening
where iframes are not used in deployment

In much the same way as with the previous patch, the value is now manageable in
[puppet-horizon](https://github.com/openstack/puppet-horizon/commit/218c35ea7bc08dd88d936ab79b14e5ce2b94ea44)
with a secure default set within TripleO with `horizon::disallow_iframe_embed:
true`


### Disable Password Reveal

In much the same vain as with `enforce_password_check` and `disallow_iframe_embed`,
this
[patch](https://github.com/openstack/puppet-horizon/commit/ff13a2140fae8561e9caff999c80beced3091be5) allows the `disable_password_reveal` value to be toggled via
[TripleO](https://github.com/openstack/tripleo-heat-templates/commit/465d91380c8ab85128aee4e36f12425519b412e3)

### Password Validation

Horizon provides a password validation check, which OpenStack cloud
operators can use to enforce password complexity checks for users
within horizon.

A dictionary containing a regular expression can be used for
password validation with help text that is displayed if the password
does not pass validation.

This is harnessed using a regex field in Horizons `local_settings.py`:

{% highlight python %}
HORIZON_CONFIG["password_validator"] = {
   "regex": '.*', ]
      "help_text": _("Your password does not meet the requirements."),
         }
{% endhighlight %}

It is now possible to inject your own password validation regex via a heat
template enviroment file using the following patches in [puppet-horizon](https://github.com/openstack/puppet-horizon/commit/047e3b5c329427a45a33d6f575f295bea56636d3) and [tripleo](https://github.com/openstack/tripleo-heat-templates/commit/0e18ac5fdec4b9eeaef7f6aa83c466e86415e4e2)

An enviroment file can be passed to `openstack overcloud deploy --templates -e passsword_validation.yaml`

`password_validation.yaml:`


{% highlight yaml %}
parameter_defaults:
  HorizonPasswordValidator: '^.{8,18}$'
  HorizonPasswordValidatorHelp: 'Password must be between 8 and 18 characters.'
{% endhighlight %}


### Secure Proxy SSL Header Option

`SECURE_PROXY_SSL_HEADER` is used to tell Horizon (Django( to take into account the X-Forwarded-Proto header. It is disabled by default as it should only be enabled if one
is running horizon behind a proxy.

This can now be toggled in
[puppet-horizon](https://github.com/openstack/puppet-horizon/commit/5211ba5fc8652fbf6b3a7d72ed3de273d43d29ee)
and is set as `true` as the default in
[tripleo-heat-templates](https://github.com/openstack/tripleo-heat-templates/commit/db31ff5e5ac0cf05261eeb3a9b17764eb8e6dab6)

* Note that the `SECURE_PROXY_SSL_HEADER` patch was part of the [TLS Everywhere](https://jaormx.github.io/2016/testing-out-the-tls-everywhere-patches-for-tripleo/) work driven by Juan Antonio Osorio

## OS Based Changes

### /etc/issue Banner

An SSH `/etc/issue` Banner can be set using the following fields in an enviroment file
from patches in
[puppet-tripleo](https://github.com/openstack/puppet-tripleo/commit/5a1764acf7623ee04d8610793f418ab1d4e2226e) and [tripleo-heat-templates](https://github.com/openstack/tripleo-heat-templates/commit/73f58792f90942be1e2dc0ef67eac0a47d9aba18)

{% highlight yaml %}
{% raw %}
resource_registry:
  OS::TripleO::Services::Sshd: ../puppet/services/sshd.yaml

parameter_defaults:
  BannerText: |
    ******************************************************************
    * This system is for the use of authorized users only. Usage of  *
    * this system may be monitored and recorded by system personnel. *
    * Anyone using this system expressly consents to such monitoring *
    * and is advised that if such monitoring reveals possible        *
    * evidence of criminal activity, system personnel may provide    *
    * the evidence from such monitoring to law enforcement officials.*
    ******************************************************************
{% endraw %}
{% endhighlight %}

## Audit entries:

Having a system capable of recording all audit events is key for troubleshooting
and peforming analysis of events that led to a certain outcome. Red Hat
Enterprise Linux already has a complete audit system which is capable of logging
many events such as someone changing the system time, changes to Mandatory
/  Discretionary Access Control.

All audit rule entries can now be managed via [puppet-tripleo](https://github.com/openstack/puppet-tripleo/commit/eb14c2a9f7acd6a7949e7aee91687756731f93db) mapped to a yaml [tripleo-heat-template-template](https://github.com/openstack/tripleo-heat-templates/commit/afdc138987db8246be1f3a0948967f10c3011bb8) file.

Rules can be entered as follows, and injected into `/etc/audit/audit.rules`. The
`auditd` service will also be verified as runnning.

{% highlight yaml %}
parameter_defaults:
  AuditdRules:
    'watch for changes to opasswd file':
      content: '-w /etc/security/opasswd -p wa -k audit_rules_usergroup_modification'
      order  : 1
    'watch for changes to issue file':
      content: '-w /etc/issue -p wa -k audit_rules_networkconfig_modification'
      order  : 2
{% endhighlight %}

A full example of audit rules that can be set using Tripleo can be found
[here](https://raw.githubusercontent.com/openstack/tripleo-heat-templates/afdc138987db8246be1f3a0948967f10c3011bb8/environments/auditd.yaml)


# Pike work items

Pike will include changes sufficient enough to provide full DISA STIG
compliance.

It is then expected that full security Hardening will be available through the use of
a single environment file, for example:

`openstack overcloud deploy --templates -e disa-stig-compliance.yaml`

## CADF Managament

Configure openstack services to emit audit events using CADF

## Further Horizon security

Configurable values for `CSRF_COOKIE_SECURE` and further Horizon values.

## Kernel Parameter Hardening

Various sysctl values will be manageable via tripleo-heat-templates, including
values for Restricting Access to Kernel Message Buffer, Disable ICMP Redirects
and Disable KDump Kernel Crash Analyzer (kdump).

## pam.d management

Values to Limit Password Reuse, Set Lockout Time For Failed Password Attempts,
Set Password Minimum Length, and other attributes such as Minimum Digits,
Minimum number of consecutive characters, retry prompts per session.

## Install and Manage AIDE

AIDE (Advanced Intrusion Detection Environment) available to nodes, along with
the option to build an itegrity database and configuration of a periodic
execution of AIDE integrity verification.

## Partitioning

Currently overcloud images contain a single flat partition. Ongoing work in
Ironic and Disc Image Builder will make it possible implement to allow layouts
such as `/var/log/audit` `/tmp` as seperate volumes in heat template formatting.

## FIPS Kernel

A full FIPS Compliant Kernel as a deploy option.

## Restrict Dynamic Mounting and Unmounting of Filesystems

* Disable Modprobe Loading of USB Storage Driver
* Disable Kernel Support for USB via Bootloader Configuration
* Disable Booting from USB Devices in Boot Firmware

## Disable Core Dumps

Disable Core Dumps for SUID programs

## Enable ExecShield

Enable Randomized Layout of Virtual Address Space.

## Kernel Message Buffer

Restrict Access to Kernel Message Buffer

## SSH Changes

* Allow Only SSH Protocol 2
* Disable GSSAPI Authentication
* Disable Kerberos Authentication
* Enable Use of StictModes
* Enable Use of Privilege Separation
* Disable Compression Or Set Compression to delayed
* Set SSH Idle Timeout Interval
* Set SSH Client Alive Count
* Disable SSH Support for .rhosts Files
* Disable Host-Based Authentication
* Enable Encrypted X11 Fordwarding
* Disable SSH Root Login
* Disable SSH Access via Empty Passwords
* Do Not Allow SSH Environment Options
* Use Only Approved Ciphers
* Use Only FIPS Approved MACs

This is not a final list, and additions may be made as the work proceeds, but it
provides a decent overview of changes
