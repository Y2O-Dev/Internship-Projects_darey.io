## DEVOPS TOOLING WEBSITE SOLUTION

In this project, I implemented solution that consists of following components: 

1. Infrastructure: AWS

2. Webserver Linux: 3 ec2 instances of Red Hat Enterprise Linux 8 AMI

3. Database Server: Ubuntu 20.04 + MySQL

4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server

5. Programming Language: PHP

6. Code Repository: GitHub

The architecture below shows a common pattern where several stateless Web Servers share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. 

![Architecture](https://user-images.githubusercontent.com/114196715/197935726-72d0fae3-620a-4d9f-bca1-62d644a0faa2.png)

---

## STEP 1 – PREPARE NFS SERVER

* Spin up a new EC2 instance with RHEL Linux 8 Operating System

* Configure LVM on the server

* Format the logical volumes as an xfs filesystem

* Create 3 Logical Volumes: lv-opt lv-apps, and lv-logs

* Create mount points on /user directory for the logical volumes as follow:
> - Mount lv-apps on /user/apps – To be used by webservers
> - Mount lv-logs on /user/logs – To be used by webserver logs
> - Mount lv-opt on /user/opt – To be used by Jenkins server in the next project.

* Install NFS server, configure it to start on reboot and make sure it is up and running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service

```

* Configure NFS to allow access to clients within the same subnet CIDR

```
sudo vi /etc/exports

/user/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/user/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/user/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)


```
- Run the exportfs command which makes local directories available for Network File System (NFS) clients to mount. It uses information in the '/etc/exports' file to export one or more directories, which must be specified with full path names.

> ` sudo exportfs -arv `

* Check which port is used by NFS and open it by editing inbound rules of the security group.

> ` rpcinfo -p | grep nfs `

NOTE: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049.

![sg_mod](https://user-images.githubusercontent.com/114786664/204312153-41d1274d-c19f-4a1d-9010-d9bc8b493069.png)

---

## STEP 2 — CONFIGURE THE DATABASE SERVER

- Configure a MySQL DBMS to work with remote Web Server
* Install MySQL server
* Create a database and name it 'tooling'
* Create a database user and name it 'webaccess'
* Grant permission to 'webaccess' user on 'tooling' database to do anything only from the webservers subnet cidr.

![tooling_db](https://user-images.githubusercontent.com/114786664/204312156-c0ede033-a650-45aa-a8df-618a03ba8287.png)

---

## Step 3 — PREPARE THE WEB SERVERS

- We want to ensure that our Web Servers can serve the same content from shared storage solutions, in this case – NFS Server and MySQL database.
- For storing shared files that our Web Servers will use, we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).
- This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

In the next steps, we shall do the following:

- Configure NFS client (this step must be done on all three servers).
- Deploy a Tooling application to our Web Servers into a shared NFS folder.
- Configure the Web Servers to work with a single MySQL database.

* Launch a new EC2 instance with RHEL 8 Operating System

* Install NFS client

> ` sudo yum install nfs-utils nfs4-acl-tools -y `

* Mount /var/www/ and /var/log/httpd/access_logs and target the NFS server’s export for apps as well as for logs.

``` 
sudo mkdir /var/www 
```
- I mounted directly by running ` sudo mount -a ` after editing the entries in the '/etc/fstab' file to include the NFS server's export for apps and logs.

> ` <NFS-Server-Private-IP-Address>:/user/apps /var/www nfs defaults 0 0 `

![etc_fstab](https://user-images.githubusercontent.com/114786664/204312142-a0194e52-c500-4146-8a66-1b312680df14.png)

---

* Verify that NFS was mounted successfully by running `mount`

* Install 'Remi repository' ( provides the latest versions of the PHP stack, full featured, and some other software, to the Fedora and Enterprise Linux users), APACHE and PHP.

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

setsebool -P httpd_execmem 1

```
![apache page](https://user-images.githubusercontent.com/114786664/204312130-02e9182d-cdd6-4d6e-b1a8-5ed850c77d19.png)

---

*Repeat the above steps in 'STEP 3' for another two ec2 instances*.

* Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /user/apps. 

- If you see the same files, it means NFS is mounted correctly.

- Create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

* Locate the log folder for Apache on the Web Server (Default access log file location for RHEL is  /var/log/httpd/access_log) and mount it to NFS server’s export for logs. Persist the mount by adding entry into /etc/fstab file.


* Fork the tooling source code from [Darey.io Github Account](https://github.com/darey-io/tooling) to your Github account

- clone the repository to your workstation

```
sudo yum install git
git init
git clone https://github.com/darey-io/tooling
cd tooling

```

* Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html.

- move the content of the /html directory in the tooling folder to /var/www/html.

- NOTE 1: Do not forget to open TCP port 80 on the Web Server.

- NOTE 2: If you encounter 403 Error, check permissions to your /var/www/html folder and also disable SELinux 

> ` sudo setenforce 0 `

- To make this change permanent,open following config file 

> ` sudo vi /etc/sysconfig/selinux ` and 

- set SELINUX=disabled,then restart httpd.

* Ensure the database server allows remote access by editing the bind address in '/etc/mysql/mysql.conf.d/mysqld.cnf'

* Create in MySQL, a new admin user with username: myuser and password: password:

```
INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);

```

* Update the website’s configuration to connect to the database (in /var/www/html/functions.php file).


* On the web server, install mysql client and connect to the database server.

- Apply tooling-db.sql script to your database using this command 

> `mysql -h <database-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

* Open the website in your browser

- Make sure you can login into the website with myuser user.

![final](https://user-images.githubusercontent.com/114786664/204312145-554df781-c76c-434a-848e-4170a82c5f89.png)

---

![set](https://user-images.githubusercontent.com/114786664/204312150-211733ad-25c7-4d93-90ec-5f22d651f996.png)
---

__END__
