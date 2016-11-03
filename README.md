## What is ansible-mariadb? [![Build Status](https://secure.travis-ci.org/nickjj/ansible-mariadb.png)](http://travis-ci.org/nickjj/ansible-mariadb)

It is an [Ansible](http://www.ansible.com/home) role to install and configure
MariaDB.

##### Supported platforms:

- Ubuntu 16.04 LTS (Xenial)
- Debian 8 (Jessie)

### What problem does it solve and why is it useful?

Setting up and securing a database is tedious and error prone. This role sets
you up with sane defaults and weekly backups.

## Role variables

Below is a list of default values along with a description of what they do.

```
---

# Which version of MariaDB do you want to install?
mariadb_upstream_version: '10.1'

# The bind address and port of the server.
mariadb_mysqld_bind_address: 'localhost'
mariadb_mysqld_port: 3306

# Your root password, use something secure!
mariadb_mysqld_root_password: 'pleasepickabetterpassword'

# Should the server get restarted on a config change? Most of the time you'll
# want this set to True but if you know what you're doing you may want to
# handle restarting the service manually for the sake of uptime.
mariadb_mysqld_retsart_on_config_change: True

# Should the role perform automated backups? Files will get automatically dumped
# to /var/lib/automysqlbackup if enabled. They will be properly rotated and
# backups will occur every Saturday (day 6).
#
# When set to False, no backups will happen.
mariadb_mysqld_backup: True
mariadb_mysqld_backup_weekly_backup_day: 6
mariadb_mysqld_backup_createdb: 'yes'

# The python-mysqldb package is required to work with Ansible. Feel free to add
# your own dependencies to this list. If you have backups enabled make sure
# not to remove the automysqlbackup package from this list.
mariadb_mysqld_dependencies:
  - 'python-mysqldb'
  - '{{ "automysqlbackup" if mariadb_mysqld_backup | bool else [] }}'

# MariaDB has a number of configuration options, it would be madness to template
# out each one.
#
# Simply add any valid MariaDB config option to the appropriate list. The list
# name matches up to the [foo] name in a MariaDB config.
#
# If you don't like these defaults then set your own or use an empty list. This
# sets up UTF-8 encoding.
mariadb_default_client_config:
  - 'default-character-set = utf8'
mariadb_default_mysqld_config:
  - 'collation-server = utf8_unicode_ci'
  - 'init-connect = "SET NAMES utf8"'
  - 'character-set-server = utf8'
mariadb_default_mysqld_safe_config: []
mariadb_default_galera_config: []
mariadb_default_mysqldump_config: []
mariadb_default_mysql_config: []
mariadb_default_isamchk_config: []

# If you want to append your own values to the defaults, add them here. This
# way you don't have to duplicate all of the default config items in your
# custom list. Both the default list and this list will be joined together.
mariadb_client_config: []
mariadb_mysqld_config: []
mariadb_mysqld_safe_config: []
mariadb_galera_config: []
mariadb_mysqldump_config: []
mariadb_mysql_config: []
mariadb_isamchk_config: []

# The APT GPG key id used to sign the MariaDB package.
mariadb_apt_keys: ['0xcbcb082a1bb943db', '0xF1656F24C74CD1D8']

# The OS distribution and distribution release, thanks https://github.com/debops.
# Doing it this way doesn't depend on having lsb_release installed.
mariadb_distribution: '{{ ansible_local.core.distribution
                         if (ansible_local|d() and ansible_local.core|d() and
                             ansible_local.core.distribution|d())
                         else ansible_distribution }}'
mariadb_distribution_release: '{{ ansible_local.core.distribution_release
                                 if (ansible_local|d() and ansible_local.core|d() and
                                     ansible_local.core.distribution_release|d())
                                 else ansible_distribution_release }}'

# Address of the MariaDB repository.
mariadb_repository: 'deb http://nyc2.mirrors.digitalocean.com/mariadb/repo/{{ mariadb_upstream_version }}/{{ mariadb_distribution | lower }} {{ mariadb_distribution_release }} main'

# How long should the apt-cache last in seconds?
mariadb_apt_cache_time: 86400
```

## Example playbook

For the sake of this example let's assume you have a group called **app** and
you have a typical `site.yml` file.

To use this role edit your `site.yml` file to look something like this:

```
---

- name: Configure app server(s)
  hosts: app
  become: True

  roles:
    - { role: nickjj.mariadb, tags: mariadb }
```

Let's say you want to backups to run on Monday and tweak a mysqld config setting.
Start by opening or creating `group_vars/app.yml` which is located relative to
your `inventory` directory and then making it look like this:

```
---

# Always set your own root password.
mariadb_mysqld_root_password: 'yourcustompassword'

# 6 = Saturday, 1 = Monday, 2 = Tuesday, etc..
mariadb_mysqld_backup_weekly_backup_day: 1

# Allocate 75% of the system's RAM to the innodb buffer pool size.
mariadb_mysqld_config:
  - 'innodb_buffer_pool_size = {{ (ansible_memtotal_mb * 0.75) | int }}M'
```

## Installation

`$ ansible-galaxy install nickjj.mariadb`

## Ansible Galaxy

You can find it on the official
[Ansible Galaxy](https://galaxy.ansible.com/nickjj/mariadb/) if you want to
rate it.

## License

MIT

## Special thanks

Thanks to [Maciej Delmanowski](https://twitter.com/drybjed) for his work done
on the
[DebOps mariadb_server role](https://github.com/debops/ansible-mariadb_server).
This role is quite inspired by his.
