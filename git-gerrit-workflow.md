---
layout: page
title: OpenStack Patch flow
subtitle: GIT Review & Gerrit
---

# Check config

~~~
git config --list
~~~

~~~
git config --global gitreview.username yourgerritusername
~~~

## New Project

### Clone

~~~
git clone https://git.openstack.org/openstack/<projectname>.git
~~~

### Set up git review
~~~
cd <projectname>
git review -s
~~~

# Work flow

There are 4 key tasks with regards to bugs that anyone can do:

1. Confirm new bugs: When a bug is filed, it is set to the “New” status. A “New” bug can be marked “Confirmed” once it has been reproduced and is thus confirmed as genuine.
2. Solve inconsistencies: Make sure bugs are Confirmed, and if assigned that they are marked “In Progress”
3. Review incomplete bugs: See if information that caused them to be marked “Incomplete” has been provided, determine if more information is required and provide reminders to the bug reporter if they haven’t responded after 2-4 weeks.
4. Review stale In Progress bugs: Work with assignee of bugs to determine if the bug is still being worked on, if not, unassign them and mark them back to Confirmed or Triaged.


If you find a bug that you wish to work on, you may assign it to yourself. When you upload a review, include the bug in the commit message for automatic updates back to Launchpad. The following options are available:

Closes-Bug: #######
Partial-Bug: #######
Related-Bug: #######


Once your local repository is set up as above, you must use the following workflow.

Make sure you have the latest upstream changes:

~~~
git remote update
git checkout master
git pull --ff-only origin master
~~~

Create topic branch:

~~~
git checkout -b TOPIC-BRANCH
~~~

For example:

~~~
git checkout -b bug/123456
~~~

Hack your changes, and then add them:

~~~
git add <file>
~~~

### Commit

Implements: blueprint BLUEPRINT
Closes-Bug: ####### (Partial-Bug or Related-Bug are options)

For example:

~~~
Adds keystone support

...Long multiline description of the change...

Implements: blueprint authentication
Closes-Bug: #123456
Change-Id: I4946a16d27f712ae2adf8441ce78e6c0bb0bb657
~~~

Make your changes, commit them, and submit them for review:

~~~
git commit -a
~~~

~~~
git review
~~~

Making a patch change:

~~~
git commit -a --amend
git review
~~~


### Squash changes

~~~
git checkout master
git pull origin master
git checkout TOPIC-BRANCH
git rebase -i master
~~~

### Pull down review

Use Patchset

For example, for https://review.openstack.org/#/c/148486/

~~~
git review -d 148486
~~~

Or change id:

~~~
git review -d I35729a86e211391f67cc959d19416c9125c6f9eb
~~~

You can also request a specific revision of the patch by appending a comma and the patch number. 
E.g, to get the second revision of that patch:

~~~
git review -d 148486,2
~~~
