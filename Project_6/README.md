# Full-scale Web Solution using WordPress CMS and MySQL RDBMS
## Your 3-Tier Setup

1. A Laptop or PC to serve as a client
2. An EC2 Linux Server as a web server (This is where you will install WordPress)
3. An EC2 Linux server as a database (DB) server

![Alt text](Images/3tier%20app.webp)

We'll be using RedHat OS for this project
In previous projects we used ‘Ubuntu’, but it is better to be well-versed with various Linux distributions, thus, for this projects we will use very popular distribution called ‘RedHat’ (it also has a fully compatible derivative – CentOS)
Note: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>


## LAUNCH AN EC2 INSTANCE THAT WILL SERVE AS “WEB SERVER”.

## Step 1 — Prepare a Web Server

1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
Learn How to Add EBS Volume to an EC2 instance [here](https://www.youtube.com/watch?v=HPXnXkBzIHw)
![Alt text](Images/step%201%20launch%20ec2%20instance.png)
![Alt text](Images/step%201.0%20launch%20ec2%20Red%20Hat.png)
![Alt text](Images/notice%20az%20and%20hit%20volumes.png)
![Alt text](Images/create%203%2010gb%20volumes.png)

- Create 10gb 3 times in the same avalibilty zone as our instance**:
![Alt text](Images/create%203%2010gb%20volumes.png)
![Alt text](Images/10gb%20az%201a%20and%20save%20x3.png)

- Now we attach all volumes created to our instance 
![Alt text](Images/now%20we%20attach%20the%203%20volumes.png)

![Alt text](Images/select%20instance%20to%20be%20attched%20to%20and%20attach.png)

2. Connect via ssh into your terminal to begin configuration

![Alt text](Images/connect%20to%20the%20p6%20instance.png)

3. Use `lsblk` command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with `ls /dev/` and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

- `lsbk`
![Alt text](Images/4%20steps%20lsblk.png)

- `ls /dev/`

![Alt text](Images/ls%20dev%20xvdf-h%20showing.png)


4. Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/xvdf`

- when promted for an input:
`n` press enter button 4 times for defualt settings
`p` press enter 
`w` press enter
`yes` press enter
respectively

![Alt text](Images/nand%20enter%20p%20enter%20wy%20for%20xvdf-h.png)

- now repeat the above steps for **xvdg and xvdh**

5. Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![Alt text](Images/lblk.png)


6. Install lvm2 package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`
Verify that your Physical volume has been created successfully by running  :
`sudo pvs`

![Alt text](Images/pv%20for%20xvdf-h%20created.png)

7. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg:
 
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
 
Verify that your VG has been created successfully by running:
`sudo vgs`

![Alt text](Images/vgs.png)

8. Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
 
`sudo lvcreate -n apps-lv -L 14G webdata-vg`
`sudo lvcreate -n logs-lv -L 14G webdata-vg`
 
Verify that your Logical Volume has been created successfully by running:

`sudo lvs`

![Alt text](Images/lvs.png)

9. Verify the entire setup
 
`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

`sudo lsblk`

![Alt text](Images/sudo%20lblk.png)

10. Use mkfs.ext4 to format the logical volumes with ext4 filesystem
 
`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![Alt text](Images/mkfs%20ext4.png)
 
11. Create /var/www/html directory to store website files
 
`sudo mkdir -p /var/www/html`
 
12. Create /home/recovery/logs to store backup of log data
 
`sudo mkdir -p /home/recovery/logs`
13. Mount /var/www/html on apps-lv logical volume
 
`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

14. Use rsync utility to back up all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
 
`sudo rsync -av /var/log/. /home/recovery/logs/`

15. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted.

`sudo mount /dev/webdata-vg/logs-lv /var/log`
 
16. Restore log files back into /var/log directory
 
`sudo rsync -av /home/recovery/logs/. /var/log`
 
17. Update /etc/fstab file so that the mount configuration will persist after restart of the server.

`sudo blkid`

![Alt text](Images/sudo%20blkid.png)


18. Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes:

`sudo vi /etc/fstab`

![Alt text](Images/fstab%20mount%20edit%20webserver.png)

edit press i
to exit press esc and type :wq

19. Test the configuration and reload the daemon
 
`sudo mount -a`
`sudo systemctl daemon-reload`
 
20. Verify your setup by running df -h, output must look like this:
![Alt text](Images/df%20-h%20after%20dameon%20restart.png)


## Step 2 — Prepare a Database Server 
Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
Repeat the same steps as for the Web Server, but instead of `apps-lv` create `db-lv` and mount it to `/db` directory instead of `/var/www/html/`.

- launch a RedHat EC2 instance that will have a role – ‘DB Server’
![Alt text](Images/Snipaste_2022-12-15_14-45-27.png)

- Create volumes and attach them like we did earlier

![Alt text](Images/create%20db%20voumes%20and%20attach%20to%20same%20az.png)

- ssh into your DB instance let's get the show on the road

- `lsbk`

- `ls /dev/`

![Alt text](Images/db%20terminal%20setup%201.png)

- Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/xvdf`

![Alt text](Images/db%20terminal%20setup%202.png)

- when promted for an input:
`n` press enter button 4 times for defualt settings
`p` press enter 
`w` press enter
`yes` press enter
respectively

- now repeat the above steps for **xvdg and xvdh**


- Use `lsblk` utility to view the newly configured partition on each of the 3 disks.


- Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
![Alt text](Images/db%20terminal%20setup%203.png)

- Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

Verify that your Physical volume has been created successfully by running:

`sudo pvs`

![Alt text](Images/db%20terminal%20setup%204.png)

- Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG db-vg
 
`sudo vgcreate db-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
 
Verify that your VG has been created successfully by running:

`sudo vgs`

![Alt text](Images/db%20terminal%20setup%204.png)

- Use lvcreate utility to create 1 logical volumes. db-lv (we'll use 25G)
 
`sudo lvcreate -n db-lv -L 25G db-vg`

- Verify that your Logical Volume has been created successfully by running:

`sudo lvs`

![Alt text](Images/db%20terminal%20setup%206%20lvs.png)


-  Verify the entire setup
 
`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

`sudo lsblk`

![Alt text](Images/db%20terminal%20setup%207%20sudo%20lsblk.png)

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem
 
`sudo mkfs -t ext4 /dev/db-vg/db-lv`

![Alt text](Images/db%20terminal%20setup%208%20mk.png)

- Create a /db directory and mount it:

`sudo mkdir /db`

`sudo mount /dev/db-vg/db-lv /db`

`df -h`

![Alt text](Images/db%20terminal%20setup%2012%20mkdir%20db%20and%20mount.png)

`sudo blkid`

![Alt text](Images/db%20terminal%20setup%2013%20blkid.png)

`sudo vi /etc/fstab`

![Alt text](Images/db%20terminal%20setup%2014%20mount%20db.png)

## Step 3 — Install WordPress on your Web Server EC2

- Update the repository
 
`sudo yum -y update`
 
- Install wget, Apache and it’s dependencies
 
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`
 
- Start Apache

    sudo systemctl enable httpd
    sudo systemctl start httpd

- To install PHP and its dependencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
```

- Restart Apache

`sudo systemctl restart httpd`
 
- Download wordpress and copy wordpress to var/www/html

 ```
mkdir wordpress
cd   wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
 ```

- Configure SELinux Policies
 ```
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_can_network_connect_db 1
```

## Step 4 — Install MySQL on your DB Server

`sudo yum update`
`sudo yum install mysql-server`

- Verify that the service is up and running by using `sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:
```
sudo systemctl restart mysqld
sudo systemctl enable mysqld
```
- In the database server terminal 
  we'll do mysql secure installation

`sudo mysql_secure_installation`

enter n 
yes and enter till the end

![Alt text](Images/db%20terminal%20setup%2016.png)

## Step 5 — Configure DB to work with WordPress

`sudo mysql`
```
CREATE DATABASE wordpress;
CREATE USER 'myuser'@'<Web-Server-Private-IP-Address>' IDENTIFIED BY 'put your password';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
- Edit the bind address:

`sudo vi /etc/my.cnf`

        [webserver]
        bind-address=<your webserver private ip address>

![Alt text](Images/db%20terminal%20setup%2015.png)



## ## Step 6 — Install MySQL on your Web Server

`sudo yum update`
`sudo yum install mysql-server`

- Verify that the service is up and running by using `sudo systemctl status mysqld`, if it is not running, restart the service and enable it so it will be running even after reboot:

`sudo systemctl restart mysqld`
`sudo systemctl enable mysqld`

- Lets test that you can connect from your Web Server to your DB server by using mysql-client

   `sudo mysql -u <your db user name> -p -h <DB-Server-Private-IP-address>`

Verify if you can successfully execute `SHOW DATABASES;` command and see a list of existing databases.




- locate and edit the wp-config.php

`cd /var/www/html/wordpress/`
`ls -l`
`sudo vi wp-config.php`

![Alt text](Images/websever%201%20wp%20config.png)


- To disable the default apache homepage

`sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup`

![Alt text](Images/websever%202%20apache%20restart.png)

`sudo systemctl restart httpd`
`sudo systemctl status httpd`

## Step 7 — Configure WordPress to connect to the remote database.

- Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

![Alt text](Images/db%20sg%20inbound%20port%203306.png)


- Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

![Alt text](Images/Final%20piece.png)

- Try to access from your browser the link to your WordPress:

        http://<Web-Server-Public-IP-Address>/wordpress/

- Fill out your DB credentials:

![Alt text](Images/Final%20piece%202.png)

**If you see this message – it means your WordPress has successfully connected to your remote MySQL database**

![Alt text](Images/Final%20piece%203.png)



Important: Do not forget to STOP your EC2 instances after completion of the project to avoid extra costs.

You have learned how to configure Linux storage susbystem and have also deployed a full-scale Web Solution using WordPress CMS and MySQL RDBMS!

# CONGRATULATIONS!

![congrats](https://i.pinimg.com/originals/47/fd/28/47fd2856377747c0f51b0adcf3050791.gif)
