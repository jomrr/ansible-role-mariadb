# Ansible Role: mariadb

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-mariadb)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-mariadb)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-mariadb)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-mariadb/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-mariadb/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-mariadb/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-mariadb/actions/workflows/main.yml?query=branch%3Amain)

Install, secure and manage MariaDB databases and accounts.

## Purpose

Install and secure a MariaDB instance using distribution packages, then manage
declared application databases and accounts.

## Scope

### Managed

- MariaDB server, client and PyMySQL packages; enabled and running service.
- A validated server snippet with bind address and TCP port.
- Local root socket authentication, anonymous and remote root account removal,
  test database and test grant removal.
- Application database and account present/absent states, passwords and
  privileges.

### Not Managed

- Replication, clustering, backups, TLS certificate deployment and firewall
  policy.
- Data directory migration, distribution configuration replacement or
  password-based root administration.
- Undeclared application databases and accounts.

## Requirements

- Ansible Core 2.20 or newer and the collections listed in collections.yml.
- MariaDB 10.4 or newer from the distribution repositories.
- A fresh installation or an existing instance allowing operating-system root to
  connect as database root over the default Unix socket. Existing password-only
  root accounts must be migrated before applying the role.
- Privilege escalation to operating-system root and package repository access.

## Dependencies

```yaml
collections:
  - name: ansible.posix
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.mariadb
    version: '>=6.0.0,<7.0.0'
```

## Role Variables

### `mariadb_service`

Type: `str`. Required: `false`.

Distribution service enabled and started by the role.

Default:

```yaml
mariadb_service: mariadb
```

### `mariadb_bind_address`

Type: `str`. Required: `false`.

Address on which MariaDB accepts TCP connections.

Default:

```yaml
mariadb_bind_address: 127.0.0.1
```

### `mariadb_port`

Type: `int`. Required: `false`.

TCP port for MariaDB clients.

Default:

```yaml
mariadb_port: 3306
```

### `mariadb_database_encoding`

Type: `str`. Required: `false`.

Character set for new databases without an item override.

Default:

```yaml
mariadb_database_encoding: utf8mb4
```

### `mariadb_database_collation`

Type: `str`. Required: `false`.

Collation for new databases without an item override.

Default:

```yaml
mariadb_database_collation: utf8mb4_unicode_ci
```

### `mariadb_databases`

Type: `list`. Required: `false`.

Application databases to create or remove; system databases and test are
reserved.

Default:

```yaml
mariadb_databases: []
```

### `mariadb_user_host`

Type: `str`. Required: `false`.

Default host component of application accounts.

Default:

```yaml
mariadb_user_host: localhost
```

### `mariadb_user_update_password`

Type: `str`. Required: `false`.

Password update policy for accounts without an item override.

Default:

```yaml
mariadb_user_update_password: always
```

### `mariadb_user_append_privs`

Type: `bool`. Required: `false`.

Extend existing grants instead of replacing them when true.

Default:

```yaml
mariadb_user_append_privs: false
```

### `mariadb_users`

Type: `list`. Required: `false`.

Application accounts to manage; present accounts require nonempty passwords.
Root, mysql, mariadb.sys and anonymous accounts are reserved.

Default:

```yaml
mariadb_users: []
```

### `mariadb_no_log`

Type: `bool`. Required: `false`.

Suppress output containing application account passwords.

Default:

```yaml
mariadb_no_log: true
```

## Managed Files

- `Debian and Ubuntu: /etc/mysql/mariadb.conf.d/99-ansible.cnf.`
- `AlmaLinux, Fedora and openSUSE: /etc/my.cnf.d/99-ansible.cnf.`

## Check Mode

Check mode is useful on an already installed, reachable instance.

- First-install check mode cannot connect to a server whose packages and service
  have only been simulated.
- The query module skips root authentication cleanup and test grant removal in
  check mode; these changes are not predicted. Account discovery is read-only
  and runs in check mode.

## Service Behavior

The service is always installed, enabled and started. Configuration changes
restart it before database objects are managed.

### Handlers

- Restart MariaDB after a changed server snippet.
- Flush privilege tables only after direct changes to root authentication
  metadata or test grants.

## Security Notes

- Root authenticates exclusively through the local Unix socket; no root password
  is stored or required.
- All root accounts outside localhost and all anonymous accounts are removed.
  The test database and grants for test and test wildcard databases are removed.
- Application accounts require nonempty passwords and cannot use reserved
  administrative names. Protect passwords with Ansible Vault; mariadb_no_log
  defaults to true.
- TCP listens on 127.0.0.1 by default. Opening access on other addresses
  requires appropriate network and TLS controls.
- Explicit state: absent removes the named database or account; database removal
  deletes its data.

## Operational Notes

- Distribution defaults retain control of data, log, PID and socket paths.
  Sockets are /run/mysqld/mysqld.sock on Debian/Ubuntu,
  /var/lib/mysql/mysql.sock on AlmaLinux/Fedora, and /run/mysql/mysql.sock on
  openSUSE.
- The role does not replace the main distribution configuration file. Native
  mariadbd --defaults-file validation checks the candidate snippet before
  deployment; it does not validate interactions with other administrator-managed
  snippets.
- Database encoding and collation are creation defaults; existing databases are
  not converted.
- User host, update_password and append_privs override their role-wide defaults
  per item. Privileges use the mariadb_user module string or dictionary syntax.
  Omitted privileges preserve existing grants.
- The former inert mariadb_role_enabled scaffold is removed: applying the role
  now establishes the running service.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Local MariaDB instance

Install and secure MariaDB with distribution paths.

```yaml
---
- name: Configure MariaDB
  hosts: mariadb_hosts
  gather_facts: true
  roles:
    - role: jomrr.mariadb
```

### Application database and account

Create a database and a local account with privileges restricted to that database.

```yaml
---
mariadb_databases:
  - name: application
mariadb_users:
  - name: application
    password: "{{ vault_mariadb_application_password }}"
    priv: 'application.*:ALL'
```

### Remove an obsolete account and database

Explicit removal deletes the named database and its data.

```yaml
---
mariadb_users:
  - name: obsolete
    state: absent
mariadb_databases:
  - name: obsolete
    state: absent
```

## References

- [Ansible MariaDB modules](https://docs.ansible.com/projects/ansible/latest/collections/ansible/mariadb/)
- [MariaDB secure installation](https://mariadb.com/docs/server/clients-and-utilities/deployment-tools/mariadb-secure-installation)
- [MariaDB authentication](https://mariadb.com/docs/server/security/user-account-management/authentication-from-mariadb-10-4)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2020 Jonas Mauer.
