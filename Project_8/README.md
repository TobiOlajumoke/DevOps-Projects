sudo lvcreate -n apps-lv -L 7G webdata-vg
sudo lvcreate -n logs-lv -L 7G webdata-vg
sudo lvcreate -n opt-lv -L 7G webdata-vg


sudo mkfs -t xfs /dev/webdata-vg/apps-lv
sudo mkfs -t xfs /dev/webdata-vg/logs-lv
sudo mkfs -t xfs /dev/webdata-vg/opt-lv


sudo mount /dev/webdata-vg/apps-lv /mnt/apps
sudo mount /dev/webdata-vg/logs-lv /mnt/logs
sudo mount /dev/webdata-vg/opt-lv /mnt/opt


sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
create user 'webaccess'@'172.31.48.0/20' identified by 'password';
grant all privileges on tooling.* to 'webaccess'@'172.31.48.0/20';


172.31.48.0/20

/mnt/apps 172.31.48.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.48.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.48.0/20(rw,sync,no_all_squash,no_root_squash)



## WEBSERVER CONFIG
sudo yum install nfs-utils nfs4-acl-tools -y
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.15.59:/mnt/apps /var/www


df -h

sudo vi /etc/fstab
add following line
172.31.15.59:/mnt/apps /var/www nfs defaults 0 0


## Install apache
```
sudo yum install httpd -y


sudo mount -t nfs -o rw,nosuid 172.31.15.59:/mnt/logs /var/log/httpd
```

`sudo vi /etc/fstab`

172.31.15.59:/mnt/logs /var/log/httpd nfs defaults 0 0


### clone tooling git
```
sudo yum install git -y
git init
git clone https://github.com/darey-io/tooling.git
```

cd to tooling 
move html file  to /var/www/html
```
sudo cp -R html/. /var/www/html
```








sudo setenforce 0
 `sudo vi /etc/sysconfig/selinux` and set SELINUX=disabled then restrt httpd.

 sudo systemctl start httpd
 sudo systemctl status httpd

 incase of error do 

 sudo chown -R apache:apache /var/log/httpd/


 check puplic ip in http://




install php 

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y 


sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y 


sudo dnf module reset php -y 


sudo dnf module enable php:remi-7.4 -y


sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y


sudo systemctl start php-fpm


sudo systemctl enable php-fpm


sudo setsebool -P httpd_execmem 1


sudo vi /var/www/html/functions.php
edit webbaccess and password and db ip addresss


sudo yum install mysql -y

cd tooling 
mysql -h 172.31.5.33 -u webaccess -p tooling < tooling-db.sql


## we checked db SG and added auroar

now check db status


sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

edit 
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0

restart mysql 

## go to websever
run 
mysql -h 172.31.5.33 -u webaccess -p tooling < tooling-db.sql



Rename the webpage to let the welcome webpage point else where 

sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.backup