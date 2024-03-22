BackupPC-Client Ansible role
============================

This role, backuppc_client, installs and configures both sides for client hosts of a Backuppc server. It works with a [backuppc_server](https://galaxy.ansible.com/udelarinterior/backuppc_server) role that configures the server (However, as far as the configuration is standard and Ansible has access, it can handle any backuppc server installation). 

It works on Debian Buster (10) and Stretch (9) for advanced configuration (databases backup) but for basic backuppc dump configuration, it can handle Ubuntu or any other Debian-based systems (PR accepted).

This role and its peer [backuppc_server](https://galaxy.ansible.com/udelarinterior/backuppc_server) are based on [hanxhx/backuppc](https://galaxy.ansible.com/hanxhx/backuppc) role.


Description
------------

This role configures the backups of hosts in a [BackupPC](https://backuppc.github.io/backuppc/) server.  It can perform the following actions:
- Set up a linux user in the client and a backup connfiguration the backuppc server, which access the client with this user.
- Optionnal upload to the client a pre_dump and a post_dump scripts with eventual sudo rights, which are executed respectively before and after backuppc dumps files. This scripts run commans needed to have coherent backups, such as database dumps, snapshots, etc. 
- Optionnal set up of a mysql user with SELECT right on all bases, to perform a dump before backup.
- Optionnal set up of a script that will dump an identified postgresql database.

Requirements
------------

You will need to have a running backuppc server managed in your Ansible inventory, whose name will be defined in `backuppc_server_name` variable. 

At this time, this role only manages backups via rsync (+ ssh) method.

Backwards' compatibility
------------------------
To ensure a smooth and progressive adoption of this version of the role accross all the hosts that use it in a cloud envireonement, the role is backwards compatible with [the v1.X.0](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v1.3.0) variables' role API. including its default values. See [defaults/main.yml](defaults/main.yml) and [tasks/compatibility.yml](tasks/compatibility.yml) files, particularly comments, for legacy variables' consideration.

Backwards role's compatibility will be dorped in next major release, so adapt your hosts variables API asap!

Role Variables
--------------

Each client configuration overrides global configuration in the server. See [defaults/main.yml](defaults/main.yml) for default variables values or definition. Hereafter are listed the variables that can be defined.

### Client access

- `backuppc_client_user`: unix user for the server to connect as to the client (default `backuppc`)
- `backuppc_client_group`: unix group of the unix user defined in the above line (default `backuppc`)
- `backuppc_client_home`:  home directory of `backuppc_client_user` user that performs backups in the client 

### Server configuration

- `backuppc_server_name`: domain name of the BackupPC server that performs the backup and where the ssh key is fetched to be configured on the client
- `backuppc_server_user`: unix users that runs BackupPC in the server. Defaults to `backuppc`.
- `backuppc_server_group`: unix group that runs BackupPC in the server. Defaults to `www-data`.
- `backuppc_server_home`: home directory of BackupPC unix user in the server. Defaults to `/var/lib/backuppc`.
- `backuppc_server_config_dir`: BackupPC package configuration files' directory. Defaults to `/etc/backuppc`. 

### Client's backup configuration in the server

Here are briefely described the role's variables that define BackupPC configuration of the client in the server. For a full documentation see [BackupPC documentation](https://backuppc.github.io/backuppc/BackupPC.html#Configuration-Parameters) itself.

The following flags define whether client and server are configured by the role: 

- `backuppc_backup_state`: absent or present (default: present). If present, configures the backups of the client in the server, else eliminates the configuration
- `backuppc_client`: if set to `false` (default value `true`) there is no ssh access or other configuration in the client host, only the server is configured to backup the host

The client's configuration file in BackupPC server is built with the following variables:

- `backuppc_rsync_share_names`: list of folder's tree points in the client to backup. For instance: 
```
backuppc_rsync_share_names:
- /etc
- /root
- /var
- /usr/local
```
With rsync method, BackupPC will perform an rsync command for each element of this list. 
- `backuppc_include_files:`: list of folders to include in the client's backup (the --include option of rsync will be built with this variable)
- `backuppc_exclude_files:`: list of folders to exclude from the client's backup (the --exclude option of rsync will be built with this variable)
- `backuppc_xfermethod`: Optional transfer method (rsync as default)
- `backuppc_more`: Optional hash with specific key/value (usefull for custom directives, such as the backup scheduling and preservation). See example in [defaults/main.yml](defaults/main.yml) file.

The following variables allow to define the execution of pre and post files' dump scripts by BackupPC: 
- `backuppc_pre_dump_script`: path of the file of the pre dump dump script, that BackupPC will execute before dumping files during a backup. It's default value is `'{{ backuppc_client_home }}/scripts/pre_dump.sh'`
- `backuppc_post_dump_script`: path of the file of the post dump dump script, that BackupPC will execute after dumping files during a backup. It's default value is `'{{ backuppc_client_home }}/scripts/post_dump.sh'`
- `backuppc_scripts_local_dir`: path in the local Ansible controller where the playbook will find the two previous scripts to install them in the client. Its default value is `'{{ playbook_dir }}/host_vars/{{ inventory_hostname }}/files/backuppc/'`. Therefore, in the directories structure of your playbook, you will have to put the pre and post dump scripts, in files with the same basenames than their respective path hereabove, in a folder called `files/backuppc`, aside the `vars` folder of the host you are configuring: 
```
 host_vars
 └── <your_host>
     ├── files
     │   └── backuppc
     │       ├── post_dump.sh
     │       └── pre_dump.sh
     └── vars
         ├── 10_kvm_virtual.yml
         └── 20_backuppc.yml
```
- `backuppc_scripts`: Scripts true/false flag. If true installs the pre and post dump scripts previously described. This flag is deprecated. It is preserved for backwards compatibility, but, in the next major version, it will be droped, and the scripts will be installed when their path is defined. 
- `backuppc_scripts_sudo`: true/false flag to give sudo access rights to pre and post dump scripts
- `backuppc_DumpPreUserCmd` and `backuppc_DumpPostUserCmd`: ssh commands for backuppc to execute the pre_dump and post_dump scripts. These variables are pre-defined according to previous flags, but can be overwritten.

- `backuppc_sudoer` the commands authorized with sudo for `backuppc_client_user` user. It is a string that can be overwritten, as far as it starts with `Cmnd_Alias BACKUPS =`, followed by the list of shell commands that the user needs to exec ute to perform a backup. As predefined, this variable includes rsync, and eventually pre and post dump scripts, according to previous flags.

The following variables give some tools to define, using hereabove described scripts, the dump of databases for backups coherency: 

- `backuppc_db_server_type`:  Three states variabale that defines eventual mysql or pgsql dump before backups. The three possible values are: pgsql, mysql or null. Behaviour of scripts and db backups are different for mysql and pgsql. Particularly, for MySQL you must add yourself the scripts `pre_dump.sh` and `post_dump.sh` that do the job of dumping databases, while for PostgreSQL these scripts are built form templates (and they dump only a database). For PostgreSQL rights on the database must be previously defined, while contrarily MySQL task assign database access rights to a given user.   
- For PostgreSQL, variables are:
  - `backuppc_db_to_dump_name` defines the pgsql database to backup, 
  - `backuppc_db_dump_user` defines the pgsql user to access the database,
  - `backuppc_db_dump_user_pass` is the password of the previous users. 
  Passwords setup and privileges of this user on the database must be set elsewhere in the playbook. 
- For Mysql, variables are:
  - `backuppc_db_server_root_pass` should be set to the appropriate value if the mysql `root` user has a password defined.  By default, the variable is undefined. It must be noticed that, in recent mysql/mariadb installation, at least in Debian, mysql installation doesn't ask for and doesn't ramdomly generate a root password. Debian maintainance is no longer done with a specific user and password, but with user root through a unix sock and not a tcp authenticated sock. If the variable remains undefined, mysql tasks will be performed using debian maintenance configuration. 
  - `backuppc_db_dump_user` and `backuppc_db_dump_user_pass` are the name of the mysql user and the correspondent password, that will be given SELECT access to all bases and will be configured as default in the `.my.cnf` file in the home directory of the `backuppc_client_user` unix user.  for the linux user that executes the backup scripts. Therefore `pre_dump.sh` or `post_dump.sh` scripts will be able to perform any database dump calling a simple mysql command. 
  - `backuppc_db_dump_user_priv` are the privileges for `backuppc_db_dump_user` to perform backups. For example: `'*.*:PROCESS,SUPER,SELECT` or `*.*:RELOAD,PROCESS,LOCK TABLES,REPLICATION CLIENT`.
  - `backuppc_db_dump_group_options` are the specific group options mariadb/mysql tools. For example: `client` (default) or `mariabackup`. 

The following variables configure the web access for the client host's backups on BackupPC web interface:  
- `backuppc_server_web_main_user`: # Main user with access to the client host's backups through BackupPC Web interface. Defaults to `backuppc`.
- `backuppc_server_web_other_users`: Other users with access to the client host's backups through BackupPC Web interface Must be defined as a string listing users separated by commas: "user1,user2". User's access must be configured in the BackupPC server.

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

Example Playbook
----------------
We consider we have a standard BackupPC instance running at  `bck-server.domain.org` managed through Ansible inventory. The following playbook configures in this server the backup of the specified folders of `client.domain.org` host, with needed ssh access. 


```YAML
- name: Backup client.domain.org host
  hosts: client.domain.org
  become: true
  vars: 
  - backuppc_server_name: bck-server.domain.org
  - backuppc_rsync_share_names:
    - /etc
    - /var
    - /opt

  roles: 
  - role: udelarinterior.backuppc_client
```

License
-------

GPLv3

Author Information
------------------

Original role [Emilien M](https://github.com/HanXHX) enhanced by [Víctor Torterola and Daniel Viñar](https://github.com/UdelaRInterior)