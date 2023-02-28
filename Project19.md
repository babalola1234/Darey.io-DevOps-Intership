## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM CLOUD -- PART 4.

- Terraform Cloud executes Terraform commands on disposable virtual machines, this remote execution is also called `remote operations`.
 
- What Terraform Cloud is and why use it -- Go to readme section for details 

### Project  Review -- `Migrate your .tf codes to Terraform Cloud `

- We will be excuting terraform codes in Project 18 from `terrraform cloud console`

# Step 1: Create a Terraform Cloud account
- Set up a Terraform account and verify your email
- Create an organization, Select `Start from scratch`, choose a name for your organization ` babadeen-link`  and create it.

![image of terraform organization](./Images/p19-terraform-organization.PNG)


- Configure your workspace 

![image of terraform workspace](./Images/p19-terraform-cloud-workspace.PNG)

- Select `version control workflow` option, as the most common and recommended way to run Terraform commands triggered from our git repository
- Connect to your repository in my case its Gitlab, `babalola deen/terraform-aws-project-19`


- Configure variables - Terraform Cloud supports `two types of variables`: `environment variables and Terraform variables.` Either type can be marked as sensitive, which prevents them from being displayed in the Terraform Cloud web UI and makes them write-only.
    - in my case I have `*.auto.tfvars` created in my reporository `terraform-aws-project-19` which terraform will upload automatically from my repository.

    - next setup two up `environmental variables` for both the `AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY`. Set the values that you used in Project 16, these credentials will be used to privision your AWS infrastructure by Terraform Cloud.

- After you have set these `two environment variables `– your Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.


# Step 2: Building AMI with Packer

- The `AMI's` are built with `Packer` -- ` HashiCorp's open-source tool for creating machine images from source configuration` -- go to readme section for more details 

    - Install Packer and make sure its configured and runing fine
     
     ![image of packer path environmental setting in Windows](./Images/p19-packer-environment-setting.PNG)
     ![image of packer path environmental setting in Windows](./Images/p19-packer-path.PNG)
    ![image of packer installed --Windows Version](./Images/p19-packer-installed.PNG)

### Runing Packer to create all the AMI's

- Cloned the Packer repo for the AMI

![image for Packer repo](./Images/p19-packer-repo.PNG)

- Ran packer command to build Bastion, Nginx, Webservers

- Below are the screen shots for all the servers ami

![image for bastion](./Images/p19-bastion-ami.PNG)

![image for nginx](./Images/p19-nginx-ami.PNG)

![image for Webservers--tooling and wordpress](./Images/p19-web-ami.PNG)

![image for ubuntu](./Images/p19-ubuntu-ami.PNG)

![image of all ami's in aws](./Images/p19-ami--all.PNG)


# Step 3 Using the Terraform cloud console to provision the aws infrastructure

- replacing the packer built ami in `terraform.auto.tfvars`

![image of new packer ami's in terraform.auto.tfvars](./Images/p19-ami-updates--auto-tfvars.PNG)

### `Run terraform plan` and `terraform apply` from web console

- Switch to "Runs" tab and click on `"Queue plan manualy"` button. If planning has been successfull, you can proceed and `confirm Apply`– press "Confirm and apply", provide a comment and "Confirm plan"

![image of terraform plan](./Images/p19-terra-cloud-plan2.PNG)

![image of terraform plan](./Images/p19-terra-cloud-plan1.PNG)

![image of terraform applied completed](./Images/p19-terra-cloud-apply-finished.PNG)


- Check the logs and verify that everything has run correctly. Note that Terraform Cloud has generated a unique state version that you can open and see the codes applied and the changes made since the last run.

### Testing automated terraform plan

- By now, you have tried to launch plan and apply manually from Terraform Cloud web console. But since we have an integration with Gitlab, the process can be triggered automatically. 

- Try to change something in any of `.tf files` and look at "Runs" tab again – plan must be launched automatically, but to apply you still need to approve manually. Since provisioning of new Cloud resources might incur significant costs. Even though you can configure `"Auto apply"`, it is always a good idea to verify your plan results before pushing it to apply to avoid any misconfigurations that can cause ‘bill shock’.

# Step 4 Configuring the infrastructure with Ansible

- Ansible: Using Ansible scripts to configure the infrastucture

- configure your ssh agent ` eval ssh-agent -s ` in other to connect to your bastion to complete your infrastructure configuration

### Updating the nginx.confi.j2 file with the internal load balancer DNS name generated  from terraform cloud apply

![image of connection to bastion](./Images/p19-nginx-status.PNG)

- Updating the `nginx.conf.j2` file to input the internal load balancer dns name generated via terraform:

![image of nginx config](./Images/p19-nginx-config.PNG)

- Verifying the inventory

![image of inventory](./Images/p19-ansible-graph.PNG)

- running the ansible-playbook command 

![image of ansible complete command](./Images/p19-ansible-complete.PNG)



# Practice Task №1
- Configure 3 branches in your terraform-cloud repository for ` dev, test, prod ` environments

![git-test-dev-prod](./Images/p19-gitlab-test-dev-prod.PNG)
- Make necessary configuration to trigger runs automatically only for dev environment

![git-test-dev-prod](./Images/p19-git-test-dev-prod.PNG)

- Create an Email and Slack notifications for certain events (e.g. started plan or errored run) and test it


![made changes in Dev](./Images/p19-gitlab-dev-merge.PNG)


![Terraform cloud planning after changes](./Images/p19-terra-cloud-plan-for-dev.PNG)


![image of slack Activation](./Images/p19-slack-activation.PNG)

![image of slack notification](./Images/p19-slack-notification-for-dev.PNG)

![image of slack notification-run apply](./Images/p19-run-apply-slack-notification.PNG)


- Apply destroy from Terraform Cloud web console

![image of cloud terra destroy](./Images/p19-terra-destroy.PNG)


# Practice Task №2 Working with Private repository

- Create a simple Terraform repository -- Forked the repo from -- https://github.com/hashicorp/learn-private-module-aws-s3-webapp

- Then under my repository's tab, clicking on tag to create tag, clicking 'Create a new release' and adding 1.0.0 to the tag version field setting the release title to "First module release

![Module-tag-and-release](./Images/p19-module-tag-and-release.PNG)

![Private module](./Images/p19-private-registry-s3-webapp.PNG)

- Clicking Publish release to create the release
- To create a Terraform module for private module registry in the terraform registry navigate to the Registry header in Terraform Cloud and select Publish private module from the upper right corner.

- Select the GitHub VCS provider and choose the name of the module repository ` terraform-aws-private-s3-webapp-p19`  and clicking the Publish module button.

- Import the module into your private registry

![image of private registry connected to private module](./Images/p19-private-registry-activated-completed.PNG)

- Create a configuration that uses the module,forking the repo ` https://github.com/babalola1234/private-module-root-p19.git` which will access the module published and Terraform will use it to create the infrastructure.

![image of new configuration created for root module-main.tf](./Images/p19-task2-main-tf.PNG)

![image of new configuration created for root module-variable.tf](./Images/p19-task2-variable-tf.PNG)

![image of new configuration created for root module-output.tf](./Images/p19-task2-output-tf.PNG)


- Create a workspace for the configuration

![New workspace created -private-module-root-p19 ](./Images/p19-task2-new-workspace-created.PNG)

- Deploy the infrastructure

![Image of planned terraform-cloud from private registry](./Images/p19-task2-plan-root.PNG)

![Image of applied terraform-cloud-from private registry](./Images/p19-task2-plan-applied-successfuly.PNG)

![Image of deployed infra](./Images/p19-task2-deployed-infra.PNG)

![Image of Deployed infra2](./Images/p19-task2-deployed-infra2.PNG)


- Destroy your deployment

![image of destroyed infra](./Images/p19-task2-destroy-plan2.PNG)



