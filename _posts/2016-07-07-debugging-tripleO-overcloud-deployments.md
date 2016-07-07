---
layout: post
title: Debugging TripleO Overcloud Deployment Failures
subtitle: How to find the needle in a haystack
bigimg: /img/path.jpg
---



~~~
Stack overcloud CREATE_FAILED
Heat Stack create failed.
~~~

So your Overcloud Deployment has failed, what now? Well these are some notes I have gathered on fixing typical errors

## Error One 'Deployment exited with non-zero status code: X'

Not the most helpful of errors?

~~~
CREATE_FAILED  resources.ControllerOvercloudServicesDeployment_Step5: Error: resources[0]: Deployment to server failed: deploy_status_code : Deployment exited with non-zer
o status code: 6
~~~

I find the typically means something has gone wrong application side, the last time this happened with me, was due to an unsufficent memory allocation needed for HA Proxy.

~~~
heat stack-list --show-nested -f "status=FAILED"
~~~

Now lets use the 'id' field of a failed node, to look at what could have gone wrong:

~~~
heat deployment-show [id]
~~~

So for example:

~~~
heat deployment-show a8e4d9b5-49b3-44c2-bc16-a0f2b0944873
~~~

~~~
"deploy_stderr": "Could not retrieve fact='apache_version', resolution='<anonymous>': undefined method `[]' for nil:NilClass\nCould not retrieve fact='apache_version', resolution='<anonymous>': undefined method `[]' for nil:NilClass\n\u001b[1;31mWarning: Scope(Class[Mongodb::Server]): Replset specified, but no replset_members or replset_config provided.\u001b[0m\n\u001b[1;31mWarning: Scope(Class[Keystone]): Execution of db_sync does not depend on $enabled anymore. Please use sync_db instead.\u001b[0m\n\u001b[1;31mWarning: Scope(Class[Keystone]):
~~~

<snip>

~~~
Could not evaluate: Cannot allocate memory - fork(2)\u001b[0m\n\u001b[1;31mWarning: /Stage[main]/Heat::Deps/Anchor[heat::service::end]: Skipping because of failed dependencies\u001b[0m\n",
~~~

So there we have it, I used a flavor with insufficent memory. This meant I needed to reconfigure the memory allocation.


