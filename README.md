# ansible-role-mariadb

**Ansible role for setting up [MariaDB](https://mariadb.org).**

## Supported Platforms

- Alpine 3.11
- Archlinux
- CentOS 8
- Debian 9, 10
- Fedora 31
- Ubuntu 18.04, 20.04

## Requirements

Ansible 2.9 or higher is recommended.

## Variables

Variables and defaults for this role:

| variable | default value in defaults/main.yml | description |
| -------- | ---------------------------------- | ----------- |
| mariadb_role_enabled | False | determine whether role is enabled (True) or not (False) |

## Dependencies

None.

## Example Playbook

```yaml
---
# role: ansible-role-mariadb
# file: site.yml

- hosts: mariadb_systems
  become: True
  vars:
    mariadb_role_enabled: True
  roles:
    - role: ansible-role-mariadb
```

## License and Author

- Author:: [jam82](https://github.com/jam82/)
- Copyright:: 2020, [jam82](https://github.com/jam82/)

Licensed under [MIT License](https://opensource.org/licenses/MIT).
See [LICENSE](https://github.com/jam82/ansible-role-mariadb/blob/master/LICENSE) file in repository.

## References

- [ArchWiki](https://wiki.archlinux.org/)
