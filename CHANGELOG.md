# Change log of [ansible-backuppc-client](https://github.com/UdelaRInterior/ansible-backuppc-client) role

## [v2.1.0](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v2.1.0) 

* resolution of marginal bug when include or explude files lists are empty, that appears with default variables value's of v1.X.0, that we preserve for backwards compatibility
* per and post dumps scripts names and placement parametrisation
* simplification ofo some formulas
* consistent and extended documentation

## [v2.0.0](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v2.0.0)

* Managment of `backuppc_rsync_share_names` configuration for clients, in order to achieve backups with several rsync
* Standardisation of API: all variables are in namespace `backuppc_`
* Backwards variables' compatibility and default values. Can change in future versions, adopt new API!
* Some new parameters, previously hard coded
* Documentation completion and reorganisation

## [v1.3.0](https://github.com/UdelaRInterior/ansible-backuppc-client/tree/v1.3.0) 

* Change the place of mysql credentials, in order to affect only backuppc user, not root
* now accepts mysql without root password defined, as is now the default installation in debian  
* now the role erases eventual old keys of the backed up host in the known_host of backuppc server 

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
