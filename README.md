# Anarcho-Tech NYC: Prosody [![Build Status](https://travis-ci.org/AnarchoTechNYC/ansible-role-prosody.svg?branch=master)](https://travis-ci.org/AnarchoTechNYC/ansible-role-prosody)

An [Ansible role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) for installing and configuring a [Prosody](https://prosody.im/) server. Notably, this role has been tested with [Raspbian](https://www.raspbian.org/) on [Raspberry Pi](https://www.raspberrypi.org/) hardware. This role's purpose is to make it simple to install and configure an [XMPP](https://xmpp.org/) server in a variety of small- to medium-sized deployments.

## Prosody server configuration

Of this role's [default variables](defaults/main.yml), which you can override using any of [Ansible's variable precedence rules](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable), one of the most important is the `prosody_config` dictionary. It describes the state of the Prosody server configuration file. The dictionary's keys map almost one-to-one to the [variables in the Prosody server configuration file](https://prosody.im/doc/configure).

It may be helpful to see a few examples.

### Enabling and disabling Prosody VirtualHost configurations

> :construction: TK-TODO

## Configurating Prosody user accounts

> :construction: TK-TODO: The `prosody_users` list….
