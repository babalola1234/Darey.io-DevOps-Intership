### AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

- Let us Get Started


- You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a company (Choose an interesting name for it) that uses `WordPress CMS `for its main business website, and a `Tooling Website` (https://github.com/babalola1234/tooling) for their `DevOps team.` As part of the company’s desire for improved security and performance, a decision has been made to use a `reverse proxy technology` from NGINX to achieve this.

` A reverse proxy server is a type of proxy server that typically sits behind the firewall in a private network and directs client requests to the appropriate backend server. A reverse proxy provides an additional level of abstraction and control to ensure the smooth flow of network traffic between clients and servers`


- `Cost, Security, and Scalability ` are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![Image of AWS Multiple WebSite Project](./images/p15-multiple-wesites-projects.PNG)


## Starting Off AWS Cloud Project
=======================================

- There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit 

   * Create an AWS Master account. (Also known as Root Account)
   * Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
   * Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
   * Move the DevOps account into the Dev OU. And Login to the newly created AWS account using the new email address.

2. Create a domain name for your company 

3. Create a hosted zone in AWS, and map it to yourdomain 

* NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

-  Project: <ACS-BABA>
-  Environment: <dev>
-  Automated: <No> 


## SET UP A VIRTUAL PRIVATE NETWORK (VPC)
==========================================

* Always make reference to the architectural diagram above and ensure that your configuration is aligned with it.

1. Create a VPC -- ACS-BABA-VPC --10.0.0.0/24
2. Create an Internet Gateway
3. Create subnets as shown in the architecture 
4. Create a route table and associate it with public subnets 
5. Create a route table and associate it with private subnets
6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accecisble from the Internet)
7. Create 3 Elastic IPs
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

- Below are images for the above created compute resources

![Image of VPC](./images/p15-vpc-baba.PNG)
![image of internetgateway](./images/p15-internet-gateway.PNG)
![image of subnets](./images/p15-acs-baba-subnets.PNG)
![image of public and private subnets](./images/p15-acs-baba-rtb-pub-private.PNG)
![image of public and private subnets associations](./images/p15-acs-baba-rtb-sub-associations.PNG)
![image of igw associateion with public subnets](./images/p15-acs-igw-association-with-pub-sub.PNG)

![image of natgateway with elastic ip](./images/p15-acs-baba-natgatway-association-pub-1.PNG)

9. Create a Security Group for the below:

   * `Nginx Servers`: Access to `Nginx` should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder. -- `From anywhere through the internet gateway`

   * `Bastion Servers`: Access to the `Bastion servers` should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

   * `Application Load Balancer`: ALB will be available from the Internet

   * `Webservers`: Access to Webservers should only be allowed from the `Nginx servers`. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.

   * `Data Layer`: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

![Images of security groups above](./images/p15-acs-baba-sg.PNG)
   ---

## Set up and configure compute resources inside your VPC. 

1. Configure TLS Certificates From Amazon Certificate Manager (ACM)

* You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

1. Navigate to AWS ACM
2. Request a public wildcard certificate for the domain name you registered in AWS
3. Use DNS to validate the domain name
4. Tag the resource
![image of certs issued](./images/p15-acs-baba-certs-issued.PNG)

2. Setup EFS
   
* Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

   * Create an EFS filesystem
   * Create an EFS mount target per AZ in the VPC, associate it with both 
 subnets dedicated for data layer
   * Associate the Security groups created earlier for data layer.
   * Create an EFS access point. (Give it a name and leave all other settings as default)
![Image of EFS created](./images/p15-acs-baba-EFS.PNG)

3.  Setup RDS


* Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

` Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases.`

* To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution – this is a more advanced concept that will be discussed in following projects.

### To configure RDS, follow steps below:
---

1. Create a subnet group and add 2 private subnets (data Layer)
2. Create an RDS Instance for mysql 8.*.*
3. To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
4. Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
5. Configure VPC and security (ensure the database is not available from the Internet)
6. Configure backups and retention
7. Encrypt the database using the KMS key created earlier
8. Enable `CloudWatch` monitoring and export Error and Slow Query logs (for production, also include Audit)

![Image of RDS created ](./images/p15-acs-baba-RDS1.PNG)
![image of private subnets added to data layer](./images/p15-acs-baba-rds-subnet-group.PNG)

- Proceed With Compute Resources below.

1. EC2 Instances --for AMIs
2. Target Groups
3. Launch Templates
4. Application Load Balancers (ALB)
5. Autoscaling Groups

![Image of EC2 instances](./images/p15-acs-baba-ec2-instances.PNG)


- Provision the Compute Resources for `Nginx` to satify the requirement below

1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)

2. Ensure that it has the following software installed:
   * python
   * ntp
   * net-tools
   * vim
   * wget
   * telnet
   * epel-release
   * htop

3. Create an AMI out of the EC2 instance

### Prepare Launch Template For Nginx (One Per Subnet)
---
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install nginx

### Configure Target Groups
---

1. Select Instances as the target type
2. Ensure the protocol HTTPS on secure TLS port 443
3. Ensure that the health check path is /healthstatus
4. Register Nginx Instances as targets
5. Ensure that health check passes for the target group


### Configure Autoscaling For Nginx
---

1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the target group you created before
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications

![image of autoscaling](./images/p15-acs-baba-auto-scaling-group.PNG)

### Set Up Compute Resources for Bastion
---

### Provision the EC2 Instances for Bastion

1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
2. Ensure that it has the following software installed

   * python
   * ntp
   * net-tools
   * vim
   * wget
   * telnet
   * epel-release
   * htop

3. Associate an Elastic IP with each of the Bastion EC2 Instances
4. Create an AMI out of the EC2 instance

### Prepare Launch Template For Bastion (One per subnet)
---
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install Ansible and git

### Configure Target Groups


1. Select Instances as the target type
2. Ensure the protocol is TCP on port 22
3. Register Bastion Instances as targets
4. Ensure that health check passes for the target group

![image of target groups](./images/p15-acs-baba-target-group.PNG)
![image of tg tooling](./images/p15-acs-baba-tg-tooling.PNG)
![image of tg nginx](./images/p15-acs-baba-tg-nginx.PNG)
![image of tg wordpress](./images/p15-acs-baba-tg-wordpress.PNG)

### Configure Autoscaling For Bastion

1. Select the right launch template
2. Select the VPC
3. Select both public subnets
4. Enable Application Load Balancer for the AutoScalingGroup (ASG)
5. Select the target group you created before
6. Ensure that you have health checks for both EC2 and ALB
7. The desired capacity is 2
8. Minimum capacity is 2
9. Maximum capacity is 4
10. Set scale out if CPU utilization reaches 90%
11. Ensure there is an SNS topic to send scaling notifications

![image of autoscaling](./images/p15-acs-baba-auto-scaling-group.PNG)
## Set Up Compute Resources for Webservers
==========================================

Provision the EC2 Instances for Webservers
---
Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

1. Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

1. Ensure that it has the following software installed

  * python
  * ntp
  * net-tools
  * vim
  * wget
  * telnet
  * epel-release
  * htop
  * php

3. Create an AMI out of the EC2 instance

![Image of AMI for Nginx, Webservers and bastion](./images/p15-acs-baba-ami-nginx-webserver-bastion.PNG)

![image for launch templates -tooling, bastion, nginx and wordpress](./images/p15-acs-baba-launch-templates.PNG)

Prepare Launch Template For Webservers (One per subnet)
---
1. Make use of the AMI to set up a launch template
2. Ensure the Instances are launched into a public subnet
3. Assign appropriate security group
4. Configure Userdata to update yum package repository and install 
wordpress (Only required on the WordPress launch template)




## CONFIGURE APPLICATION LOAD BALANCER (ALB)
================================================

Application Load Balancer To Route Traffic To NGINX
---

`Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request`.

1. Create an Internet facing ALB
2. Ensure that it listens on HTTPS protocol (TCP port 443)
3. Ensure the ALB is created within the appropriate VPC | AZ | Subnets
4. Choose the Certificate from ACM
5. Select Security Group
6. Select Nginx Instances as the target group

![image of ALB internet facing](./images/p15-acs-baba-ALB-int-ext.PNG)

Application Load Balancer To Route Traffic To Web Servers
---

`Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.`

* To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

1. Create an Internal ALB
2. Ensure that it listens on HTTPS protocol (TCP port 443)
3. Ensure the ALB is created within the appropriate VPC | AZ | Subnets
4. Choose the Certificate from ACM
5. Select Security Group
6. Select webserver Instances as the target group
7. Ensure that health check passes for the target group

![image of ALB internet facing](./images/p15-acs-baba-ALB-int-ext.PNG)
### NOTE: This process must be repeated for both WordPress and Tooling websites.
---

## Configuring DNS with Route 53
=================================


Earlier in this project you registered a domain and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.

You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

Create other records such as CNAME, alias and A records.

NOTE: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read here to get to know more about the differences.

Create an alias record for the root domain and direct its traffic to the ALB DNS name.
Create an alias record for tooling.babadeen.link and direct its traffic to the ALB DNS name.

![image of wordpress sites](./images/p15-acs-baba-wordpress-site.1.PNG)
![image of wordpress sites](./images/p15-acs-baba-wordpress-site.1-valid-cert.PNG)
![image of wordpress sites](./images/p15-acs-baba-tooling-website.PNG)
