BackupPC-Client Ansible role
============================

This role, backuppc_client, installs and configures client hosts of a Backuppc server. It works with a [backuppc_server](https://galaxy.ansible.com/udelarinterior/backuppc_client) role that configures the server.

It works on Debian Stretch. It should work on Ubuntu or other Debian-based systems (PR accepted).

Description
------------

This role configures the backups of hosts in a [BackupPC](https://backuppc.github.io/backuppc/) server.  It can perform the following actions:
- configure a linux user named backuppc and a backup client with this user in the backuppc server,
- configure a pre_dump and a post_dump scripts with eventual sudo rights.
- configure a mysql user named backuppc with SELECT right on all bases.
- configure a script that will dump an identified postgresql database.


Requirements
------------

This is an role for installing and maintaining the backuppc client. It will install the backuppc client on any host with an operating system that is defined here.

Note
----

At this time, this role only manages backups via rsync (+ ssh) method.

This role and backuppc_server role are based on [hanxhx/backuppc](https://galaxy.ansible.com/hanxhx/backuppc) role.

Role Variables
--------------

### Client vars

Each client configuration overrides global configuration. Thew following variables can be defined:

- `backup_state`: absent or present (default: present). If present, configures the backups of the client in the server, else eliminates the configuration
- `include_files:`: default files (directories) list of folders in the client to backup
- `exclude_files:`: default files (directories) list of folders to exclude in backups
- `backuppc_client_user`: unix user for the server to connect as to the client (default `backuppc`)
- `backuppc_client_group`: unix group of the unix user defined in the above line (default `backuppc`)
- `backuppc_client`: if set to `false` (default value `true`) the client host is not configured, only the serve to backup the host
- `backuppc_local_fetch_dir`:  home directory of user backuppc which is used to perform backups in client 
- `backuppc_server_home`:  home directory of user backuppc which is used to perform backups in the server
- `backuppc_server_name`: domain name of the BackupPC server that performs the backup and where the ssh key is fetched.
- `backuppc_users`: Users with access to the backuppc web interface who manage the backup ("user1,user2"). They must be configured in the backuppc server.

- `backuppc_scripts`: Scripts true/false flag. If true installs the scripts to launch before and after the files dump to the server. This scripts _must_ be called **`pre_dump.sh`** and **`post_dump.sh`** and should be placed in the folder `host_vars/<client_host>/files/backuppc/`.
- `backuppc_scripts_sudo`: true/false flag to give sudo rights to pre_dump.sh and post_dump.sh scripts
- `backuppc_DumpPreUserCmd` and `backuppc_DumpPostUserCmd`: ssh commands for backuppc to execute the pre_dump and post_dump scripts. These variables are pre-defined according to previous flags, but can be overwritten.
- `backuppc_sudoer` the commands authorized with sudo for backuppc user. This variable is pre-defined according to previous flags, but can be overwritten.

- `backuppc_db_server_type`:  Three states variabale that defines eventual mysql or pgsql dump before backups. The three possible values are: pgsql, mysql or null. Behaviour of scripts and db backups are different for mysql and pgsql. Particularly, for MySQL you must add yourself the scripts `pre_dump.sh` and `post_dump.sh` that do the job of dumping databases, while for PostgreSQL these scripts are built form templates (and they dump only a database). For PostgreSQL rights on the database must be previously defined, while contrarily MySQL task assign database access rights to a given user.   
- For PostgreSQL, variables are:
  - `backuppc_db_to_dump_name` defines the pgsql database to backup, 
  - `backuppc_db_dump_user` defines the pgsql user to access the database,
  - `backuppc_db_dump_user_pass` is the password of the previous users. 
  Passwords setup and privileges of this user on the database must be set elsewhere in the playbook. 
- For Mysql, variables are:
  - `backuppc_db_server_root_pass` must be set to the appropriate value if the mysql `root` user has a password defined.  By default, the variable is undefined. It must be noticed that, in recent mysql/mariadb installation, at least in debian, mysql instalation doesn't ask for and doesn't ramdomly generate a root password. Debian maintainance is no longer done with a specific user and password, but with user root through a unix sock and not a tcp authenticated sock. If the variable remains undefined, mysql tasks will be performed using debian maintenance configuration. 
  - `backuppc_db_dump_user` and `backuppc_db_dump_user_pass` are the name of the mysql user and the correspondent passwword, that will be used when calling a mysql command from a `pre_dump.sh` or `post_dump.sh` script. This user will be given SELECT access to all bases, and will be configured ad the mysql user and password for the linux user that executes the backup scripts. 
 option defines a mysql user named backuppc with .  using this user. For Pgsql option, the scripts are constructed from templates.  The database to backup is indicated in the `backuppc_db_to_dump_name` variable

- ,  and  , as well as user and password to use in the script's template.

- `xfermethod`: (O) transfer method (rsync as default)
- `more`: (O) hash with specific key/value (usefull for custom directives)
(O): Optional

### Mysql script examples

For a Mysql backup, to dump all databases before the files dump you can use the following scripts, that will take advantage of the mysql backuppc user configured by the role:

- `pre_dump.sh`
```bash
#!/bin/bash
for DataB in `mysql -e "show databases" | grep -v Database`; do mysqldump --single-transaction $DataB > "$DataB.sql"; done
tar -czvf dump.sql.tar.gz *.sql
rm *.sql
```

- `post_dump.sh`
```bash
#!/bin/bash
rm dump.sql.tar.gz
```

### About backups

This role downloads backuppc SSH public key of backuppc user in the backuppc server. You must deploy it on each client!

Dependencies
------------

Previously install a BackupPC serrver, eventually with  [ansibble-backuppc](https://galaxy.ansible.com/udelarinterior/backuppc_client) server role. 

Example Playbook
----------------

### Quick and dirty

You should look at [defaults/main.yml](defaults/main.yml).

License
-------

GPLv3

Author Information
------------------

Original role [Emilien M](https://github.com/HanXHX) enhanced by [Víctor Torterola and Daniel Viñar](https://github.com/UdelaRInterior)