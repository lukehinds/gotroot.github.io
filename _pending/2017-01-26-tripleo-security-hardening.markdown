---
layout: post
title: TripleO Security Hardening
subtitle: Simple Notes on Rust Lang
---

I just wrapped up working on a new project to provide TripleO instantiated
security hardening.

This allows operators to provide security hardening as a default for overcloud
nodes.

List of patches, and show of yaml

Create all-in-one yaml files `disa.yaml'


## /etc/issue Banner

An SSH `/etc/issue` Banner can be set using the following in an enviroment file

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

## Horizon Password Validation

resource_registry:
  OS::TripleO::Services::Horizon: ../puppet/services/horizon.yaml

parameter_defaults:
  PasswordValidator: '.*'
  PasswordValidatorHelp: 'Your password does not meet the requirements.'

# Auditd entries:

A full range of auditd entries are available in a yaml template file.

parameter_defaults:
  AuditdRules:
    'watch for changes to opasswd file':
      content: '-w /etc/security/opasswd -p wa -k audit_rules_usergroup_modification'
      order  : 1
    'watch for changes to issue file':
      content: '-w /etc/issue -p wa -k audit_rules_networkconfig_modification'
      order  : 2

# Defaults

## CSRF_COOKIE_SECURE parameter set to True

## DISABLE_PASSWORD_REVEAL parameter set to True


