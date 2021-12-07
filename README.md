# Ansible for FPP

This ansible project allows for Fleet Patching and Provisioning automation. 

## Playbook Execution

### Inventory

If running a playbook locally, edit the inventory file in this github repo. Add your FPP host and your Exadata hosts. Be sure to add in any required ssh args. Check the sample_inventory file for an example or refer to https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups for more information. 


## Ansible Codebase

This codebase contains a set of playbooks used to automate FPP operations. The playbooks reference ansible roles (an ansible file structure used to group reusable components), where the majority of work takes place. The roles > defaults > main.yml files for each role has default variables to set according to your environment. There are a few other places as well where changes should be made - search "TO EDIT" within the project.

### Roles

**grid_fpp.yml**, **rdbms_fpp.yml**

The grid_fpp and rdbms_fpp roles perform grid/rdbms specific fpp operations. The playbooks call tasks within these roles, and then these roles call helper functions in the same role and in the common roles.

**fpp_common.yml**, **target_common.yml**, **oci_common.yml**

The common roles run any tasks that must be performed on a specific host, such a shell scripts and file creations. If the role is being called, the assumption is that ansible is already operating on the correct host; any delegation happens in the actual playbooks, grid_fpp tasks, or rdbms_fpp tasks. The fpp_common runs on the fpp host, target_common on the exadata hosts, and oci_common on the localhost.


### Playbooks

General Variables - should always be included in a playbook run
- fpp_host (n3db1) - note: this must be the specific hostname, not a hostgroup encompassing the name in the inventory, as some playbooks will need the name.
- identity_file (/home/oracle/.ssh/idrsa.key) - path to the exadata ssh key from the fpp host

**rdbms_create_image.yml**
- Creates and registers a new rdbms image. It will only need to be run once per image so be sure the hostgroup specified (exa_host) only contains one cluster.
- Runtime Variables
    - exa_host (ecc1n1) - host where temporary home will be created and queried for create image operations 
    - exadata_type (exacc, exacs)
    - version (19.0.0.0) - base version
    - version_tag (19.13.0.0.0) - image tag for database home creation. Run cswlib showimages --product database to see available image tags. 
    - OPTIONAL: image_name (DB191300) - specify custom image_name if default name already registered 

**rdbms_create_wc_new.yml**
- Creates and adds a new RDBMS working copy. It will need to be run once per vm cluster so the hostgroup specified (exa_group) should contain one node per cluster.
- Runtime Variables
    - exa_group (exadata1) - hosts where fresh database homes will be created and registered as working copies 
    - image_name (DB191300) 
    - version (19.0.0.0)
    - version_tag (19.13.0.0.0)
    - OPTIONAL: db_home_name - specify custom db_home_name if default name already exists 
    - OPTIONAL: osdbagrp_groups

**rdbms_create_wc_existing.yml**
- Takes an existing database home and registers it to FPP. As with creating a new home, it will need to be run once per vm cluster so the hostgroup specified (exa_group) should contain one node per cluster. If running it on multiple clusters, remember that the database home to be registered should have the same name across the clusters. 
- Runtime Variables
    - exa_group (exadata1)
    - image_name (DB191200)
    - db_home_name (dbhome1_191200)
    - OPTIONAL: osdbagrp_groups

**rdbms_patch.yml**
- Moves a database to a new home. As databases should be patched individually, only one host should be specified. The actual database home names should be provided and the playbook will automatically parse for the corresponding working copy names.
- Runtime Variables 
    - exa_host (ecc1n1)
    - source_home (dbhome1_191200)
    - dest_home (dbhome1_191300)
    - db_unique_name (a4db0_iad3zx)
    - OPTIONAL: patch_error_param (-revert, -continue) - this parameter should be included on patch failures in order to choose to continue the process or revert. Note that an error qualifies as a patch failure only when it fails during the final patch step, any failures prior to that should be addressed separately
    - OPTIONAL: ignorewcpatches_param (-ignorewcpatches) - include when the patchedwc does not contain all the bug fixes of the sourcewc. Default behavior is to require all fixes in the new home.
    - OPTIONAL: forcerolling_param (-forcerolling) - include when the patchedwc contains a bug fix which is identified as 'non-rolling'. Default behavior is to require all fixes in the new home to be rolling, but under advisement of Oracle ExaCC support it is possible to force a rolling apply.

**rdbms_delete_home.yml**
- Deletes a database home from the vm cluster and from FPP. It will need to be run once per vm cluster so the hostgroup specified (exa_group) should contain one node per cluster. If running it on multiple clusters, remember that the database home to be deleted should have the same name across the clusters. 
- Runtime Variables
    - exa_group (exadata1)
    - exadata_type (exacc or exacs)
    - db_home_name (dbhome1_191200)
    - OPTIONAL: orphan_home - set orphan_home to true if home does not exist in FPP (i.e. if exists in the vm cluster but is not a registered working copy)

**gi_create_image.yml**
- Creates and registers a new grid image. Run once per image so be sure the hostgroup specified (exa_host) only contains one cluster. The host should have an active grid home that has already been patched to the correct version and is the image to be saved.
- Runtime Variables 
    - exa_host (ecc1n1)
    - container_url
        - OPTIONAL: curl_https_proxy
    - version (19.0.0.0)
    - version_tag (19.13.0.0.0)
    - OPTIONAL: image_name (GI191300) - specify custom image_name if default name already registered 

**gi_create_wc_new.yml**
- Creates and adds a new GI working copy. It will need to be run once per vm cluster so the hostgroup specified (exa_group) should contain one node per cluster.
- Runtime Variables
    - exa_group (exadata1)
    - container_url
        - OPTIONAL: curl_https_proxy
    - image_name (GI191300)
    - OPTIONAL: gi_home_suffix (grid191300_1) - path suffix to create custom gi home path instead of FPP generated path
    - OPTIONAL: wc_name - specify custom wc_name if default name already registered 

**gi_create_wc_existing.yml**
- Takes an existing grid home and registers it to FPP. Unlike other create WC operations, this will run on only one vm cluster as a time, as we are specifying an exact grid home path.  
- Runtime Variables
    - exa_host (ecc1n1)
    - image_name (GI191300) - to create new home and add wc
    - gi_home_path (/u02/app/19.0.0.0/grid1) - existing grid home to register as working copy. This home can be active or inactive.

**gi_patch.yml**
- Patches grid home in batches and will run on only one vm cluster as a time. Hostgroup exa_group should contain all hosts for the vm cluster in order to run patching prerequisties on the hosts included in the current batch.
- Runtime Variables
    - exa_cluster (ecc1_cluster)
    - source_wc (GI191300_ecc9)
    - dest_wc (GI191300_ecc9)
    - batch_list ((ecc1n1),(ecc1n3,ecc1n5),(ecc1n2,ecc1n4, ecc1n6)) - comma separated list of hosts to indicate batches for patching. First batch should always continue only one node. 
    - OPTIONAL: patch_error_param (-revert, -continue)
    - OPTIONAL: ignorewcpatches_param (-ignorewcpatches)
    - OPTIONAL: forcerolling_param (-forcerolling)

**gi_delete_home.yml**
- Deletes a grid home from the vm cluster and from FPP. This will run on only one vm cluster as a time, as we want to take care not to try to delete any grid homes that are still in use.
    - exa_host (ecc1n1)
    - wc_name (GI191300_ecc9)

**delete_image.yml**
- Deletes a rdbms or grid image from FPP.
- Runtime Variables
    - image_name (DB1911_210420)

## Additional Resources

OCI Collection for ansible: https://oci-ansible-collection.readthedocs.io/en/stable/collections/oracle/oci/index.html



