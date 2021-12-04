


TO DO
- TEST
    - multiple hosts in one group (where possible)
    - continue on a failed database patch
    - revert on a failed database patch
    - pre-gihome script
        - Note: set correct permissions / owner for action script
- Organization
    - should we be running these tasks from the same servers we are patching/doing working copies? I.e. when should the main playbook be running from "exahost" --> could it run from localhost??
- do more greps when ONLY parsing outputs of a shell command a single time
- check if variables are defined correctly / exist
- figure out a better way to select only the current batch / first node to run
- do proper clean-up on failures
- debug messages: one line where possible, var where needed, and not repetitive
- setting vars vs. passing vars
- Clean Up
    - task names / comments
    - README
    - set assert name + quiet

Full Test
- gi_create_image
    - Fail Case --> Same Image Name
- gi_create_wc
    - new (with and without path)
    - existing (curr and not curr home)
    - Fail Case --> Not existing Image
    - Fail Case --> oh_dirname already exists
- gi_patch
    - continue
    - revert
    - Fail Case --> Source not WC, Dest not WC
    - Fail Case --> incorrectly defined flags
- rdbms_create_image
- rdbms_create_wc
    - new
    - existing
- rdbms_patch
    - continue
    - revert
    - Fail Case --> Source not WC, Dest not WC
    - Fail Case --> DB Unique Name does not exist
    - Fail Case --> incorrectly defined flags
- delete_image
- rdbms_delete_wc
    - wc + home
    - orphan home
- gi_delete_wc

Finalize with Fiserv
- max_fail_percentage / failure behavior
- naming standards
    - wc_name using "cluster name" instead of client_name
- any_error_fatal?
    - gihome prereqs?
- Out of Scope
    - automating Post tasks --> not working with UA because won't move to next checkpoint until the end
    - Reporting / Orphan Homes and Working Copies





