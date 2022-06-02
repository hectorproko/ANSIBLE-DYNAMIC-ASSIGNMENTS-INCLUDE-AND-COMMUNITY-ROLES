## ANSIBLE-DYNAMIC-ASSIGNMENTS-INCLUDE-AND-COMMUNITY-ROLES
PROJECT 13

#### ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES
#### INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE
In repo [ansible-config-mgt](https://github.com/hectorproko/ansible-config-mgt) I create a new branch ``dynamic-assignments``  

In this new branch I create a folder ``dynamic-assignments`` and inside it a file ``env-vars.yml``

``` bash 
───────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
       │ File: dynamic-assignments/env-vars.yml
───────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ ---
   2   │ - name: collate variables from env specific file, if it exists
   3   │   hosts: all
   4   │   tasks:
   5   │     - name: looping through list of available files
   6   │       include_vars: "{{ item }}" #include is deprecated now we use variants include_role, include_tasks, separating features of module
   7   │       with_first_found: #implies that the first one found is used
   8   │         - files:
   9   │             - uat.yml #this will only load the first file in the loop
  10   │             - stage.yml
  11   │             - prod.yml
  12   │             - dev.yml
  13   │           paths:
  14   │             - "{{ playbook_dir }}/../env-vars" #playbook_dir determines the location of the running playbook
  15   │       tags:
  16   │         - always
───────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

``` bash
├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
```
#### UPDATE SITE.YML WITH DYNAMIC ASSIGNMENTS
#### LOAD BALANCER ROLES
