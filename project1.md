# Documentation for project 1 - WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS

` Step 1 - installing apcahe and updating firewall`

` updating the list of package manager`

`sudo apt update`

` installing apache2 package`

`sudo apt install apache2 -y `

` verifying apache2 is installed`

`sudo systemctl apache2`

![project image](./images/Project1-Image1-systemctl-status-apache2.PNG)

` http://my-ip:80`

![project image](./images/Project1-Image2-apache-default-page.PNG)

`  Step 2 - installing mysql`

` sudo apt install mysql-server -y `

` loging into my sql database`

` sudo mysql `

![project image](./images/Project1-Image3-mysql-passoword.PNG)



` # setting mysql root passoword within mysql databae before securing it`

` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'; `

` # starting interactive script `

` sudo mysql_secure_installation `

` # logging into mysql after setting root passowrd `

` sudo mysql -p `

*  Step 3 - installing PHP,libapache2-mod-php and  php-mysql 

` sudo apt install php libapache2-mod-php php-mysql -y `

*  to check the php version `

` php -v ` 

![project image](./images/project-1-image-4-php-version.PNG)

## Step 4 - Virtual host for apache website `

*  creating projectlamp dir 

`sudo mkdir /var/www/projectlamp`

* assigning owrnership to current user `

` sudo chown -R $USER:$USER /var/www/projectlamp `

*  creating a new config file called project.conf` 

![project image](./images/project-1-image-5-projectlamp-conf.PNG)

*  enabling the virtual host `

` sudo a2ensite projectlamp `

*  disable the default host `

` sudo a2dissite 000-default `

* checking for syntax error`

![project image](./images/project-1-image-6-ena-disab-configtest.PNG)

* restarting apache2`

` sudo systemctl reload apache2 `

*  creating an indel file to test the web server `

` sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html` 


![project image](./images/project-1-image-7-new-website-test-page.PNG)


### step 5 -- enabling php on the website `

` sudo vim /etc/apache2/mods-enabled/dir.conf `

![project image](./images/project-1-image-8-mods-enable-dir-conf.PNG)


*  restarting apache2 to load the new config `

` sudo systemctl reload apache2 `

* creating a new test script for index.php `

`vim /var/www/projectlamp/index.php with this script below `

` <?php
phpinfo(); `

![project image](./images/project-1-image-9-new-php-test-page.PNG)


` removing the php file `

` sudo rm /var/www/projectlamp/index.php `


* [Project-1-completion-review-link](https://github.com/babalola1234/project-1.git)














