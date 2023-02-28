# LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER


## Step 1 — Prepare a Web Server
` Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.` 

` Attach all three volumes one by one to your Web Server EC2 instance `

` Use lsblk to display block devices are attached to the server. 
 and  ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

 ` lsblk `
 ` ls /dev/ ` 

 ![image for lsblk](./images/Project-6-image-2-Attcahed-3-volumes-onwebserver.PNG)

Use df -h command to see all mounts and free space on your server

` df-h `

![image of df -h](./images/Project-6-image-2b-Attcahed-3-volumes-onwebserver%20df-h.PNG)

### Use gdisk utility to create a single partition on each of the 3 disks` 

` sudo gdisk /dev/xvdf ` 

` sudo gdisk /dev/xvdg `

` sudo gdisk /dev/xvdh `

![image for xvdf volumes](./images/Project-6-image-3-gdisk-xvdf.PNG)

![second image for xvdf](./images/Project-6-image-3b-gdisk-xvdf-2.PNG)

![image for xvdg](./images/Project-6-image-4-gdisk-xvdg.PNG)

![image for xvdh](./images/Project-6-image-5-gdisk-xvdh.PNG)

*  Using lsblk utility to view the newly configured partition on each of the 3 disks.

`lsblk `

![lsblk and df-h image ](./images/Project-6-image-6-lsblk-df-h-output.PNG)


* Install lvm2 package 

` sudo yum install lvm2. ` 

![lvm2 install image ](./images/Project-6-image-7-lvm2-installation.PNG)

 * check for available partitions. 

 ` sudo lvmdiskscan `

 ![image of lvmscan](./images/Project-6-image-8-lvmdiskscan.PNG)


* Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

` sudo pvcreate /dev/xvdf1 `


` sudo pvcreate /dev/xvdg1 `

` sudo pvcreate /dev/xvdh1 `

![images of pvcreate](./images/Project-6-image-9-pvcreate.PNG)

* Verify that your Physical volume has been created successfully by running 

` sudo pvs ` 

![image of pvs](./images/Project-6-image-9a-sudo-pvs.PNG)


*  Use vgcreate utility to add all 3 PVs to a volume group (VG) and name the VG webdata-vg 

` sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1 `

![images of vgcreate](./images/Project-6-image-10-vgcreate-vgs.PNG)


* Verify that your VG has been created successfully by running 

` sudo vgs `

![image of vgs](./images/Project-6-image-10a-sudo-vgs.PNG)

*  Use lvcreate utility to create 2 logical volumes, apps-lv and logs-lv. Use half of the PV size for apps-lv and the remaining space of the PV size for logs-lv.  Note: apps-lv will be used to store data for the Website while logs-lv will be used to store data for logs

` sudo lvcreate -n apps-lv -L 14G webdata-vg `

` sudo lvcreate -n logs-lv -L 14G webdata-vg ` 

![images for lvcreate for both apps-lv and logs-lv](./images/Project-6-image-11-lvcreate-sudo-lvs.PNG)

![images after lvcreate for lsblk](./images/Project-6-image-13-sudo-lsblk-after-lvcreate.PNG)


## Verify that your Logical Volume has been created successfully by running 

` sudo lvs `

![image of lvs](./images/Project-6-image-11a-sudo-lvs.PNG)

### Verify the entire setup for  - VG, PV, and LV

` sudo vgdisplay -v  `
` sudo lsblk ` 

![images of the VG, PV, LV and lsblk](./images/Project-6-image-12-sudo-vgdisplay-v.PNG)

### Use mkfs.ext4 to format the logical volumes with ext4 filesystem

` sudo mkfs -t ext4 /dev/webdata-vg/apps-lv `

` sudo mkfs -t ext4 /dev/webdata-vg/logs-lv `

![images of mkfs ](./images/Project-6-image-14-mkfs-ext4-apps-lv-logs-lv.PNG)

* Create /var/www/html directory to store website files and  Create /home/recovery/ logs directory to store backup of log data

` sudo mkdir -p /var/www/html ` 

` sudo mkdir -p /home/recovery/logs `

![images for both html and logs](./images/Project-6-image-15-mkdir-var-www-html-and-home-recovery-logs.PNG)

* Mount /var/www/html on apps-lv logical volume

` sudo mount /dev/webdata-vg/apps-lv /var/www/html/ ` 

![images of mount /dev/webdata-vg/apps-lv /var/www/html](./images/Project-6-image-16a-mount-apps-lv-to-html.PNG)

* Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

` sudo rsync -av /var/log/. /home/recovery/logs/ ` 

![images of rsync /var/log/. /home/recovery/logs](./images/Project-6-image-16b-rsync-var-log-to-recovery-logs.PNG)

### Mount /var/log on logs-lv logical volume. Note that all the existing data on /var/log will be deleted. That is why backed on /home/recovery/logs)

` sudo mount /dev/webdata-vg/logs-lv /var/log `

![images of mount /dev/webdata-vg/logs-lv /var/log](./images/Project-6-image-16-mount-dev-webdata-vg-logs-lv-var-log.PNG)

### Restore log files back into /var/log directory

` sudo rsync -av /home/recovery/logs/. /var/log ` 

![images of rsync recovery/logs/. /var/log](./images/Project-6-image-17-resync-recovery-log-to-var-logs.PNG)

## Update /etc/fstab file so that the mount configuration will persist after restart of the server. 

` sudo blkid `

![images of blkid](./images/Project-6-image-18-blkid-output.PNG)


### UPDATE THE `/ETC/FSTAB` FILE The UUID of the device will be used to update the /etc/fstab file;

` sudo vi /etc/fstab `

![images of the updated /etc/fstab](./images/Project-6-image-19-etc-fstab-outout.PNG)



* Test the configuration and reload the daemon

 ` sudo mount -a `

 ` sudo systemctl daemon-reload `

 ![images of mount-a and systemctl reload](./images/Project-6-image-20-mount-a-and-systemctl-reload.PNG)


### Verify your setup by running df -h, output must look like this:


![image for the df -h ](./images/Project-6-image-20a-df-h-output.PNG)



# Step 2 — Prepare the Database Server
### Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

### second EC2 created with Redhat OS a nd 3 new vilemes attached

 ` lsblk `

 ![image for the 3 new volumes](./images/Project-6-image-21-3new-volumes-db-server.PNG)

 ` ls /dev/ ` 

 ![image for lsblk and ls /dev for new volumes](./images/Project-6-image-21a-3new-volumem-ls-dev-output.PNG)


Use df -h command to see all mounts and free space on your server

 ` df-h `

![image of df -h](./images/Project-6-image-22-lsblk-2n-EC2.PNG)

### Using gdisk utility to create a single partition on each of the 3 disks` 

` sudo gdisk /dev/xvdf ` 

` sudo gdisk /dev/xvdg `

` sudo gdisk /dev/xvdh `

![image for xvdf volumes](./images/Project-6-image-23a-gdisk-xvdf.PNG)

![image for xvdg volumes](./images/Project-6-image-23b-gdisk-xvdg.PNG)

![image for xvdh volumes](./images/Project-6-image-23c-gdisk-xvdh.PNG)

### Use lsblk utility to view the newly configured partition on each of the 3 disks.` 

`lsblk `

![lsblk and df-h image 3 disks](./images/Project-6-image-24-lsblk-after-gdisk-for-3-new-volumes.PNG)

### Install lvm2 package 

` sudo yum install lvm2. ` 

![lvm2 install image](./images/Project-6-image-25-lvm-installation.PNG)

### checking  for available partitions. 

 ` sudo lvmdiskscan `

 ![image of lvmdiskscan](./images/Project-6-image-26-lvmscan.PNG)

 ### Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

` sudo pvcreate /dev/xvdf1 `

` sudo pvcreate /dev/xvdg1 `

` sudo pvcreate /dev/xvdh1 `

![images of pvcreate for the 3 volumes](./images/Project-6-image-27-pvcreate.PNG)

Verify that your Physical volume has been created successfully by running 

` sudo pvs ` 

![image of pvs](./images/Project-6-image-27a-sudo-pvs.PNG)

### Use vgcreate utility to add all 3 PVs to a volume group (VG) and name the VG dbdata-vg 

` sudo vgcreate dbdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1 `

![images of vgcreate and vgs output](./images/Project-6-image-28-vgcreate-sudo-vgs.PNG)

### Use lvcreate utility to create 2 logical volumes, db-lv and dblogs-lv. Use half of the PV size for apps-lv and the remaining space of the PV size for logs-lv.  Note: apps-lv will be used to store data for the Website while logs-lv will be used to store data for logs

` sudo lvcreate -n db-lv -L 14G dbdata-vg `

` sudo lvcreate -n dblogs-lv -L 14G dbdata-vg ` 

![images for lvcreate for both db-lv and dblogs-lv](./images/Project-6-image-29-lvcreate.PNG)



## Verify that your Logical Volume has been created successfully by running 

` sudo lvs `

![image of lvs](./images/Project-6-image-29a-sudo-lvs.PNG)


### Verify the entire setup for  - VG, PV, and LV

` sudo vgdisplay -v  `

` sudo lsblk ` 

![images of the VG, PV, LV ](./images/Project-6-image-31-vgdisplay-v.PNG)

![images after for lsblk](./images/Project-6-image-30-sudo-lsblk.PNG)


### Use mkfs.ext4 to format the logical volumes with ext4 filesystem

` sudo mkfs -t ext4 /dev/dbdata-vg/db-lv `

` sudo mkfs -t ext4 /dev/dbdata-vg/dblogs-lv `

![images of mkfs ](./images/Project-6-image-32-mkfs-ext4.PNG)

### Create /db directory to store db files and  Create /home/recovery/dblogs directory to store backup of dblog data

` sudo mkdir /db ` 
` sudo mkdir -p /home/recovery/dblogs `

![images for /db and dblogs dir](./images/Project-6-image-33-mkdir-db-dblogs.PNG)

### Mount /db on db-lv logical volume

` sudo mount /dev/dbdata-vg/db-lv /db` 

![images of mount /dev/dbdata-vg/db-lv /db](./images/Project-6-image-34-sudo-mount-db-lv-db.PNG)


### Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

` sudo rsync -av /var/log/. /home/recovery/dblogs/ ` 

![images of rsync /var/log/. /home/recovery/dblogs](./images/Project-6-image-35-var-log-rcovr-dblogs.PNG)

### Mount /var/log on dblogs-lv logical volume. Note that all the existing data on /var/log will be deleted. That is why backed on /home/recovery/dblogs)

` sudo mount /dev/dbdata-vg/dblogs-lv /var/log `

![images of mount /dev/dbdata-vg/dblogs-lv /var/log](./images/Project-6-image-36-dblogs-lv-var-logs.PNG)

### Restore log files back into /var/log directory

` sudo rsync -av /home/recovery/dblogs/. /var/log ` 

![images of rsync recovery/dblogs/. /var/log](./images/Project-6-image-37-recov-dblogs-var-log.PNG)

## Update /etc/fstab file so that the mount configuration will persist after restart of the server. 

` sudo blkid `

![images of blkid](./images/Project-6-image-38a-blkid.PNG)


### UPDATE THE `/ETC/FSTAB` FILE The UUID of the device will be used to update the /etc/fstab file;

` sudo vi /etc/fstab `

![images of the updated /etc/fstab](./images/Project-6-image-38b-editing-etc-fstab.PNG)


### Test the configuration and reload the daemon

 ` sudo mount -a `
 ` sudo systemctl daemon-reload `

 ![images of mount-a and systemctl reload](./images/Project-6-image-38c-systemctl-reload.PNG)


### Verify your setup by running df -h, output must look like this:


![image for the df -h ](./images/Project-6-image-38d-df-h.PNG)


# Step 3 — Install WordPress on your Web Server EC2
### Update the repository

` sudo yum -y update `

![image of yum update](./images/Project-6-image-web1-update.PNG)

### install wget, Apache and it’s dependencies

` sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json `

![image of installing  wget httpd php php-mysqlnd php-fpm php-json](./images/Project-6-image-web2-wget-httpd-phpetc.PNG)

### Start Apache

` sudo systemctl enable httpd `

` sudo systemctl start httpd `

` sudo systemctl status httpd `

![image of enable and start apache httpd and status](./images/Project-6-image-web3-httpd%20status.PNG)



### To install PHP and it’s depemdencies


` sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm `

![repo image installation for php](./images/Project-6-image-web4-repo-install.PNG)

` sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm `

![image for yum utills install](./images/Project-6-image-web5-yum-utils.PNG)


` sudo yum module list php` 

![image module list](./images/Project-6-image-web6-module-list.PNG)

` sudo yum module reset php `

![yum module reset](./images/Project-6-image-web7-module-reset.PNG)

` sudo yum module enable php:remi-7.4 `

![yum module enable php:remi-7.4](./images/Project-6-image-web8-enable-7-4.PNG)

` sudo yum install php php-opcache php-gd php-curl php-mysqlnd `

![php ad its dependencies image](./images/Project-6-image-web9-install-php.PNG)

` sudo systemctl start php-fpm `

` sudo systemctl enable php-fpm `

![image for start,enable and status php](./images/Project-6-image-web10-start-enable-php.PNG)


` setsebool -P httpd_execmem 1 `

![image for setting selinux bool](./images/Project-6-image-web10a-setsebool.PNG)

### Restart Apache

` sudo systemctl restart httpd `

![restarting https httpd](./images/Project-6-image-web18-restart-httpd.PNG)

![restarting https httpd](./images/Project-6-image-web18a-restart-apache.PNG)

### Download wordpress and copy wordpress to var/www/html

  ` mkdir wordpress && cd wordpress `

  ![image for mkdir and cd wordpress](./images/Project-6-image-web20-mkdir%20wordpress.PNG)
  
  ` sudo wget http://wordpress.org/latest.tar.gz `

  ![image for wget wordpress](./images/Project-6-image-web10-wget-wordpress.PNG)

  ` sudo tar xzvf latest.tar.gz `

  ![image tar wordpress](./images/Project-6-image-web11-tar.PNG)

  ` sudo rm -rf latest.tar.gz ` 

  ![image sudo rm latest.tar](./images/Project-6-image-web19-sudo-rm-latest-tar.PNG)


  ` sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php `

  ` sudo cp -R wordpress /var/www/html/ `

  ![image cp wordpress](./images/Project-6-image-web12-rm-sudo-cp.PNG)

### Configure SELinux Policies

 ` sudo chown -R apache:apache /var/www/html/wordpress `

 ` sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R `

  ` sudo setsebool -P httpd_can_network_connect=1 `

  ![image chown,chcon and setsebool](./images/Project-6-image-web13-sudo-chown-chcon-setbool.PNG)



  # Step 4 — Install MySQL on your DB Server EC2
` sudo yum update `

` sudo yum install mysql-server `


![image for mysql-server installation](./images/Project-6-image-43-mysql-server.PNG)


### Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:

` sudo systemctl restart mysqld `

` sudo systemctl enable mysqld `

![image for restart and enable-mysqld](./images/Project-6-image-44-restart-enable-mysql.PNG)


# Step 5 — Configure DB to work with WordPress

` sudo mysql `

CREATE DATABASE wordpress;

CREATE USER `ec2-user`@`172.31.17.251` IDENTIFIED BY 'mypass';

GRANT ALL ON wordpress.* TO 'ec2-user'@'172.31.17.251';

FLUSH PRIVILEGES;

SHOW DATABASES;

exit

![mysql image ](./images/Project-6-image-45-db-to-wordpress-config.PNG)


# Step 6 — Configure WordPress to connect to remote database.Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32


# Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client
` sudo yum install mysql `

![image yum install mysql](./images/Project-6-image-web14-mysql-installation-on-webserver.PNG)

` sudo mysql -u ec2-user -p -h 172.31.17.52 `

![image of test remote connection to db](./images/Project-6-image-web15-ec2-user-to-db-server.PNG)


### Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

### Change permissions and configuration so Apache could use WordPress:

### Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

Try to access from your browser the link to your WordPress http://18.220.230.96/wordpress/

![image of wordpress landing page](./images/Project-6-image-web17-wordpress-landing-page.PNG)

![image of wordoress page](./images/Project-6-image-web17a-wordpress-login-page.PNG)

![image of welcome to wordpress](./images/Project-6-image-web17b-welcome-to-wordpress.PNG)