---
layout: post
title: Debugging Puppet Modules in the Overcloud
---

I have been working a lot recently on OpenStack TripleO via Puppet module
development. When wanting to test any code or changes, it is quite
a long drawn out process to go through cycles of making code changes, deploying
the entire overcloud, checking the results to then repeat the cycle again.

This quick tutorial will go through how to inject a puppet module into your
overcloud and then use `puppet apply` to test a specific puppet module of your
choosing.

I guess this will likely be obvious to already seasoned puppet developers, but
for someone like myself who has just crossed over from the python world, this
helped me save a lot of time during debugging.

For reference we will use a current patch I am working on that is yet to merge,
and its related tripleo-heat-templates. The module itself is situated in the
`puppet-horizon` repository, and provides a means for a cloud operator to inject
the horizon `password_validaton` fields into `local_settings`.

I am going to assume you already have an undercloud deployed, and base the
tutorial at that juncture (after image upload and introspection, and before the overcloud is deployed).

## Uploading Puppet Modules

First of all create a `puppet-modules` directory in `/home/stack` on your
undercloud.

`cd` into this directory and clone your wanted `puppet-<module>` repository, by removing the
`puppet-` prefix like so:

```
git clone git://git.openstack.org/openstack/puppet-horizon horizon
```

This will then leave you with `/home/stack/puppet-modules/horizon`

For this example, I am going to cherry pick a non merged patch into the new
`horizon` repository / directory.

```
git fetch git://git.openstack.org/openstack/puppet-horizon refs/changes/53/413653/11 && git checkout FETCH_HEAD
```

Note: The above patch may well already be merged. If that is the case, you  of
course do not need to cherry pick any patches.

Next we clone our `tripleo-heat-templates` repository, to allow us to have some
hiera data for our manifest parameters.

```
git clone git://git.openstack.org/openstack/puppet-horizon horizon
```

And get the patch we need:

```
git fetch git://git.openstack.org/openstack/tripleo-heat-templates refs/changes/44/413644/5 && git checkout FETCH_HEAD
```

We will now deploy our patches to swift, for inject into the overcloud.

For this we will use a tool called `upload-puppet-modules` which will already be present on
your undercloud node.

```
upload-puppet-modules -d ~/puppet-modules
```

This will now tarball our `puppet-modules/horizon` directory and upload it to
swift, where it will then be deployed to every overcloud node via heat and
os-config.

Next we deploy the overcloud itself, while sourcing the TripleO templates we
cloned earlier and an enviroment file to provide the hiera data needed for horizons
password_validaton fields.

```
openstack overcloud deploy --templates ~/tripleo-heat-templates -e
~/tripleo-heat-templates/enviroments/horizon_password_validation.yaml
```

## Running puppet apply locally

Let's now assume something is wrong in our code, and we need to make some small
tweaks to debug. We can now do so directly on the overcloud.

ssh from to an overcloud node which contains your puppet module (this maybe
contigent upon how you have set up your `roles_data.yaml`). Note that you will
likely ssh as `heat-admin`, and you may need to `sudo su` to root to run
`puppet-apply`, as most of the time you will be asking puppet to change files
owned by root.

Change to the `/etc/puppet/` directory where you will find the `modules`
directory (in here will be the `horizon` directory / repository that we uploaded
earlier to swift).

From here, we run `puppet apply`, but we pass in the name of the module class we
wish to run (if we omit this, puppet will run all manifests and you will have
a long wait again).

We will use the class name of 'horizon' which can be seen in the [init.pp](https://github.com/openstack/puppet-horizon/blob/master/manifests/init.pp#L387) file in the `puppet-horizon` puppet module.

```
puppet apply -e 'include horizon'
```

This will now run the manifest (should take about 15 seconds) and execute your puppet code.

If you're not happy with the result for whatever reason, then you can directly tailor the `/etc/puppet/modules/horizon/*` files, or the hiera data found in `/etc/puppet/hieradata/*`, and then just re-run `puppet apply -e 'include horizon'` again.
