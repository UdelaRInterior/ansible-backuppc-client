BackupPC-Client Ansible role
============================

This role, backuppc_client, configures both sides for client hosts of a Backuppc server. It has a sister role [backuppc_server](https://galaxy.ansible.com/udelarinterior/backuppc_server) that configures the server (However, as far as the configuration is standard and Ansible has access, it can handle almost any backuppc server installation). 

It works on Debian Buster (10) and Stretch (9) for advanced configuration (databases backup) but for basic backuppc dump configuration, it can handle Ubuntu or any other Debian-based systems (PR accepted).

This role and its sister [backuppc_server](https://galaxy.ansible.com/udelarinterior/backuppc_server) are based on [hanxhx/backuppc](https://galaxy.ansible.com/hanxhx/backuppc) role.


Description
------------

This role configures the backups of hosts in a [BackupPC](https://backuppc.github.io/backuppc/) server.  It can perform the following actions:
- Set up a linux user in the client and its backup configuration in the BackupPC server, which access the client with this user.
- Optional upload to the client or template of a pre dump and a post dump scripts with eventual sudo rights, which are executed respectively before and after backuppc dumps files. These scripts usually run commands needed to have coherent backups, such as database dumps, snapshots, etc. 
- Optional set up of a mysql user with SELECT rights on all bases, to perform a dump before backup.
- Optional set up of a script that will dump an identified postgresql database.

Requirements
------------

We will need to have a running backuppc server managed in your Ansible inventory, whose name will be defined in `backuppc_server_name` variable. We can achieve that with our [`backuppc_server` role](https://galaxy.ansible.com/udelarinterior/backuppc_server). 

At this time, this role only manages backups via rsync (+ ssh) method.

Backwards' compatibility
------------------------
To ensure a smooth and progressive adoption of this version of the role accross all the hosts that use it in a cloud envireonement, the role is backwards compatible with [the v2.X.Y](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v2.1.1) variables' role API, including its default values. See [defaults/main.yml](defaults/main.yml) and [tasks/compatibility.yml](tasks/compatibility.yml) files, particularly comments, for legacy variables' considerations. However, as announced in v2.0.0 release, the role is no longer compativle with v1.X.Y API. Don't use this version if you didn't update your host variables to v2.X.Y API!

Backwards role's compatibility may be dorped in next major release, so adapt your hosts variables to new API asap!

Role Variables
--------------

Each client configuration overrides global configuration in the server. See [defaults/main.yml](defaults/main.yml) for default variables values or definition. Hereafter are listed the variables that can be defined.

- `backuppc_server_name`: domain name of the BackupPC server that performs the backup and where the ssh key is fetched to be configured on the client

### Client access

- `backuppc_client_user`: unix user for the server to connect as to the client (default `backuppc`)
- `backuppc_client_group`: unix group of the unix user defined in the above line (default `backuppc`)
- `backuppc_client_home`:  home directory of `backuppc_client_user` user that performs backups in the client  (default `/var/lib/backuppc`)

### Server configuration

- `backuppc_server_user`: unix users that runs BackupPC in the server. Defaults to `backuppc`.
- `backuppc_server_group`: unix group that runs BackupPC in the server. Defaults to `www-data`.
- `backuppc_server_home`: home directory of BackupPC unix user in the server. Defaults to `/var/lib/backuppc`.
- `backuppc_server_config_dir`: BackupPC package configuration files' directory. Defaults to `/etc/backuppc`. 

### Client's backup configuration in the server

Here are briefely described the role's variables that define BackupPC configuration of the client in the server. For a full documentation see [BackupPC documentation](https://backuppc.github.io/backuppc/BackupPC.html#Configuration-Parameters) itself, or the [config.pl.j2 template comments](https://github.com/UdelaRInterior/ansible-backuppc/blob/v3.0.0/templates/etc/backuppc/config.pl.j2) of the server role.

The following flags define whether client and server are configured by the role: 

- `backuppc_backup_state`: absent or present (default: present). If present, configures the backups of the client in the server, else eliminates the configuration
- `backuppc_client`: if set to `false` (default value `true`) there is no ssh access or other configuration in the client host, only the server is configured to backup the host
#  
- `backuppc_BackupsDisable`: Whether automatic backups are enabled and manual ones remain possible. By default (0), backups are enabled and run according to schedule. With 1, automatic backups are disabled and maual backups are possible. With 2, all backups are disabled

#### What to backup

- `backuppc_RsyncShareName`: list of folder's tree points in the file system of the client to backup. For instance: 
```
backuppc_RsyncShareName:
- /etc
- /root
- /var
- /usr/local
```
With rsync method, BackupPC will perform an rsync command for each element of this list. 
Note: as stated above, only rsync transfer method is considered as transfer method, so we don't mention `SmbShareName`, `TarShareName` or other similar BacupPC configuration parameters, or related. 

- `backuppc_BackupFilesOnly`: List of directories or files to include in backups (the --exclude options of rsync will be built with this variable). This can be set to a string, a list of strings, or, in the case of multiple shares, a dict of strings or lists. A dict is used to give a list of directories or files to backup for each share (the share name is the key). If a dict is used, a special key `"*"` means it applies to all shares that don't have a specific entry. By default, this variables is undefined, so the BackupPC parameters will be empty.

For instance, the following configuration parameters: 
```
backuppc_RsyncShareName:
- /etc/gitlab
- /var/opt/gitlab

backuppc_BackupFilesOnly:
  # Configuration archives fo GitLab instance:
  "/etc/gitlab":
    - /gitlab.rb
    - /gitlab-secrets.json
  # Gitlab backup file
  "/var/opt/gitlab":
    - /backups/dump_backuppc_gitlab_backup.tar
```
will perform the backup of the three needed file of a GitLab instance. (`dump_backuppc_gitlab_backup.tar` is built by a script just before the dump, see hereafter) 

- `backuppc_BackupFilesExclude`: list of folders to exclude from the client's backup (the --exclude options of rsync will be built with this variable). Its structure and syntax is similar to previous variable `backuppc_BackupFilesOnly`. By default, this variables is undefined, so the BackupPC parameters will be empty.

- `backuppc_XferMethod`: Optional transfer method (rsync as default and, as mentionned, the only one option for the role)

- `backuppc_more`: Optional dict variable with specific parameter/value configuration of BackupPC. For a full documentation of parameters see [BackupPC documentation](https://backuppc.github.io/backuppc/BackupPC.html#Configuration-Parameters) itself, or the [config.pl.j2 template comments](https://github.com/UdelaRInterior/ansible-backuppc/blob/v3.0.0/templates/etc/backuppc/config.pl.j2) of the server role. For instance with the following values whe keep incremental backups por six months and full backups for two years, with a minimum of ten of each:
```yaml
backuppc_more:
  ## 105 weeks, 2 years
  FullKeepCnt: 105
  FullKeepCntMin: 10
  FullAgeMax: 730
  ## 183 days, half a year
  IncrKeepCnt: 183
  IncrKeepCntMin: 10
  IncrAgeMax: 730
```

### Pre and post scripts 

The following variables allow to define the configuration and execution of pre and post files' dump scripts, BackupPC will execute respectively before and after dumping files, during a backup cycle. 

The two following variables of script's paths are undefiend by default and then scripts are not configured at all. When one of them is defiend, the corresponding script will be uploaded or templated and configured to be executed during backups.

Paths can be absolute or relative path. If a path is relative, the home dir of backuppc user, `backuppc_server_home`, is prependend to the script's path.

- `backuppc_pre_dump_script`: path on the backuppc client of the file of the pre dump script, that will be executed before files dump during a backup. 
- `backuppc_post_dump_script`: path on the backuppc client of the file of the post dump scriptthat will be executed after files dump during a backup. 

- `backuppc_scripts_local_dir`: path in the local Ansible controller where the playbook will find the two previous scripts to install them in the client. Its default value is `'{{ playbook_dir }}/host_vars/{{ inventory_hostname }}/files/backuppc/'`, i.e. a folder called `files/backuppc`, aside the `vars` folder of the host you are configuring.

For instance, we could define:
```
backuppc_pre_dump_script: scripts/pre_dump.sh 
backuppc_post_dump_script: scripts/post_dump.sh
```

 Therefore, in the directories structure of your playbook, you will have to put the pre and post dump scripts, in files with the same basenames than their respective path hereabove, in : 
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
The pre and post dump scripts will end up in the following files: 
- `'{{ backuppc_client_home }}/scripts/post_dump.sh'`
- `'{{ backuppc_client_home }}/scripts/pre_dump.sh'`


Scripts can also be templated from the playbooks host variables: 
- `backuppc_pre_dump_template`: Path to the Jinja2 template to build the pre dump script.
- `backuppc_post_dump_template`: Path to the Jinja2 templates to build the post dump script.

If any of the two previous variables are defiend, the scripts will be templated with the instead of uploaded form previous directory. We must set the variable to a path valid for the Ansible playbook. For instance, with: 
```
backuppc_pre_dump_template: '{{ playbook_dir }}/host_vars/{{ inventory_hostname }}/templates/pre_dump.sh.j2'
backuppc_post_dump_template: '{{ playbook_dir }}/host_vars/{{ inventory_hostname }}/templates/post_dump.sh.j2'
```
the role will take its templates from a folder `/templates` aside host's variables folder. 

The following flags allow to execute scripts with sudo: 

- `backuppc_scripts_sudo`: true/false Flag to execute pre dump and post dump scripts with sudo
- `backuppc_script_pre_sudo`: false/ture Flag to execute pre dump script with sudo
- `backuppc_script_post_sudo`: false/true Flags to execute post dump script with sudo

The first flag is just a shortcut. If second or third flag is defined differently than first, its configuration prevails.

- `backuppc_DumpPreUserCmd` and `backuppc_DumpPostUserCmd`: ssh commands for backuppc to execute the pre_dump and post_dump scripts. These variables are pre-defined according to previous flags, but can be overwritten (you must know what you are doing)

- `backuppc_sudoer`: the sudoers dconfig line of commands authorized with sudo for `backuppc_client_user` user. This variable predefined; it includes rsync, and eventually pre and post dump scripts, according to previous flags. If you know what you are doing, it is a string that can be overwritten, as far as it starts with `Cmnd_Alias BACKUPS =`, followed by the list of shell commands that the user needs to exec ute to perform a backup. 

### PostgreSQL and MySQL dumps for backups

The following variables give some tools to define, using hereabove described scripts (so you can not use them simultaneously on your own, particularly for PostgreSQL), the dump of databases for backups coherency: 

- `backuppc_db_server_type`:  Three states variabale that defines eventual mysql or pgsql dump before backups. The three possible values are: pgsql, mysql or null. Behaviour of scripts and db backups are different for mysql and pgsql. Particularly, for MySQL you must add yourself the scripts, por instance `pre_dump.sh` and `post_dump.sh` that do the job of dumping databases, while for PostgreSQL these scripts are built form templates (and they dump only a database). For PostgreSQL rights on the database must be previously defined, while contrarily MySQL task assigns database access rights to a given user.   
- For PostgreSQL, variables are:
  - `backuppc_db_to_dump_name` defines the pgsql database to backup, 
  - `backuppc_db_dump_user` defines the pgsql user to access the database,
  - `backuppc_db_dump_user_pass` is the password of the previous users. 
  Passwords setup and privileges of this user on the database must be set elsewhere in the playbook. 
- For Mysql, variables are:
  - `backuppc_db_server_root_pass` should be set to the appropriate value, if the mysql `root` user has a password defined. By default, the variable is undefined. It must be noticed that, in recent mysql/mariadb installation, at least on Debian, installation doesn't ask for and doesn't ramdomly generate a root password. Debian maintainance is no longer done with a specific user and password, but with user root through a unix sock and not a tcp authenticated sock. If the variable remains undefined, mysql tasks will be performed using debian maintenance configuration. 
  - `backuppc_db_dump_user` and `backuppc_db_dump_user_pass` are the names of the mysql user and the correspondent password, that will be given SELECT access to all databases and will be configured as default in the `.my.cnf` file in the home directory of the `backuppc_client_user` unix user, to allow access from the linux user that executes the backup scripts. Therefore `pre_dump.sh` or `post_dump.sh` scripts will be able to perform any database dump calling a simple mysql command, without specifiyng user or password.
  - `backuppc_db_dump_user_priv` are the privileges for `backuppc_db_dump_user` to perform backups. For example: `'*.*:PROCESS,SUPER,SELECT` or `*.*:RELOAD,PROCESS,LOCK TABLES,REPLICATION CLIENT`.
  - `backuppc_db_dump_group_options` are the specific group options mariadb/mysql tools. For example: `client` (default) or `mariabackup`. 

#### Mysql script examples

For a Mysql backup, to dump all databases before the files dump you can use the following scripts, that will take advantage of the mysql backuppc user configured by the role:

- `pre_dump.sh` script:
```bash
#!/bin/bash
for DataB in `mysql -e "show databases" | grep -v Database`; do mysqldump --single-transaction $DataB > "$DataB.sql"; done
tar -czvf dump.sql.tar.gz *.sql
rm *.sql
```

- `post_dump.sh` script: 
```bash
#!/bin/bash
rm dump.sql.tar.gz
```

### Web users with access to backups

BackupPC users are configured, with their credentials and other data (mail) in the BackupPC server to have web access. For instance, with our [`backuppc_server` role](https://github.com/UdelaRInterior/ansible-backuppc/), they are defined with the variable `backuppc_srv_web_users`. Their usernames can be set in the following variables, to give web access to the client host's backups on BackupPC web interface:  
- `backuppc_server_web_main_user`: Main user with access to the client host's backups through BackupPC Web interface. Defaults to `backuppc`. She also recieves administrative e-mails.
- `backuppc_server_web_other_users`: Other users with access to the client host's backups through BackupPC Web interface Must be defined as a string listing users separated by commas: "user1,user2". 

Example Playbook
----------------
We consider we have a standard BackupPC instance running at  `bck-server.domain.org` managed through Ansible inventory. The following playbook configures in this server the backup of the specified folders of `client.domain.org` host, with needed ssh access. 


```YAML
- name: Backup client.domain.org host
  hosts: client.domain.org
  become: true
  vars: 
    backuppc_server_name: bck-server.domain.org
    backuppc_RsyncShareName:
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