# WEB SOLUTION WITH WORDPRESS

Readme
In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

The focus of this  project will be:

* Configuration storage subsystem for Web and Database servers based on Linux OS. 

* Installation of WordPress as the web application layer, and connect it to a remote MySQL database server.

## Step 1 — Prepare a Web Server

NOTE: I spinned up an ec2 instance with red hat linux OS AMI.

* Launch an EC2 instance that will serve as "Web Server". Create 3 EBS volumes in the same AZ as your Web Server EC2, each of 10 GiB size.
* Attach each of the EBS volumes to your web server instance.
* Open up the Linux terminal to begin configuration, ssh into your server.
* Verify that the block devices are attached to the server. 

> `lsblk`

All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there; their names will likely be xvdb, xvdc, xvdf.

![proj5_1](https://user-images.githubusercontent.com/114786664/204256866-df3a6b89-c737-4ddf-8874-e7eb5f2166f2.png)

* Use `df -h` command to see all mounts and free space on the server.
* Use gdisk utility to create a single partition (10GB) on each of the 3 disks:

```
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdh

```
* Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

* Install lvm2 package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.
* Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

```
* Verify that the physical volumes were created successfully by running 

> ` sudo pvs `

![Screenshot from 2022-10-19 21-24-52](https://user-images.githubusercontent.com/114786664/204256896-84e78f4e-6a4c-4451-8811-1d280e9843a6.png)

* Using the `vgcreate` utility, create a volume group named 'webdata-vg' and add all the three PVS to the volume group.

> ` sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1 `

* Verify that the volume group was created successfully using 

> ` sudo vgs `

![p5_5](https://user-images.githubusercontent.com/114786664/204256828-729627aa-5ccf-429f-b17a-0a175b9be398.png)


* Using the `lvcreate ` utility, create two logical volumes(LVs); 'apps-lv' and 'logs-lv'. Assign each LVs about half of the volume group size which is about 30GB.

- I assigned both 14GB size; apps-lv will be used to store data for the Website while logs-lv will be used to store data for logs.

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg

``` 

* Verify that the LVs were created successfully using 
> ` sudo lvs `

![p5_6](https://user-images.githubusercontent.com/114786664/204256832-f889f44e-858d-4fc3-9d6f-2f1fe6ce8442.png)

* Verify the entire setup:

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk

```

![proj5_3](https://user-images.githubusercontent.com/114786664/204256886-39c46ba7-6243-4b65-9ea5-98f45cff6097.png)

* Use mkfs.ext4 to format the logical volumes with ext4 filesystem:

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

```

* Create /var/www/html directory to store website files:

> ` sudo mkdir -p /var/www/html `

* Create /home/recovery/logs to store backup of log data:

> ` sudo mkdir -p /home/recovery/logs `

* Mount /var/www/html on apps-lv logical volume:

> ` sudo mount /dev/webdata-vg/apps-lv /var/www/html/ `

* We shall mount 'logs-lv' logical volume on /var/log directory . Since this mount point isn't an empty directory, there is need to backup its content before mounting as mounting would cover up this content and will render it unavailable until one unmounts..

- Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs :

> ` sudo rsync -av /var/log/. /home/recovery/logs/ `

- Proceed to mount logs-lv logical volume on /var/log. (Note that all the existing data on /var/log will be covered up and rendered unavailable to whatever process uses them)

> ` sudo mount /dev/webdata-vg/logs-lv /var/log `

- Then, restore the log files back into /var/log directory:

> ` sudo rsync -av /home/recovery/logs/. /var/log `


```
 sudo blkid
 sudo vi /etc/fstab

``` 

* Test the configuration and reload the daemon. The ` mount -a ` command checks to ensure that no error was made while updating the fstab file.

```
 sudo mount -a
 sudo systemctl daemon-reload

```

* Verify the setup by running 
> ` df -h `

![p6_6](https://user-images.githubusercontent.com/114786664/204256841-c2e0651f-e6e7-482a-bd63-bec1e6496f6b.png)

## Step 2 — Prepare the Database Server

*  A second RedHat EC2 instance was launched and the above steps were repeated. instead of 'apps-lv' logical volume, 'db-lv' was created and mounted to '/db' directory instead of /var/www/html directory.


## Step 3 — Install WordPress on the Web Server EC2

* After updating the server ` sudo yum update -y `, install wget, Apache and its dependencies

> ` sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json `

- Start Apache

```
 sudo systemctl enable httpd
 sudo systemctl start httpd

```

- To install PHP and it’s dependencies using REMI repository

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

` sudo systemctl restart httpd `

- Download wordpress and copy wordpress to var/www/html

```
  mkdir wordpress
  cd  wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/

```

* Configure SELinux Policies

```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1

```


## Step 4 — Install MySQL on the DB Server EC2

* Update and install mysql server:

``` 
 sudo yum update
 sudo yum install mysql-server

```

* Verify that the service is up and running by using 

> `sudo systemctl status mysqld `.

![p6_mysql](https://user-images.githubusercontent.com/114786664/204256862-a775cfa2-da61-4aa3-a455-30d10b92442d.png)

---

## Step 5 — Configure DB to work with WordPress.

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER 'myuser'@'172.31.84.166' IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'172.31.84.166';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit

```

## Step 6 — Configure WordPress to connect to remote database.

- Since our web server housing wordpress will be required to connect to our database server, ensure to configure the database server to allow remote access by editing/adding the bind address (set to 0.0.0.0) in the mysql configuration directory; '/etc/my.cnf'.
 
> ` sudo vi /etc/my.cnf `

- Add : 	[mysqld]
		bind-address=0.0.0.0

- We should also open mysql port 3306 on the database server to allow traffic from the private IP of our web server.

* Install mysql client on the web server and test that you can connect to the database server.

> ` sudo yum install mysql `
> ` sudo mysql -u admin -p -h <DB-Server-Private-IP-address> `

* Verify that you can execute the 'SHOW DATABASES;' to access the list of database therein.

![Mysql from server](https://user-images.githubusercontent.com/114786664/204256824-73e44bf0-cceb-4b91-9003-51a026d3afee.png)

![mysql](https://user-images.githubusercontent.com/114786664/204256819-8e152b78-c7b8-477a-86d6-2026a5976862.png)

* Change permissions and configuration so Apache could use WordPress

> ` sudo chown -R apache:apache /var/www/html/wordpress `

* From your browser, try to access the link to your WordPress ` http://<Web-Server-Public-IP-Address>/wordpress/ `

![wp](https://user-images.githubusercontent.com/114786664/204256901-613b5cc3-9306-4b33-b937-301f8679c51c.png)

- Unlike other files, wp-config.php file does not come built-in with WordPress rather it’s generated specifically for your site during the installation process.

- Without this information your WordPress website will not work, and you will get the ‘error establishing database connection‘ response.

![Screenshot from 2022-10-20 17-35-37](https://user-images.githubusercontent.com/114786664/204256898-b85ac808-5fe1-4387-8aaa-e737ba1fe238.png)

- The wp-config.php file should be opened and the required database information should be updated.

![wp2](https://user-images.githubusercontent.com/114786664/204256906-79f44f0c-2165-4d67-bcaa-2e1810a507b9.png)

