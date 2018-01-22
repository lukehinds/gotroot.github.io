---
layout: post
title: TripleO AIDE Service
subtitle: Intrusion Detection in TripleO
tags:
  nfvpe
---

Just a quick post about some [patches](https://review.openstack.org/#/q/topic:bp/tripleo-aide-database+(status:open+OR+status:merged)) that have landed in TripleO (upstream Red Hat
OSP Director) to deploy AIDE to the overcloud.

## What's AIDE?

AIDE (Advanced Intrusion Detection Environment) is a file and directory integrity checker. It is used as medium to reveal possible unauthorized file tampering / changes.

AIDE creates an integrity database of file hashes, which can then be used as a comparison point to verify the integrity of the files and directories.

## How do we use this?

The TripleO AIDE service allows an operator to populate entries into an AIDE configuration, which is then used by the AIDE service to create an integrity database.

This can be achieved using an environment file with the following YAML structure:

{% highlight yaml %}
resource_registry:
OS::TripleO::Services::Aide: ../puppet/services/aide.yaml

parameter_defaults:
AideRules:
'TripleORules':
  content: 'TripleORules = p+sha256'
  order  : 1
'etc':
  content: '/etc/ TripleORules'
  order  : 2
'boot':
  content: '/boot/ TripleORules'
  order  : 3
'sbin':
  content: '/sbin/ TripleORules'
  order  : 4
'var':
  content: '/var/ TripleORules'
  order  : 5
'not var/log':
  content: '!/var/log.*'
  order  : 6
'not var/spool':
  content: '!/var/spool.*'
  order  : 7
'not /var/adm/utmp'
  content: '!/var/adm/utmp$ '
  order: 8
'not nova instances'
  content: '!/var/lib/nova/instances.*'
  order: 9
{% endhighlight %}

If above environment file were saved as aide.yaml it could then be passed to the overcloud deploy command as follows:

`openstack overcloud deploy --templates -e /path/to/aide.yaml`

Let’s walk through the different values used here.

First an ‘alias’ name `TripleORules` is declared to save us repeatedly typing out the same attributes each time. To the alias we apply attributes of `p+sha256`. In AIDE terms this reads as monitor all file permissions `p` with an integrity checksum of `sha256`. For a complete list of attributes that can be used in AIDE’s config files, refer to the AIDE MAN page.

Complex rules can be created using this format, such as the following:

`MyAlias = p+i+n+u+g+s+b+m+c+sha512`

The above would translate as monitor permissions, inodes, number of links, user, group, size, block count, mtime, ctime, using sha256 for checksum generation.

Note, the alias should always have an order position of 1, which means that it is positioned at the top of the AIDE rules and is applied recursively to all values below.

Following after the alias are the directories to monitor. Note that regular expressions can be used. For example we set monitoring for the var directory, but overwrite with a not clause using ! with `!/var/log.*` and `!/var/spool.*`.

## Further AIDE values

The following AIDE values can also be set.

**AideConfPath**: The full POSIX path to the aide configuration file, this defaults to /etc/aide.conf. If no requirement is in place to change the file location, it is recommended to stick with the default path.

**AideDBPath**: The full POSIX path to the AIDE integrity database. This value is configurable to allow operators to declare their own full path, as often AIDE database files are stored off node perhaps on a read only file mount.

**AideDBTempPath**: The full POSIX path to the AIDE integrity temporary database. This temporary files is created when AIDE initializes a new database.

**AideHour**: This value is to set the hour attribute as part of AIDE cron configuration.

**AideMinute**: This value is to set the minute attribute as part of AIDE cron configuration.

**AideCronUser**: This value is to set the linux user as part of AIDE cron configuration.

**AideEmail**: This value sets the email address that receives AIDE reports each time a cron run is made.

**AideMuaPath**: This value sets the path to the Mail User Agent that is used to send AIDE reports to the email address set within AideEmail.

## Cron configuration

The AIDE TripleO service allows configuration of a cron job. By default it will send reports to `/var/log/audit/`, unless `AideEmail` is set, in which case it will instead email the reports to the declared email address.

## AIDE and Upgrades

When an upgrade is performed, the AIDE service will automatically regenerate a new integrity database to ensure all upgraded files are correctly recomputed to possess a updated checksum.

If `openstack overcloud deploy` is called as a subsequent run to an initial deployment and the AIDE configuration rules are changed, the TripleO AIDE service will rebuild the database to ensure the new config attributes are encapsulated in the integrity database.
