
 # Project 5 - Create and configure two Linux-based virtual servers (EC2 instances in AWS). 
## Server A name - `mysql server`
## Server B name - `mysql client`


### Step 1  On mysql client Linux Server I installed MySQL Client software. 

` sudo apt update `

![sudo apt update mysql-client](./images/Project-5-image-11-sudo-apt-update-client-sql.PNG)

` sudo apt-get install mysql-client -y `

![installing mysql client image](./images/Project-5-image-12-sudo-mysql-client-install.PNG)

## Step 2 On mysql server Linux Server I install MySQL server software. 

` sudo apt update `

![sudo apt update mysql-server ](./images/Project-5-image-9-sudo-apt-update.PNG)

` sudo apt install mysql-server -y `

![installing mysql server image](./images/Project-5-image-10-sudo-apt-install-mysql-server.PNG)


* By default, both EC2 virtual servers are located in the same local virtual network so this will allow comunication btween both machines `

## Using mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, I opened port 3306 on mysql server `

* For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’. 

![port 3306 opened on mysql server for only mysql client ip image](./images/Project-5-image-1-port-3306.PNG)

## Step 3  You might need to configure MySQL server to allow connections from remote hosts. allowing access from mysql client

` sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf  >> Replaced ‘127.0.0.1’ to ‘0.0.0.0’ `

![remote access image](./images/Project-5-image-3-remote-access-comand.PNG)
![allowed remote connection on mysql server image](./images/Project-5-image-2-remote-access.PNG)

 

* By default, mysql username and password you are using is allowed to access mysql-server locally. 

## Step 4 I created a separate user for the purpose of the remote connection called babadeen by runing the below command.

` CREATE USER 'babadeen'@'172.31.31.27' IDENTIFIED BY 'abcd1234'; `

![ new user created on mysql sever image](./images/Project-5-image-8-CREATE-NEW-USER-ON-MYSQL-SERVER.PNG)


* Run a command like below to access from all machines and all databases.

GRANT ALL PRIVILEGES ON database.* TO 'babadeen'@'172.31.31.27'; OR
GRANT ALL PRIVILEGES ON *.* TO 'babadeen'@'%';

* Run a command like below to give access from specific IP

` GRANT ALL PRIVILEGES ON *.* TO 'babadeen'@'172.31.31.27; `

![grant command image](./images/Project-5-image-7-grant-and-flush-priviledge.PNG)

* Finally, you may also need to run
 
 ` FLUSH PRIVILEGES;`

 ![flush prviledges image](./images/Project-5-image-7-grant-and-flush-priviledge.PNG)
 
* Restart mysql server. ` 

` sudo /etc/init.d/mysql restart `

![reatarting my mqlserver to effect changes](./images/Project-5-image-6-restarting-mysql-server.PNG)

* Testing remote Connection

` mysql -h 172.31.16.6 -u babadeen -pabcd1234 `

![mysql remote connetion image](./images/Project-5-image-4-remote-connection-from-mysql-client.PNG)


*  If you get a mysql shell, don’t forget to run show databases; to check if you have right privileges from remote machines.


![SHOW DATABSES; output when connected](./images/Project-5-image-5-showdatabase-output.PNG)



