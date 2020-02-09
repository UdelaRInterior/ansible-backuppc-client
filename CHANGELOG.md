# Change log of [ansible-backuppc-client](https://github.com/UdelaRInterior/ansible-backuppc-client) role

## [v1.2.0](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v1.2.0) 

* The unix user used by the server to access the client host is now defined as a variable, as well as it's group 

## [v1.1.0](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v1.1.0) 

* Possiblity to configure only the client access to the server, not the host client itself. New variables: 
  *  `backuppc_client`: boolean, whether to configure the client host or not,
  * `backuppc_server_home`:  home directory of user backuppc which is used to perform backups in the server,
* Ansible 2.9 compatibility,
* Variable name changes: `backuppc_db_dump` becomes `backuppc_db_server_type`


## [v1.0.0](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v1.0.0) 

* First stable version. 
* configure a host to be backed up by a backuppc server. Configure the client as well as the server,
* options to manage:
  * pre-dump and post-dump scripts in the client,  
  * dump of mysql or postgresql databases in these scripts
