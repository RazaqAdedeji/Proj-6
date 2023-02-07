# Project 6 Documentation

- Step 1 - Prepare a web server

![E2C instance](./images/2%20E2c%20instance.png)

- Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB

![3 volumes attached](./images/3%20volume%20created.png)

'lsblk'

![lsblk command](./images/lsblk%20first.png)

'df -h'

![all mount & free spacee](./images/df%20-h.png)

'!sudo gdisk /dev/xvdf'

![gdisk xvdf](./images/sudo%20gdisk%20%3Adev%3Axvdf.png)

sudo gdisk /dev/xvdg

![gdisk xvdg](./images/sudo%20gdisk%20%3Adev%3Axvdg.png)

sudo gdisk /dev/xvdg

![gdisk xvdh](./images/sudo%20gdisk%20%3Adev%3Axvdh.png)

'lsblk'

![lsblk utuility](./images/lsblk%20web%20server.png)

- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.

'sudo pvcreate /dev/xvdf1'

'sudo pvcreate /dev/xvdg1'

'sudo pvcreate /dev/xvdh1'

'sudo pvs'

![sudo pvs](./images/sudo%20pvs.png)

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

'sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1'

- Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

'sudo lvcreate -n apps-lv -L 14G webdata-vg'

'sudo lvcreate -n logs-lv -L 14G webdata-vg'

- Verify that your Logical Volume has been created successfully by running 

'sudo lvs'

- Verify the entire setup 

'sudo vgdisplay -v #view complete setup - VG, PV, and LV'

'sudo lsblk'

![sudo vgdisplay 1](./images/sudo%20vgdisplay%3Asudo%20lsblk.png)

![sudo vgdisplay 2](./images/sudo%20vgdisplay%3Asudo%20lsblk%202.png)

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem

'sudo mkfs -t ext4 /dev/webdata-vg/apps-lv'

'sudo mkfs -t ext4 /dev/webdata-vg/logs-lv'

![sudo mkfs](./images/sudo%20mkfs%20-t.png)

- Create /var/www/html directory to store website files

'sudo mkdir -p /var/www/html'

- Create /home/recovery/logs to store backup of log data

'sudo mkdir -p /home/recovery/logs'

- Mount /var/www/html on apps-lv logical volume

'sudo mount /dev/webdata-vg/apps-lv /var/www/html/'

![mount var/ww/html](./images/sudo%20mount%20%3Adev.png)

- Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

'sudo mount /dev/webdata-vg/logs-lv /var/log'

- Restore log files back into /var/log directory

'sudo rsync -av /home/recovery/logs/. /var/log'

- Update /etc/fstab file so that the mount configuration will persist after restart of the server.

'sudo blkid'

'sudo vi /etc/fstab'

![fstab](./images/fstab%20.png)

- Test the configuration and reload the daemon
 'sudo mount -a'

 'sudo systemctl daemon-reload'

 - Verify your setup by running df -h

 ## Step 2 - Step 2 — Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

## Step 3 — Install WordPress on your Web Server EC2

- Update the repository

'sudo yum -y update'

'sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json'

![wget httpd](./images/wget%20httpd%20php.png)

- Start Apache

'sudo systemctl enable httpd'

'sudo systemctl start httpd'

- To install PHP and it’s depemdencies

'sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1'

'sudo systemctl restart httpd'

- Download wordpress and copy wordpress to var/www/html
  
  'mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/'

  ![wordpress](./images/mkdir%20wordpress.png)

 - Configure SELinux Policies
  
  'sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1'

![chown](./images/chcon%20httpd.png)

  - Step 4 — Install MySQL on your DB Server EC2
'sudo yum update'

'sudo yum install mysql-server'

'sudo systemctl restart mysqld'

'sudo systemctl enable mysqld'

## Step 5 — Configure DB to work with WordPress

'sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit'

![mysql](./images/start%20mysq%20in%20db.png)

## Step 6 - Step 6 — Configure WordPress to connect to remote database.

![mysql connect](./images/sud%20mysql%3Amysql%20u%20root%20p.png)

![wordpress webpage](./images/Wordpress%20final.png)

![wordpress webpage](./images/wordpress%20finl.png)


