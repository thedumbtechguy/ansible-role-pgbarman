# Ansible Role: PgBarman

An Ansible role for installing and configuring PgBarman.

This role does not manage users or directories and if the defaults are not used, you will need to manage those separately.

## Requirements

> This role has been tested on `Ubuntu 16.04` and `Ubuntu 16.10` only.

## Variables

### Global

- `pgbarman_user`: user to run barman as.
  - Default: `barman`
  > **NOTE**: you are responsible for creating the user if defaults are not used.

- `pgbarman_user_group`: barman user group.
  - Default: `barman`
  > **NOTE**: you are responsible for creating the group if defaults are not used.

- `pgbarman_home`: home directory of barman user.
  - Default: `/var/lib/barman`
  > **NOTE**: you are responsible for creating the directory and setting the user home if the defaults are not used.

- `pgbarman_configuration_directory`: where to read additional configuration files from.
  - Default: `/etc/barman.d`
  > **NOTE**: you are responsible for creating the directory if defaults are not used.

- `pgbarman_pg_pass`: list of credentials to put in the pg pass file. file is created when `defined` and deleted when `not defined`.
  - Default: `undefined`
  - Example:
  ```yaml
  pgbarman_pg_pass:
    - "hostname:port:database:username:password"
  ```

- `pgbarman_log_level`: logging level.
  - Default: `WARNING`
  - Options:
    - `CRITICAL`
    - `ERROR`
    - `WARNING`
    - `INFO`
    - `DEBUG`
    - `NOTSET`

- `pgbarman_compression`: compression level to use for wal files.
  - Default: `None`
  - Options:
    - `None`
    - `bzip2`
    - `gzip`
    - `pigz`
    - `pygzip`
    - `pybzip2`

- `pgbarman_backup_cron`: list of backup cron jobs to create
  - Default: `[]`
  - Example:
  ```yaml
  pgbarman_backup_cron:
    - name: pg_backup_job # name of job. this name is used to identify the job so do not change it.
      server: pg # name of server in 'pgbarman_server_configuration'. 'all' can be used for sequential backup of all servers.
      month: * # month of the year the job should run ( 1-12, *, */2, etc ). default(omit)
      weekday: * # day of the week that the job should run ( 0-6 for Sunday-Saturday, *, etc ). default(omit)
      day: * # day of the month the job should run ( 1-31, *, */2, etc ). default(omit)
      hour: * # hour when the job should run ( 0-23, *, */2, etc ). default(omit)
      minute: * # minute when the job should run ( 0-59, *, */2, etc ). default(omit)
      state: present # 'present' or 'absent'. default('present'). mark as absent if you wish to remove the job.
  ```


You can find more global configuration variables in [the global configuration](templates/barman.global.conf.j2) template.

### Servers

- `pgbarman_server_configuration`: list of server configurations.
  - Default: `[]`
  - Example:
  ```yaml
    pgbarman_server_configuration:
      - name: pg # server name/identifier. required
        description: Configuration for pg server # server description. default(name)
        config: # configuration items. each item is a name, value pair. required
          - { name: 'conninfo', value: 'host=pg user=barman dbname=postgres' }
          - { name: 'streaming_conninfo', value: 'host=pg user=streaming_barman' }
          - { name: 'backup_method', value: 'postgres' }
          - { name: 'streaming_archiver', value: 'on' }
          - { name: 'slot_name', value: 'barman' } # a slot is automatically created when this value item is defined
          - { name: 'backup_method', value: 'postgres' }
  ```

You can view the [ssh](files/ssh-server-template.conf) and [streaming](files/streaming-server-template.conf) configuration templates for guidelines.

Most of the global configuration items can be overriden in the per server as well. Read the [manual](http://docs.pgbarman.org/) for more information.

## Usage Example

```yaml
- hosts: all
  vars:
    pgbarman_pg_pass:
      - "*:*:*:barman:pa55w0rd"
      - "*:*:*:streaming_barman:p455w0rd"
    pgbarman_server_configuration:
      - name: db1
        description: streaming test for db1
        config:
            - { name: 'conninfo', value: 'host=192.168.160.143 user=barman dbname=postgres' }
            - { name: 'streaming_conninfo', value: 'host=192.168.160.143 user=streaming_barman' }
            - { name: 'backup_method', value: 'postgres' }
            - { name: 'streaming_archiver', value: 'on' }
            - { name: 'slot_name', value: 'barman' }
    pgbarman_backup_cron:
      - name: pg_backup_job
        server: pg
        weekday: 6
        hour: 0
        minute: 0
  roles:
    - thedumbtechguy.pgbarman
```


## License

MIT / BSD

## Author Information

This role was created by [TheDumbTechGuy](https://github.com/thedumbtechguy) ( [twitter](https://twitter.com/frostymarvelous) | [blog](https://thedumbtechguy.blogspot.com) | [galaxy](https://galaxy.ansible.com/thedumbtechguy/) )

## Credits

