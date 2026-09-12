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
- A validated server snippet with TCP listener control, bind address and port,
  including Unix-socket-only operation.
- Optional TLS configuration using existing certificates and an optional
  encrypted TCP transport requirement.
- Local root socket authentication, anonymous and remote root account removal,
  test database and test grant removal.
- Application database and account present/absent states, passwords and
  privileges.
- Startup plugin loading and native options, including Audit, GSSAPI and PAM,
  with distribution package defaults.
- Authentication plugin selection and native authentication strings for managed
  application accounts.

### Not Managed

- Replication, clustering, backups, TLS certificate deployment and firewall
  policy.
- Data directory migration, distribution configuration replacement or
  password-based root administration.
- Undeclared application databases and accounts.
- Kerberos realms, KDCs, keytab provisioning, PAM service policy and audit log
  directories or retention outside plugin settings.

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
  - name: community.crypto
    version: '>=3.0.0'
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

### `mariadb_tcp_enabled`

Type: `bool`. Required: `false`.

Enable TCP connections in addition to the distribution Unix socket. False
disables all TCP listeners using skip-networking.

Default:

```yaml
mariadb_tcp_enabled: true
```

### `mariadb_bind_address`

Type: `str`. Required: `false`.

Address on which MariaDB accepts TCP connections; only used when
mariadb_tcp_enabled is true.

Default:

```yaml
mariadb_bind_address: 127.0.0.1
```

### `mariadb_port`

Type: `int`. Required: `false`.

TCP port for MariaDB clients; only used when mariadb_tcp_enabled is true.

Default:

```yaml
mariadb_port: 3306
```

### `mariadb_tls_enabled`

Type: `bool`. Required: `false`.

Configure TLS with existing certificate files. False leaves distribution TLS
settings in effect.

Default:

```yaml
mariadb_tls_enabled: false
```

### `mariadb_tls_cert_file`

Type: `path`. Required: `false`.

Absolute path to the existing PEM server certificate and intermediate chain;
required when TLS is enabled.

### `mariadb_tls_key_file`

Type: `path`. Required: `false`.

Absolute path to the existing unencrypted PEM private key; required when TLS is
enabled and readable by MariaDB.

### `mariadb_tls_ca_file`

Type: `path`. Required: `false`.

Optional absolute path to an existing PEM CA bundle. Empty leaves the CA setting
unconfigured.

Default:

```yaml
mariadb_tls_ca_file: ''
```

### `mariadb_require_secure_transport`

Type: `bool`. Required: `false`.

Require TLS for TCP connections while allowing local Unix sockets. Requires
mariadb_tls_enabled and MariaDB 10.5.2 or newer. False leaves the distribution
transport policy in effect.

Default:

```yaml
mariadb_require_secure_transport: false
```

### `mariadb_database_encoding`

Type: `str`. Required: `false`.

Character set for new databases without an item override.

Default:

```yaml
mariadb_database_encoding: utf8mb4
```

### `mariadb_plugins`

Type: `list`. Required: `false`.

Server plugins loaded from distribution plugin directories on startup. Enabled
plugins must initialize successfully.
Audit, GSSAPI and PAM use distribution package defaults; other plugins may
supply their packages explicitly.

Default:

```yaml
mariadb_plugins: []
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

### `mariadb_user_plugin`

Type: `str`. Required: `false`.

Authentication plugin for accounts without an item override. The default
requires a nonempty database password.

Default:

```yaml
mariadb_user_plugin: mysql_native_password
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

Application accounts to manage; mysql_native_password accounts require nonempty
database passwords.
Other authentication plugins use their native authentication strings. Root,
mysql, mariadb.sys and anonymous accounts are reserved.

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
- mysql_native_password application accounts require nonempty passwords. Other
  authentication plugins must omit password and use their native authentication
  strings as appropriate. Reserved administrative account names remain
  prohibited. Protect authentication secrets with Ansible Vault; mariadb_no_log
  defaults to true.
- TCP listens on 127.0.0.1 by default. Opening access on other addresses
  requires appropriate network and TLS controls. Set mariadb_tcp_enabled to
  false to disable TCP completely, including loopback connections.
- Enable mariadb_tls_enabled with mariadb_tls_cert_file and mariadb_tls_key_file
  to use existing PEM files. An optional mariadb_tls_ca_file supplies a CA
  bundle. Clients must verify the server certificate and hostname.
- Set mariadb_require_secure_transport to true to reject unencrypted TCP
  connections; Unix socket administration remains available. This setting
  requires MariaDB 10.5.2 or newer and mariadb_tls_enabled.
- PAM with pam_use_cleartext_plugin sends the PAM password to the server; use
  certificate-verified TLS and enforce secure transport. PAM policy and helper
  permissions must already permit the MariaDB service to authenticate accounts.
- Audit event selection and output are native server_audit options. Enable
  server_audit_logging explicitly; audit logs can contain application query data
  and should have appropriately restricted storage and retention.
- Explicit state: absent removes the named database or account; database removal
  deletes its data.

## Operational Notes

- Distribution defaults retain control of data, log, PID and socket paths.
  Sockets are /run/mysqld/mysqld.sock on Debian/Ubuntu,
  /var/lib/mysql/mysql.sock on AlmaLinux/Fedora, and /run/mysql/mysql.sock on
  openSUSE. Platform variables record the native data directory, including
  /var/lib/mariadb on Ubuntu 26 and /var/lib/mysql on the other tested
  platforms. Verification uses these values without relocating existing data.
- The role does not replace the main distribution configuration file. Native
  mariadbd --defaults-file validation checks the candidate snippet before
  deployment; it does not validate interactions with other administrator-managed
  snippets.
- mariadb_tcp_enabled controls skip-networking explicitly. When false, the role
  omits bind-address and port from its snippet; all clients must use the
  distribution Unix socket. Database and user management remain available
  through that socket. Set it back to true to restore TCP with
  mariadb_bind_address and mariadb_port; changes restart MariaDB.
- Database encoding and collation are creation defaults; existing databases are
  not converted.
- TLS paths must be absolute and readable by the MariaDB service, including
  directory permissions and any SELinux or AppArmor policy. Provision
  certificates and an unencrypted private key before applying the role; restrict
  key access to root and the MariaDB service account.
- Disabling mariadb_tls_enabled removes the role's TLS options and restores
  distribution settings; newer MariaDB versions may still offer automatic TLS.
  Disabling mariadb_require_secure_transport likewise restores the distribution
  policy.
- Certificate replacement at an unchanged path is managed externally; restart
  MariaDB or issue FLUSH SSL after renewal. Changes to configured paths notify
  the role's restart handler.
- The community.crypto collection is used only to generate disposable Molecule
  test certificates.
- User host, update_password and append_privs override their role-wide defaults
  per item. Privileges use the mariadb_user module string or dictionary syntax.
  Omitted privileges preserve existing grants.
- mariadb_plugins entries require the plugin name and library basename. The role
  uses plugin-load-add and FORCE to make enabled plugin initialization
  mandatory. Options take native MariaDB names and scalar values; enabled
  defaults to true.
- Set enabled to false to write OFF and keep a plugin disabled across restarts,
  including plugins loaded by distribution snippets or mysql.plugin. Removing an
  entry only removes this role's configuration; external loading rules remain in
  effect. Plugin packages are retained, and accounts referencing disabled
  plugins are not migrated automatically.
- Audit and PAM libraries are included in the base server package on
  Debian/Ubuntu and openSUSE; Red Hat distributions use mariadb-pam for PAM.
  GSSAPI uses mariadb-plugin-gssapi-server on Debian/Ubuntu and
  mariadb-gssapi-server on Red Hat; openSUSE includes it in the server package.
  An item's packages list overrides these defaults; unknown plugins default to
  [].
- GSSAPI requires an existing keytab readable by the MariaDB service and a
  matching gssapi_principal_name. Client accounts use plugin_hash_string for
  their Kerberos principal, including the realm. Realm configuration and client
  tickets remain external.
- PAM accounts use plugin_auth_string for the service name in /etc/pam.d; the
  role does not create or replace that service. The openSUSE package only
  accepts the service name mysql; provision /etc/pam.d/mysql there. Set
  mariadb_user_plugin for a role-wide authentication policy or plugin on
  individual accounts.
- The upstream user module gives password precedence over plugin selection. The
  role rejects password with other plugins and accepts at most one of password,
  plugin_auth_string and plugin_hash_string to avoid unintended authentication
  methods.
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

### Unix socket only

Accept local Unix socket connections without opening any TCP listener.

```yaml
---
mariadb_tcp_enabled: false
```

### Require encrypted TCP connections

Use certificates provisioned by an external PKI. Requires MariaDB 10.5.2 or newer.

```yaml
---
mariadb_tls_enabled: true
mariadb_tls_cert_file: /etc/mariadb/tls/server.crt
mariadb_tls_key_file: /etc/mariadb/tls/server.key
mariadb_tls_ca_file: /etc/mariadb/tls/ca.crt
mariadb_require_secure_transport: true
```

### Audit database activity

Send selected audit events to syslog using the distribution's logging service.

```yaml
---
mariadb_plugins:
  - name: server_audit
    library: server_audit
    options:
      server_audit_logging: 'ON'
      server_audit_output_type: SYSLOG
      server_audit_events: CONNECT,QUERY_DDL,QUERY_DML
```

### GSSAPI and PAM application authentication

Use an existing Kerberos keytab and PAM service with required TLS.

```yaml
---
mariadb_tls_enabled: true
mariadb_tls_cert_file: /etc/mariadb/tls/server.crt
mariadb_tls_key_file: /etc/mariadb/tls/server.key
mariadb_require_secure_transport: true
mariadb_plugins:
  - name: gssapi
    library: auth_gssapi
    options:
      gssapi_keytab_path: /etc/mysql/mariadb.keytab
      gssapi_principal_name: mariadb/db.example.com@EXAMPLE.COM
  - name: pam
    library: auth_pam
    options:
      pam_use_cleartext_plugin: 'ON'
mariadb_users:
  - name: kerberos_application
    plugin: gssapi
    plugin_hash_string: application@EXAMPLE.COM
    priv: 'application.*:SELECT'
  - name: pam_application
    plugin: pam
    plugin_auth_string: mysql
    priv: 'application.*:SELECT'
```

### Disable a plugin

Keep PAM disabled even when another configuration source loads its library.

```yaml
---
mariadb_plugins:
  - name: pam
    library: auth_pam
    enabled: false
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

- [MariaDB Audit Plugin](https://mariadb.com/docs/server/reference/plugins/mariadb-audit-plugin)
- [MariaDB GSSAPI authentication](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-plugin-gssapi)
- [MariaDB PAM authentication](https://mariadb.com/docs/server/reference/plugins/authentication-plugins/authentication-with-pluggable-authentication-modules-pam/authentication-plugin-pam)
- [MariaDB TLS configuration](https://mariadb.com/docs/server/security/encryption/data-in-transit-encryption/securing-connections-for-client-and-server)
- [Ansible MariaDB modules](https://docs.ansible.com/projects/ansible/latest/collections/ansible/mariadb/)
- [MariaDB secure installation](https://mariadb.com/docs/server/clients-and-utilities/deployment-tools/mariadb-secure-installation)
- [MariaDB authentication](https://mariadb.com/docs/server/security/user-account-management/authentication-from-mariadb-10-4)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2020-2026 Jonas Mauer.
