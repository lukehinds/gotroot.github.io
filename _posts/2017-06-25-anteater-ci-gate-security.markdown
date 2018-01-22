---
layout: post
title: Anteater - CI Gate Security
tags:
  nfvpe
---

## Introduction

 This post is about my latest project that just went live in the Linux Foundation project, OPNFV.

 The project is named 'Anteater' and is centered on providing security checks on git commits before they are merged into any given branch.  However, before I dive into the guts of the application though, let me give some background on the problems it is designed to solve.

 ## CI Based attacks

 Continous Integration has been a much needed change in the development world, no denying that. Bugs are now caught often and early and the old issue of 'works for me' is almost resigned to the past, thanks to the close alignment of testing and production enviroments.

 However, as with anything, it brings its own new risks to the table...

 Using CI a developer can stage / commit code changes to a code (git) repository, and immediately gain test feedback from a series of unit or functional "quality gate" tests. These tests will either verfify that key programatic functions return to correct inputs and validation as per their design, or perform 'mocked' key use cases to insure the key functionaility of the application is healthy and no new of regressive bugs are introduced.

 If the gate tests are passed, then the code is allowed to move from its quarantned state to 'aprroved' where it is merged into the main code branch often called 'master', from there master is branched or tagged to make a release.  

 Code that merged into master and tagged for a release is then pushed to production by the same DevOps oriented automation scripts or applications (puppet, chef, ansible etc) deploy the application in the test environment.

 So this then begs the question, what is something malicious was allowed through gate? There is a very good chance it will be deployed into a production enviroment, which could allow someone a backdoor to then steal private data or worse.

 So who then would submit something malicious in patch? Well there are two types of individuals here, one is someone making a genuine mistake, another is some
