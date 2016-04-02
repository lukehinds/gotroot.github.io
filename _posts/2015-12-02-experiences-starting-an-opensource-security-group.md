---
layout: post
title: Experiences starting an opensource security group.
subtitle: In the OPNFV
---


I would like to merit a post on the experience of starting a security group within an open source project, namely the [OPNFV](https://www.opnfv.org/) Security Group - what it is we do, why, and how we function as an opensource security unit.

# HOW IT STARTED

Shortly after OPNFV was launched as a Linux Foundation project in September 2014, a momentum of activity was quick to gather. A pent up demand for NFV upstream requirements, resulted in many project proposals being put forward by the group's initial members (since then OPNFV now has forty four approved projects!).

From having seen other open source projects leave security as a later consideration (and often suffering very badly for it), I had the idea of trying to get the OPNFV project 'into good habits early'. An email was sent to the tech-discuss mailing list, to gauge how much of an interest there maybe in starting a security group. The response back was very convincing, around 30 emails came back with support of the idea, with not a single dissenting voice (more a curiosity to know more).

From there, we went through the normal process of going forward to the technical steering committee. I fleshed out a project proposal, consisting of what we would do, and our overall mission statement, and with Juan Antonio Osorio Robles (Ozz) and Ari Pietikäinen on-board (the first two members to join me in the group), a pitch was made.

'A group dedicated to improve OPNFV security through architecture, documentation, code review, upstream inter-work with other groups, vulnerability management and security research.'   - opnfv security group mission statement.

The case was made to the tech steering committee, and the approval was granted.

# EBBS AND FLOWS

We started off with a lot of keen people showing up for the first few meetings, some being very gung-ho and others having a more tentative air. Over time the attendance fell each week (which is typical) until we were left with a core group of 4/6 people.

We kept momentum going, and even though it at times felt like progress was slow with not enough hours in the day, we now have a good foundation, and a future that I feel excited to be a part of. I guess we had experienced the typical cycle that an opensource project has to traverse - initial euphoria, followed by novelty diminished, and then an emphasis that you need to deliver. You pass each juncture and find yourself on a more solid footing.

Best of all new people join all the time (as others fade out) which injects more enthusiasm back into the project. Some of the new people, stick around, and become a core part of the group.

# WHAT WE DO TODAY- ONE YEAR ON.

So, a year has passed  now and the role of the group has grown over time (and I hope that we grow a lot more still).

The group is now a project that fully intends to stick around. We still continue to meet each week and discuss topics and what can be done.

We are in a good position to deliver what we envisioned as being the initial core goals of the group. Me being who I am, I always think we could have done more, and faster, but if I get real with myself, the fact we are still going strong and seeing some fruits of labour come into being, is in fact really good.

Here are the current projects we now have in place to help make our parent project, OPNFV a more secure gig.

# SECURITY GUIDE

We are busy co-authoring the OPNFV Security Guide, which will act as documentation for NFV infrastructure operators to understand how to secure the OPNFV platform and its ecosystem. This is a bottom up approach, from trusted boot, to trusted compute, integrity, security of the entire overlay network to the isolation of the VNF itself (there is a lot more besides that, but I don't want to put the entire table of contents in here).

As the opnfv project scales out over the ETSI-NFV architecture, so will the guide. Quite often the guide won't seek to re-invent the wheel, so perfectly good well maintained upstream security materials will be referenced (a good example being the excellent work of the OpenStack security guide).

We found the best approach here, was to use a good CI model to maintain the documentation, whereby content is submitted to gerrit for review, and then a jenkins job builds the sphinx developed documentation into html and PDF format. This way the Security Guide will be a continuous living document.

# INSPECTOR

Inspector is a project to improve the auditability of any given component within the OPNFV upstream eco-system. Essentially, for a system to be considered highly secure (I resisted saying 'carrier grade' there),  it needs to provide an extensive trusted audit of who did exactly what, and at which recorded time, and is the integrity of that information assured. If systems are not fully accountable, then you can never have a true picture of what happened, or often, what has already occurred within your infrastructure.

# SECURE CODE REVIEW

'The more eyes, the better' is a common mantra in open source. This means that the more people reading code, the greater the chance that someone will discover a flaw which could result in some form of security attack. This was the driver for putting into place the secure code review system. Anyone committing any code or change proposal, can tag the security group to offer feedback, via gerrit review.

I presented an overview of this recently, at the OPNFV summit, where we look for the following sorts of examples...

## OS COMMAND (SHELL) EXECUTION.

A very common functional case, in cloud infra stacks, is shell execution. For example a lot of OpenStack projects make calls and monitor CLI type workloads on the Host OS (Linux), and they do so by using common linux commands that are executed via code (most often python in the OpenStack world). This creates an environment, where a lot of power is at hand, to execute CLI commands on compute, network or controller nodes.

The problem with this approach, typically known as shell execution, is that it can allow a hacker to run their own often 'end game' commands on a system, and here we really are talking the worst of the worst, so everything from fork bombs to good old 'rm -rf /'

Here is a good example in Java of someone opening themselves wide open to this sort of attack.

```
String script = System.getProperty("SCRIPTNAME");
if (script != null)
System.exec(script);
The issue here, if not clear already, is that if an attacker has control over declaring the string variable 'script', then they could use that to invoke any manner of horribleness imaginable.
```

Here is another example, this time in python, with the infamous 'Shell = True'.

```
>>> from subprocess import call
>>> filename = input("What file would you like to display?\n")
What file would you like to display?
non_existent; rm -rf / --no-preserve-root #
>>> call("cat " + filename, shell=True)
```

Shell = True, results in filename, (being a CLI statement and not a filename) to be freely executed, in this case a recursive delete of your root filesystem.

Shell = True, can be used securely, but it requires caution. Avoid lots of pipes '|' or be careful of insecure variable injection.

The list goes on from here, but other usual suspects checked for, our race conditions, SQL injections, Improper Initialization, Cleartext transmission etc. More can be found on the wiki page.

# VULNERABILITY MANAGEMENT PROCESS

We have a full vulnerability handling process in place, which we are due to imminently launch. Some of structure kindly came from the OpenStack security group, who allowed us to use the process, by means of the creative commons license they assigned. The key differences with how ours operates, is that we interface with other projects, so if the code in question is upstream, we use the upstream vulnerability management process.

The process itself, uses a Responsible Disclosure Approach to handling vulnerabilities in the system. This then allows for the code to be fixed, patches created, and all before the issue is made public. By doing this, downstream stakeholders have time to prepare maintenance windows or customized patches deployment scripts.

# LINUX FOUNDATION CORE INFRASTRUCTURE SECURITY BADGE PROGRAM

A new one to the table... we just agreed to look at being an early adopter of the new LF Core infrastructure Security Badge Program. By working through the CI criteria we show that we, as an opensource project, are committed to security. This means that we adhere to a set of criteria which deems a project to have secure practises in its daily operative mode. I am going to a full write up on this later, once we complete, but we will consider including dynamic and static code analysis as part of our CI process among many others.

Other projects note of a mention, include efforts to align with the ETSI NFV Security standards. Internal secure development procedures.

# INTERESTED IN CONTRIBUTION

The group is very much open to others joining in, and we are a very friendly bunch of folks (if you don't mind me saying so myself). If you would like to see what we are about and are curious to know more, we meet weekly on irc.freenode.net @ 14:00 UTC in channel #opnfv-sec.

If you're part of an open source project, and would like to ask any questions, you're of course welcome too.

https://wiki.opnfv.org/security

[summit](![Multi-Tenancy](https://raw.githubusercontent.com/lukehinds/lukehinds.github.io/master/img/secgroup.jpg))

*Ray Paik Linux Foundation (left), Myself (middle) and Mark Beierl EMC( right)  - OPNFV Summit 11/2015*
