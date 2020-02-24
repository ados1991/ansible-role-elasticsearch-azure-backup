ansible-role-elasticsearch-azure-backup
=========

Manage backup of data nodes on azure storage.

Requirements
------------

Elastisearch must be installed on your hosts.
This role is tested with elasticsearch version==7.3.2

Warnings Note
------------

Due to restart of elasticsearch service because of some changes, in case of you are using the role with a cluster.
You must use a rolling update playbook for applying a proper rolling restart on cluster.

Eg:

    ---
    - hosts: elasticsearch_cluster
      max_fail_percentage: 50
      serial: "50%"
      roles:
        - ansible-role-elasticsearch-azure-backup
        
For more details :
- https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html#rolling-update-batch-size

Role Variables
--------------

Required vars (used by main.yml):

elasticsearch_azure contains all client accounts that you can set for your elasticsearch cluster.
this accounts are authentification accounts used to backup and store snaphots on azure storage.
the struture of the var is below :

    elasticsearch_azure:
      client:
        account_1:
            account (Required, desc: storage account name in azure)
            endpoint_suffix (Optional, default: core.windows.net)
            key (Required, desc: secret key for access)
            max_retries (Optional, default: 3)
            proxy.host (Optional)
            proxy.port (Optional)
            proxy.type (Optional)
            sas_token (Optional)
            timeout (Optional, default: 10s)
        account_2:
            ...
Eg: 

    elasticsearch_azure:
      client:
        default:
          account: vdc1backupelasticsearch
          key: secret_key
        primary:
          account: vdc1backupelasticsearch
          key: another_secret_key
          max_retries: 10
          timeout: 10s


For more details : 
 - https://www.elastic.co/guide/en/elasticsearch/plugins/current/repository-azure-client-settings.html

Optional vars that can be overidden (used by main.yml):

    elasticsearch_conf_dir: "/etc/elasticsearch"
    elastisearch_plugin_bin: "/usr/share/elasticsearch/bin/elasticsearch-plugin"
    elasticsearch_home: "/usr/share/elasticsearch"
    elasticsearch_user: "elasticsearch"
    elasticsearch_group: "elasticsearch"

Required vars (used by create_backup.yml):


    action: create_backup
    elasticsearch_azure_backup_name: backup_name
    elasticsearch_azure_backup_config:
        type: azure
        settings:
            client (Azure named client to use. Defaults to default.)
            container (Container name. You must create the azure container before creating the repository. Defaults to elasticsearch-snapshots.)
            base_path (Specifies the path within container to repository data. Defaults to empty (root directory).)
            chunk_size (Big files can be broken down into chunks during snapshotting if needed. Specify the chunk size as a value and unit, for example: 10MB, 5KB, 500B. Defaults to 64MB (64MB max).)
            compress (When set to true metadata files are stored in compressed format. This setting doesn’t affect index files that are already compressed by default. Defaults to true.)
            max_restore_bytes_per_sec (Throttles per node restore rate. Defaults to 40mb per second.)
            max_snapshot_bytes_per_sec (Throttles per node snapshot rate. Defaults to 40mb per second.)
            readonly (Makes repository read-only. Defaults to false.)
            location_mode (primary_only or secondary_only. Defaults to primary_only. Note that if you set it to secondary_only, it will force readonly to true.)
            
Eg:

    action: create_backup
    elasticsearch_azure_backup_name: backup_name
    elasticsearch_azure_backup_config:
        type: azure
        settings:
            "client": "primary"
            "container": "backups"
            "base_path": "elas"

For more details :
 - https://www.elastic.co/guide/en/elasticsearch/plugins/master/repository-azure-repository-settings.html
    
Required vars (used by delete_backup.yml):
    
    action: delete_backup
    elasticsearch_azure_backup_name: backup_name
    
Required vars (used by take_snapshot.yml):

    action: take_snapshot
    elasticsearch_azure_backup_name: backup_name
    elasticsearch_azure_snapshot: snapshot_name
    elasticsearch_azure_snapshot_config:
      indices (specify which indices to backup.)
      ignore_unavailable (if index doesn’t exist skip to the next index in the list otherwise break execution if is set to false.)
      include_global_state (setting it to false prevent Elastic to put the global cluster state from being put in the snapshot, this allow to restore the snapshot on another cluster with different attributes.)
      metadata:
        key1: value1
        key2: value2

Eg:

    elasticsearch_azure_backup_name: backup_name
    elasticsearch_azure_snapshot: "azure_snapshot"
    elasticsearch_azure_snapshot_config:
      indices: "myindex"
      ignore_unavailable: true
      include_global_state: false
      metadata:
        taken_by: "DevOps"
        taken_because: "backup before upgrading"

For more details :
- https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html

Optional vars that can be overidden (used by create_backup.yml, delete_backup.yml, take_snapshot.yml):

    elasticsearch_api_scheme: "http"
    elasticsearch_api_host: "localhost"
    elasticsearch_api_port: 9200
    elasticsearch_api_url: "{{ elasticsearch_api_scheme }}://{{ elasticsearch_api_host }}:{{ elasticsearch_api_port }}"
    elasticsearch_validate_certs: "no"

Dependencies
------------

No dependencies.

Example Playbook
----------------

The call below will install repository-azure plugin and configure settings for elastisearch to be allowed to use backup of type azure to store a snapshot. 

    - hosts: servers
      vars:
        elasticsearch_azure:
          client:
            default:
              account: vdc1backupelasticsearch
              key: secret_key
            primary:
              account: vdc1backupelasticsearch
              key: another_secret_key
              max_retries: 10
              timeout: 10s

      roles:
         - { role: ansible-role-elasticsearch-azure-backup }
         
The call below will create a backup of type azure. This allows later to save the snapshot in azure storage.

    - hosts: servers
      vars:
        elasticsearch_azure_backup_name: backup_name
        elasticsearch_azure_backup_config:
            type: azure
            settings:
                "client": "primary"
                "container": "backups"
                "base_path": "elas"
      roles:
         - { role: ansible-role-elasticsearch-azure-backup, action: create_backup }
    
The call below will create a snapshot on azure storage.

    - hosts: servers
      vars:
        elasticsearch_azure_backup_name: backup_name
        elasticsearch_azure_snapshot: "azure_snapshot"
        elasticsearch_azure_snapshot_config:
          indices: "myindex1, myindex2"
          ignore_unavailable: true
          include_global_state: false
          metadata:
            taken_by: "DevOps"
            taken_because: "backup before upgrading"
      roles:
         - { role: ansible-role-elasticsearch-azure-backup, action: take_snapshot }
         
The call below will delete a backup.

    - hosts: servers
      vars:
        elasticsearch_azure_backup_name: backup_name
      roles:
         - { role: ansible-role-elasticsearch-azure-backup, action: delete_backup }

License
-------

BSD

Author Information
------------------

Team DevOps.