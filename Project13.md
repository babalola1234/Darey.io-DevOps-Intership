# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

### The use of ` include module ` in dynamic assignment, hence ` include = Dynamic ` in import module all statements are pre-processed at the time the playbooks are parsed, meaning during actual excution of the playbook if any statement changes, such statement will not be processed or considered. Hence its ` Static ` 

### in ` include module ` all statements are processed only during execution of the playbook. Meaning if they are changes to the statement during execution the new one will be used.

### It is recommended to use ` static assignments ` in playbooks becuase its more reliable and easy to debug.

# Using Dynamic Assignment in my existing structure.

* Step 1 I created a new branch ` called Dynamic-Assignment `

![image of dynamic-assignment on GitHub](./images/P13-image-0-dynamic-assignement-GitHub.PNG)

* I also created a new directory localy on ansibled server called ` dynami-assignment ` and created a new file in it called ` env-vars.yml `

![image of dynamic-assignment dir](./images/P13-image-0-dynamic-assignement.PNG)

![image of env-vars.yml and its contents](./images/P13-image-1a-env-vars-yml-file.PNG)

* I created another dir called ` env-vars ` to keep each environemnts variables files.

![image of env-var and roles dir](./images/P13-image-1-roles-and-envar-dir.PNG)

# Updating site.yml with dynamic assignemnets 

* Step 2 I updated site.yml with the statement below in other to make use use of dynamic assignemnet.

![image of updated site.yml](./images/P13-image-1b-updated-site-yml.PNG)

# Using Community Roles to create my MySql database

* Step 3 -- In this case I am  going to use an already created role by geerlingguy. it makes life so simple except that a lot of modifiying needs to be 
done to suite my needs as thr error occurs when runing the playbooks

* Before I downloaded MySQl role,I  Commited and Pushed to ` main branch ` my ` ansible-config-mgt ` 

` git init `

` git pull https://github.com/babalola1234/ansible-config-mgt.git `

` git remote add origin https://github.com/babalola1234/ansible-config-mgt git`


 * Step 4 I created roles inside ` ansible-config-mgt `  Downloaded 
 
 ` MySQL role developed by geerligguy ` and installed it using 
 
 ` ansible-galaxy install geerlingguy.mysql ` and rename it with the command below 

 ` mv geerlingguy.mysql/ mysql `

 ![image of renaming MySql](./images/P13-image-2-renaming-ansible-mysql.PNG)

* I made some configurations changes in ` main.yml ` in ` defaults dir ` under ` mysql dir `  according to the ` README.md ` file.  

![image for editing main.yml file](./images/P13-image-4a-editing-main-yml.PNG)

* Now I commited and pushed my new changes below.

![images of newly comited changes to roles-feature branch](./images/P13-image-3-git-push-upstream.PNG)

![image of merge in GitHub](./images/P13-image-6-merge-roles.PNG)

![image of merge in GitHub](./images/P13-image-6a-main-dynamic-merger.PNG)


# Creating and Configuring Load balancer Roles for both Nginx and Apache 

Step 1 I am going to use the comunity developed roles from geerliguy as well with some modifications in other to work to suite my environment .

![image from ansible-galaxy](./images/P13-image-7-LB-nginx-geerliguy.PNG)

![image from ansible-galaxy](./images/P13-image-7a-LB-apache-geerliguy.PNG)

` ansible-galaxy install geerlingguy.nginx `

` ansible-galaxy install geerlingguy.apache `

* Renamining both nginx and apache LBs

` mv geerlingguy.nginx/ nginx && mv geerlingguy.apache/ apache `

![image for installed nginx and apache](./images/P13-image-7c-installed-and-renamed-nginx-and-apache.PNG)


* Edited the main.yml/defaults file with the 2 uat-webservers  and https options, main.yml/tasks and main.yml/templates after reading the README.md file for nginx

![image for main.yml with uat-webservers](./images/P13-image-7d-mai-yml-nginx-uptream-uat-webservers.PNG)

* commited and pushed my nginx and apache config update

![image of commit and push](./images/P13-image-8-comitted-changes-after-nginx-apache-config.PNG)

* included some config in main/defaults and set-Redhat.yml

![image of main/defaults](./images/P13-image-8b-add-uat-webservers-ips-lb-username-main-default-apache.PNG)

![image of apache vhosts](./images/P13-image-8d-apache-vhost-main-defaults.PNG)

![image of main/set-redhat](./images/P13-image-8c-install-seboolean-task-set-redhat.PNG)


# Since I cant use both Ninx and Apache Load balancer, I need to add a condition to enable one. This can be achived by using variables.

* Declaring variables in ` defaults/main.yml` for both Nginx and Apache . 

* both variable named  ` enable_nginx_lb ` and ` enable_apache_lb ` 

* both values are set to ` false `

` enable_nginx_lb: false ` 

* another variable is declared  ` load_balancer_is_required ` and its set to ` false ` 

![image for nginx main.yml enable and loadbalancer](./images/P13-image-7d-mai-yml-nginx-uptream-uat-webservers.PNG)

` enable_apache_lb: false ` 

* another variable is declared for ` load_balancer_is_required ` 
and its set to ` false ` 

![image for apache main.yml enable and loadbalancer false](./images/P13-image-9b-apache-default-main-yml-LB-to-false.PNG)

* updated both assignement and site.yml resp.

![image of assignement and playbook-site-yml-update](./images/P13-image-11-plybook-site-yml-update.PNG)

* Runing ansible playbook to test each load uat-loadbalacer.

` ansible-playbook -i inventory/uat.yml playbooks/site.yml `

![images of ansible-playbooks](./images/P13-image-14a-ansible-script-succesfull-httpd.PNG)

![image of systemctl status httpd](./images/P13-image-14-httpd-started-and-runing-web2.PNG)

![image of systemctl status for nginx](./images/P13-image-14-nginx-started-and-runing-web2.PNG)












