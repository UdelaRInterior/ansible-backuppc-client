BackupPC-Client Ansible role
============================

This role, backuppc_client, installs and configures client hosts of a Backuppc server. It works with a [backuppc_server](https://github.com/UdelaRInterior/ansible-backuppc) role that configures the server. 

It works on Debian Stretch. It should work on Ubuntu or other Debian-based systems, but no direct support will be added (PR accepted).

Description
------------

This role configures the backups of hosts in a backuppc server.  It can perform the following actions: 
- configure a linux user named backuppc and a backup client with this user in the backuppc server, 
- configure a mysql user named backuppc with SELECT right on all bases,
- configure a pre_dump and a post_dump scripts with eventual sudo rights. 


Requirements
------------

This is an role for installing and maintaining the backuppc client. It will install the backuppc client on any host with an operating system that is defined here.

Note
----

At this time, this role only manages backups via rsync (+ ssh) method.

This role and backuppc_server role are based on [hanxhx/backuppc](https://galaxy.ansible.com/hanxhx/backuppc) role

Role Variables
--------------

### Client vars

Each client configuration overrides global configuration.

- `state`: absent or present (default: present). If present, configures the backups of the client in the server, else eliminates the configuration 
- `include_files:`: default files (directories) list of folders in the client to backup
- `exclude_files:`: default files (directories) list of folders to exclude in backups
- `backuppc_local_fetch_dir`:  home directory of user backuppc which is used to perform backups (in client and server) 

- `backuppc_mysql_dump`:  true/false flag for a mysql dump before backups. If true, defines a mysql user named backuppc with SELECT access to all bases (the mysql dump itself must be executed in following scripts)
- `backuppc_scripts`: Scripts true/false flag. If true installs the scripts to launch before and after the files dump to the server. This scripts _must_ be called **`pre_dump.sh`** and **`post_dump.sh`** and should be placed in the folder `host_vars/<client_host>/files/backuppc/`.
- `backuppc_scripts_sudo`: true/false flag to give sudo rights to pre_dump.sh and post_dump.sh scripts
- `backuppc_DumpPreUserCmd` and `backuppc_DumpPostUserCmd`: ssh commands for backuppc to execute the pre_dump and post_dump scripts. These variables are pre-defined according to previous flags, but can be overwritten.
- `backuppc_sudoer` the commands authorized with sudo for backuppc user. This variable is pre-defined according to previous flags, but can be overwritten. 

- `backuppc_users`: Users with access to the backuppc web interface who manage the backup ("user1,user2"). They must be configured in the backuppc server. 

- `xfermethod`: (O) transfer method (rsync as default)
- `more`: (O) hash with specific key/value (usefull for custom directives)

(O): Optional

### About backups

This role downloads backuppc SSH public key of backuppc user in the backuppc server. You must deploy it on each client!

Dependencies
------------

Previously install backuppc-server https://galaxy.ansible.com/hanxhx/backuppc

Example Playbook
----------------

### Quick and dirty

You should look at [defaults/main.yml](defaults/main.yml).

License
-------

GPLv2

Author Information
------------------

Original role [Emilien M](https://github.com/HanXHX) enhanced by [Víctor Torterola and Daniel Viñar](https://github.com/UdelaRInterior)