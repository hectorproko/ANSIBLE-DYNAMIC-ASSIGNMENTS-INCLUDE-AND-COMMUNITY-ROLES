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
      include_vars: "{{ item }}" 
      with_first_found: #implies that the first one found is used
        - files:
            - uat.yml
            - stage.yml
            - prod.yml
            - dev.yml
          paths:
            - "{{ playbook_dir }}/../env-vars" #playbook_dir determines the location of the running playbook
      tags:
        - always
```  
We are including the variables using a loop. `with_first_found` implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.  

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

**Updating site.yml with dynamic assignments**  

Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)


#### LOAD BALANCER ROLES

Installing 2 new roles [Nginx](https://galaxy.ansible.com/nginxinc/nginx) and [Apache](https://galaxy.ansible.com/geerlingguy/apache)  

``` bash
ansible-galaxy install nginxinc.nginx && ansible-galaxy install geerlingguy.apache
```
Moving the roles from default location to local repo (renaming it in the process) to be pushed

``` bash
hector@hector-Laptop:~/ansible-config-mgt/roles$ mv /home/hector/.ansible/roles/nginxinc.nginx ./nginx
hector@hector-Laptop:~/ansible-config-mgt/roles$ mv /home/hector/.ansible/roles/geerlingguy.apache ./apache
hector@hector-Laptop:~/ansible-config-mgt/roles$ ls
apache  common nginx  webservers
hector@hector-Laptop:~/ansible-config-mgt/roles$
```
For this particular test I'm going to edit **site.yml** `ansible-config-mgt/playbooks/site.yml`  

``` bash
- name: include the env-vars playbook
  import_playbook: ../dynamic-assignments/env-vars.yml 

- name: Loadbalancers assignment
  import_playbook: ../static-assignments/loadbalancers.yml
  when: load_balancer_is_required
```
Calling `env-vars.yml` will load variables into the playbook `site.yml`, in this case variable values from `ansible-config-mgt/env-vars/uat.yml` which for this example contains the following

``` bash
enable_nginx_lb: true
enable_apache_lb: false
load_balancer_is_required: true
```
Next `site.yml` imports `ansible-config-mgt/static-assignments/loadbalancers.yml`

``` bash
- hosts: lb
  roles: 
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
```

So based on the values of the variables in `uat.yml`, `loadbalancers.yml` calls either:  
`ansible-config-mgt/roles/apache/tasks/main.yml` or `ansible-config-mgt/roles/nginx/tasks/main.yml`