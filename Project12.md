### ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

* In this project I will continue working with ansible-config-mgt repository and will make some improvements of the existing  code.  Refactor an exiting Ansible code, create assignments, and use the imports functionality. Imports allow to effectively re-use previously created playbooks in a new playbook – it allows to organize  tasks and reuse them when needed.


### Step 1 – Jenkins job improvement

* Before I begin, I made some changes to the Jenkins job – by installing  Copy Artifact plugin.

* went to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and installed the plugin without restarting Jenkins

![Image of copy Artifact plugin](./images/P12-image-3a-copy-artifact-plugins.PNG)

*  created a new directory called ansible-config-artifact – This new directory will store  all artifacts after each build.

` sudo mkdir /home/ubuntu/ansible-config-artifact `

![image of ansible-config-artifact](./images/P12-image-0-ansible-config-artifact-dir.PNG)

* Changed permissions to the directory, so Jenkins could save files there – 

` chmod -R 0777 /home/ubuntu/ansible-config-artifact `

![image of ansible-config-artifact-permission](./images/P12-image-1-ansible-config-artifact-dir-0777.PNG)

* created a new freestayle job and named it save_artifacts, this project will be triggered when jenkins build and save each builds in it

![image of freestyle job save_artifacts](./images/P12-image-2a-save-artifiacts1.PNG)

![image of freestyle job save_artifacts](./images/P12-image-2-save-artifiacts1.PNG)

![image of freestyle job save_artifacts](./images/P12-image-2-save-artifiacts1.PNG2.PNG)

![image of freestyle job save_artifacts](./images/P12-image-2-save-artifiacts1.PNG2.PNG3.PNG)

* Ran a test build by making a code change in git hub

![image of freestyle job save_artifacts-test job](./images/P12-image-3-test-save-artifacts.PNG)

![image of freestyle job save_artifacts-test job](./images/P12-image-3-test-save-artifacts.PNG2.PNG)



### REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML
### Step 2 – Refactor Ansible code by importing other playbooks into site.yml

* Before starting to refactor the codes, I pulled down the latest code from master (main) branch, and created a new branch, name it refactor.

![image of new branch-refactor](./images/P12-image-3b-refactor-branch.PNG)


### However, breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

## Importing other playbooks.

* Within playbooks folder, I created a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that I created previously. 

![image of site.yml](./images/P12-image-4a-site-yml.png)

* I also created a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of my work. 

![image of static-assignement](./images/P12-image-4b-static-assignment.PNG)

* Moved common.yml1 file into the newly created static-assignments folder.

![image of common.yml inside static-assignment](./images/P12-image-4c-common-yml.png)

* Inside the site.yml file, imported common.yml1 playbook with the below codes.

![image of site.yml imported codes](./images/P12-image-4d-imported-codes.PNG)


* The code above uses built in import_playbook Ansible module.

* My folder structure should look like the below

![image of folder structure](./images/P12-image-4-after-mv-command.PNG)


* Ran ansible-playbook command against the dev environment, Since wireshark is already installed I created another playbook under static-assignments and name it common-del.yml. In this playbook, configured  deletion of wireshark utility.

![image of common-del-yml](./images/P12-image-5c-common-del-yml.PNG)

* updated site.yml with - import_playbook: ../static-assignments/common-del.yml  and ran it against dev servers:


![image of updated site.yml](./images/P12-image-4d-imported-codes.PNG)


* changed dir into ansible-config-mgt in order to run the ansible playbook 

* first I pinged all my servers to see if i can reach them.

![image of ansible-ping](./images/p12-image-5-ansible-ping-all-servers.PNG)

`  cd /home/ubuntu/ansible-config-mgt `

` ansible-playbook -i inventory/dev.yml /home/ubuntu/playbooks/site.yml `

![image of anbible playbook](./images/p12-image-5a-ansible-playbook-site-yml.png)

* verified  wireshark is deleted on all the servers by running wireshark --version and which wireshark command 

![image to confirm wireshark was removed](./images/p12-image-6-wireshark-removed-from-nginx-lb.PNG)

![image to confirm wireshark was removed](./images/p12-image-6a-wireshark-removed-from-web1-server.PNG)



### CONFIGURING UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’

### Step 3 – Configure UAT Webservers with a role ‘Webserver’

* I will spin up 2 new Web Servers as uat and use a dedicated role to make the configuration reusable.

* Launched 2 fresh EC2 instances using RHEL 8 image, and them named  ccordingly as – Web1-UAT and Web2-UAT.

![image of 2 new uat servers](./images/p12-image-7b-2-uat-webservers.PNG)

* To create a role, I must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.
There are two ways how you can create this folder structure:

* I can use an Ansible utility called ansible-galaxy inside ansible-config-mgt/roles directory or I need to create roles directory upfront

` mkdir roles `

` cd roles `

` ansible-galaxy init webserver `


The entire folder structure should look like below,

![imahe of role creation](./images/p12-image-7-role-creation-ansible-galaxy-after-rm-test-file-vars.PNG)

![imahe of role creation](./images/p12-image-7-role-creation-ansible-galaxy.PNGb4-rm-test-file-vars.PNG)

* I updated my inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of my 2 UAT Web servers

![image of 2 uat ip addresses](./images/p12-image-7a-uat-ips.PNG)

* I ensured I used ssh-agent to ssh into the Jenkins-Ansible instance just as I have done in project 11;


* In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.

![image of etc/ansible/ancible.cfg role path](./images/p12-image-7d-roles-modification.PNG)

 * changed dir to tasks directory, and within the main.yml file, pasted the below code.

![image of mail.yml](./images/p12-image-8a-main-yml-file.PNG)


* The above code Installed and configured Apache (httpd service)
Cloned Tooling website from GitHub https://github.com/babalola1234/tooling.git and the tooling website code is deployed to /var/www/html on each of 2 UAT Web server and Made sure httpd service is started


### REFERENCE WEBSERVER ROLE
### Step 4 – Reference ‘Webserver’ role

* Within the static-assignments folder, I created a new assignment for uat-webservers - uat-webservers.yml. This is where I will reference the role. the below codes were pasted 

![image of uat-weserver-roles](./images/p12-image-9a-uat-weserver-roles.PNG)

* I need to reference the entry point to the  ansible configuration in the site.yml file. Therefore, the below code is pasted inside site.yml.

* So, I have this in site.yml below

![image of updated site.yml file](./images/P12-image-4d-imported-codes.PNG)


### Step 5 – Commit & Test

* Commited all changes, created a Pull Request and merge them to my main branch,  webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to my Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

* ran the playbook against my uat inventory below

` ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml `

![image of ansible playbook out put](./images/p12-image-10-comitpand-test-uat-servers.PNG)

![image of ansible playbook out put2](./images/p12-image-10a-comitpand-test-uat-servers.PNG)

* below are the Jenkins job and artifacts 

![image of jenkins job](./images/p12-image-9-jenkins-job.PNG)

![image of ansible-artifacts](./images/p12-image-11-ansible-config-artifact.PNG)

` http://18.223.114.31/index.php `

![image of uat-web1 url](./images/p12-image-10b-uat-web1-index-php.PNG)

` http://18.222.17.118/index.php `


![image of uat-web2 url](./images/p12-image-10c-uat-web2-index-php.PNG)



