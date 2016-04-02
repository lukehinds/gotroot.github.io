This is an extension of a forum post that got out of hand. Someone was asking how the access control structure worked in keystone,with a focus on its handling of a true multi tenant cloud. I then practically ended up writing a small dissertation, so it made sense to move it to here and polish it up.

Multi tenancy in Keystone, is realised using 'Domains'. Domain are silos that pool projects, groups, users and roles.

To understand domains, a good reference point, is to think of a standard multi tenancy enterprise application, consisting of different companies, for example BMW, Ford. A domain will then contain a set of private objects, not exposed to a different domain, that shares the same openstack cloud.

Each domain will then have its own top level Domain Administrator, with that administrator having rights to perform the following actions:

* Create/Update/Delete Projects within their Domain

* Create/Update/Delete Users within their Domain

* Grant/Revoke Roles on Projects

Domains and the Domain administrator, need to be created by what is termed the 'The Cloud Administrator'. This is the main 'admin' set up during the initial install of keystone (think 'source openrc admin admin')

![Multi-Tenancy](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/domains-new-1024x641.png)

# SEPARATE IDENTITY BACK-ENDS

Now one might wonder, its very likely that domains are going to want seperate indenity backends. You can't expect BMW and Ford to share the same LDAP system. Conveniently the keystone developers thought of this, and so you can have dedicated identity driver configurations for each domain.

We enable these as follows:

~~~
[identity]
domain_specific_drivers_enabled = True
domain_config_dir = /etc/keystone/domains
~~~

Within the domain_config_dir, we then have our individual domain config files, for example:

~~~
keystone.bmw.conf
keystone.ford.conf
~~~

Domains without a specific configuration file will default to the options from the primary configuration file. It is best to keep the default configuration to SQL, so as to cater to the service accounts. We do so in /etc/keystone/keystone.conf as follows:

~~~
[identity]
driver = sql
~~~

Note, for versions earlier then Liberty, the full classpath is needed

~~~
driver = keystone.identity.backends.sql.Identity
~~~

We then set out specific domain driver, in keystone.bmw.conf

~~~
[identity]
driver = ldap

[ldap]
url=ldap://mock.bmw.com
user_tree_dn=cn=users,cn=accounts,dc=mock,dc=bmw,dc=com
user_id_attribute=uid
user_name_attribute=uid
group_tree_dn=cn=groups,cn=accounts,dc=mock,dc=bmw,dc=com
~~~

And then in keystone.ford.conf

~~~
[identity]
driver = ldap

[ldap]
url=ldap://mock.ford.com
user_tree_dn=cn=users,cn=accounts,dc=mock,dc=ford,dc=com
user_id_attribute=uid
user_name_attribute=uid
group_tree_dn=cn=groups,cn=accounts,dc=mock,dc=ford,dc=com
~~~

The domains are then added for both BMW & Ford:

~~~
$ openstack --os-identity-api-version=3 domain create bmw
$ openstack --os-identity-api-version=3 domain create ford
~~~

Note: instead of using directories, you can also use SQL

~~~
[identity]
domain_specific_drivers_enabled = true
domain_configurations_from_database = true
~~~

This will then allow domains to be configured or queried via the API. Plus point here being, no need to restart keystone.

sources:

http://docs.openstack.org/developer/keystone/configuration.html

http://beta.rdoproject.org/documentation/domains/
