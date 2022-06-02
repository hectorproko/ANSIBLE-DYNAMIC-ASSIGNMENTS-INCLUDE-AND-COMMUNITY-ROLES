## ANSIBLE-DYNAMIC-ASSIGNMENTS-INCLUDE-AND-COMMUNITY-ROLES
PROJECT 13

#### ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES
#### INTRODUCING DYNAMIC ASSIGNMENT INTO OUR STRUCTURE
In repo [ansible-config-mgt](https://github.com/hectorproko/ansible-config-mgt) I create a new branch ``dynamic-assignments``  

In this new branch I create a folder ``dynamic-assignments`` and inside it a file ``env-vars.yml``

``` bash 
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}" #include is deprecated now we use variants include_role, include_tasks, separating features of module
      with_first_found: #implies that the first one found is used
        - files:
            - uat.yml #this will only load the first file in the loop
            - stage.yml
            - prod.yml
            - dev.yml
          paths:
            - "{{ playbook_dir }}/../env-vars" #playbook_dir determines the location of the running playbook
      tags:
        - always
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
