# Anarcho-Tech NYC: Prosody [![Build Status](https://travis-ci.org/AnarchoTechNYC/ansible-role-prosody.svg?branch=master)](https://travis-ci.org/AnarchoTechNYC/ansible-role-prosody)

An [Ansible role](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) for installing and configuring a [Prosody](https://prosody.im/) server. Notably, this role has been tested with [Raspbian](https://www.raspbian.org/) on [Raspberry Pi](https://www.raspberrypi.org/) hardware. This role's purpose is to make it simple to install and configure an [XMPP](https://xmpp.org/) server in a variety of small- to medium-sized deployments.

## Prosody server configuration

To configure a Prosody server, several default variables are defined:

* `prosody_server_username`: The OS-level user account name under which Prosody will run. This defaults to `prosody`. Don't change this if you are installing Prosody from the project's `deb` file, as this is the name of the account that is packaged with the Prosody distribution.
* `prosody_server_data_dir`: Path to the directory in which Prosody will store user information. This defaults to `/var/lib/{{ prosody_server_username }}`, which is also the `prosody` user's default `$HOME` directory.
* `prosody_server_run_dir`: Path to the server's runtime directory, which stores ephemeral files used while the server is running. This defaults to `/var/run/{{ prosody_server_username }}`.

The bulk of Prosody's configuration is handled by a dictionary variable called `prosody_config`. It describes the state of the Prosody server configuration file. The dictionary's keys map almost one-to-one to the [variables in the Prosody server configuration file](https://prosody.im/doc/configure). An exception is the `VirtualHosts` key, which is a list of dictionaries each [describing a Prosody `VirtualHost` to configure](#configuring-prosody-virtualhosts).

It may be helpful to see a few examples.

1. Default Prosody installation:
    ```yml
    prosody_config:
      admins: []
      modules_enabled:
        - roster
        - saslauth
        - tls
        - dialback
        - disco
        - carbons
        - pep
        - private
        - blocklist
        - vcard
        - version
        - uptime
        - time
        - ping
        - register
        - admin_adhoc
      modules_disabled: []
      allow_registration: false
      c2s_require_encryption: true
      s2s_require_encryption: true
      s2s_secure_auth: false
      pidfile: "{{ prosody_server_run_dir }}/prosody.pid"
      authentication: internal_hashed
      archive_expires_after: "1w"
      certificates: certs
      VirtualHosts:
        - domain: localhost
    ```
    This is equivalent to the Prosody 0.10 configuration file that ships with the project. If the above settings are what you desire, you need not include a `prosody_config` dictionary at all.
1. Prosody server responsible for two domains (`localhost` and `open-registration.local`), the latter of which allows in-band XMPP account registration while the former does not:
    ```yml
    prosody_config:
      admins:
        - admin@localhost
      allow_registration: false
      VirtualHosts:
        - domain: localhost
        - domain: open-registration.local
          allow_registration: true
    ```
1. Prosody server with [SQL-backed data store](https://prosody.im/doc/modules/mod_storage_sql) on PostgreSQL:
    ```yml
    prosody_config:
      storage: "sql"
      sql:
        driver: "PostgreSQL"
        database: "prosody_db"
        host: "localhost"
        port: 3306
        username: prosody
        password: your_password
    ```
1. Prosody server with [certain user data pieces split across multiple storage backends](https://prosody.im/doc/storage):
    ```yml
    prosody_config:
      default_storage: internal
      storage:
        accounts: sql
        roster: sql
      sql:
        driver: "MySQL"
        database: "prosody_db"
        host: "localhost"
        port: 3306
        username: prosody
        password: your_password
    ```
    The above will store user accounts and passwords, as well as user contact lists (rosters) in the MySQL database on the localhost, but other data, such as user's own profile information (vCards), will use the default `internal` (filesystem-based) storage backend.

### Configuring Prosody VirtualHosts

A single Prosody XMPP server can serve multiple domains. That is to say, if your server is addressable on `example.com` but you want your users to have JabberIDs in the form of `user@otherdomain.com`, you can configure Prosody with `otherdomain.com` as a VirtualHost to claim responsibility for it, even though the server itself is available only at `example.com`. To do so, you will need to [configure Prosody VirtualHosts](https://prosody.im/doc/configure#adding_a_host).

In the `prosody_config` dictionary, a `VirtualHosts` list provides an interface to configuring any number of Prosody VirtualHosts. For the most part, an item in the `VirtualHosts` list is equivalent to a VirtualHost-specific configuration that overrides the same setting in the global configuration dictionary. This allows you to create a default (global) configuration that can have per-host overrides.

Each VirtualHost dictionary further accepts the following additional keys:

* `domain`: The fully-qualified domain name (FQDN) of the VirtualHost.
* `state`: Can be `present` (the default if undefined), in which case the VirtualHost-specific configuration will be written to disk, or `absent`, in which case it will be removed.
* `enabled`: Boolean that determines whether or not the VirtualHost will be active (loaded by Prosody and symlinked to [the `virtualhosts-enabled` directory](enabling-and-disabling-prosody-virtualhost-configurations)). Defaults to `true` if left undefined.

#### Enabling and disabling Prosody VirtualHost configurations

Individual VirtualHost configurations are stored in the `/etc/prosody/virtualhosts-available` directory when their `state` is set to `present`. For example, if you have a VirtualHost with a `domain` of `otherdomain.com`, the file `/etc/prosody/virtualhosts-available/otherdomain.com.cfg.lua` will be written. If this VirtualHost is also `enabled`, that file will be symlinked from `/etc/prosody/virtualhosts-enabled/otherdomain.com.cfg.lua`. The main prosody configuration file (`/etc/prosody/prosody.cfg.lua`) [`Include`'s](https://prosody.im/doc/packagers#including_separate_files) any file ending in `.cfg.lua` in the directory `/etc/prosody/virtualhosts-enabled` when the `prosody_config.VirtualHost` key is defined.

You can manually disable a Prosody VirtualHost by removing the symbolic link pointing to the VirtualHost configuration file and then reloading the server:

```sh
sudo -u prosody rm /etc/prosody/virtualhosts-enabled/otherdomain.cfg.lua
sudo prosodyctl reload
```

## Configuring Prosody user accounts

You can define a list of user accounts to ensure are present or absent from the Prosody server using the `prosody_users` list. The items in this list are dictionaries that describe the user account's properties. The keys of this dictionary are as follows:

* `jid`: The [bare JabberID](https://xmpp.org/rfcs/rfc3920.html#rfc.section.3.3) of the user account.
* `password`: The password for the user account. You should almost certainly [ensure this value is encrypted with Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html#encrypt-string-for-use-in-yaml).
* `state`: Can be `present` (the default), to ensure the user account exists, or `absent`, to ensure it does not.
