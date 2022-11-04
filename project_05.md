# IMPLEMENTATION OF A CLIENT/SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS) IN AWS

MYSQL is a relational database with a three layered architecture. The layers of the architecture include a server end (comprises the logic of the DB), the storage engine (hold every user-created table in the database system.), and the client-end or query execution end (interract with the end-user). The Client/Server Architecture model entails only the two  of the three arms.

> ### Steps
1. Lauching two EC2 instances
2. Configuration of Mysql server 
3. Configuration of Mysql client 
4. Connecting to the server from the client side

## Step 1: Lauching two EC2 instances

* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user with administrative privilege on the root account for security best practices
* Launch two new EC2 instances of t2.micro family with Ubuntu Server 20.04 LTS (HVM)
- After launching, the first one should be renamed *server* while the second instance *client*

---

## Step 2: Configuration of Mysql server 
- Update the server
- Upgrade and 
- Install Mysql server
- All three tasks can be done using a single line of command:

> `sudo apt update && sudo apt upgrade -y && sudo apt install mysql-server `

- **Troubleshoot the Mysql_Secure_Installation error by:**

>> i. Open up the MySQL prompt: ` sudo mysql `

>> ii. ALTER USER command to change the root user’s authentication method to one that uses a password

>>> ` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; `

>> iii. Exit the MySQL prompt

>> iv. Run ` mysql_secure_installation ` to set password
- Login back to Mysql server using the newly created password

- Create a user with below command:

> ` CREATE USER 'y2o'@'%' IDENTIFIED WITH mysql_native_password BY 'password'; `

- Create a database name

> ` create database test_db `

- Verify that the database is created with:

> ` show databases; `

![server_side](https://user-images.githubusercontent.com/114786664/199962845-42c3d4ae-9419-46c4-85e7-f7f7930ba1c2.png)

-  Grant privileges with:

> `GRANT ALL PRIVILEGES ON *.* TO 'y2o'@'%' WITH GRANT OPTION; `

- Flush privileges with:

> `flush privileges; `

- Exit Mysql

- Reload/Restart MySQL:

> `sudo systemctl reload mysql ` and check the status with:

> `sudo systemctl status mysql `

- We'll need to configure the sg TCP port of the Mysql server to the default 3306 from anywher, or better still from the client server private ip

![sg_rule](https://user-images.githubusercontent.com/114786664/199962989-fbda6e0c-b1ab-481c-93f7-52bd7f900c7f.png)

- Finally, we'll need to configure Mysql server to accept connection from remote host by changing the **bind-address** to **0.0.0.0** 
using the vim editor from this path:

> `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf `

---

## Step 3:Configuration of Mysql client 

- Update, upgrade, and install mysql:

> `sudo apt update;sudo apt upgrade -y; sudo apt install mysql `

---

## Step 4:Connecting to the server from the client side

- Connect to the server from the client with the following command:

> `sudo mysql -h  172.31.84.56 -u y2o -p `

![connectn_from_client](https://user-images.githubusercontent.com/114786664/199963087-a408928a-c889-49cc-88ac-43d9f6077a6c.png)

---
__END__