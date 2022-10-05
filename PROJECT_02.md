## Web Stack (LEMP) Implementation in AWS
A web stack is a compilation of technologies, often needed for web or mobile  applications especially implementing websites. Its a solution stack that comprised of frameworks, programming languages, patterns, servers, libraries, softwares etc used by it developers. Its otherwise called web application stack.

**LEMP**
- Linux
- Nginx
- MySQL
- PHP

## Steps
0. Lauching virtual server (EC2) on AWS 
1. Installing the Nginx web server
2. Installing mysql
3. Installing php
4. Configuring Nginx to use php
5. Testing php with Nginx
6. Retrieving data from mysql db with php

## Step 0 – Lauching virtual server (EC2) on AWS 
* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user with administrative privilege on the root account for security best practices
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

### Connecting to the EC2 terminal
* change directory to the directory wherein your key pair is located;
` cd ~/Downloads `

* Change premissions for the private key file (.pem),
` sudo chmod 0400 <private-key-name>.pem `

* Login to the Ubuntu server using the instance public IP address.

* ssh into the ec2 terminal using the bash shell.
` ssh -i <private-key-name>.pem ubuntu@<Public-IP-address> `

![1 ssh](https://user-images.githubusercontent.com/114786664/194129443-7eaf51ff-f928-441b-957b-50ba3cca21c4.png)

## Step 1 – Installing the Nginx web server

* To install nginx web server, first update the list of packages in package manager and subsequent Nginx installation using ‘apt’:

>` sudo apt update; sudo apt install nginx `

- To verify that Nginx is running:

>` sudo systemctl status nginx `

* Check if you can access the web server locally from the ubuntu shell
```
> curl http://localhost:80
or
> curl http://127.0.0.1:80
```
* To test how our nginx server will receive requests from the internet, open a web browser and access the folowing URL:
>` http://<Public-IP-Address>:80 `

![2 nginx](https://user-images.githubusercontent.com/114786664/194129838-08f8d9c6-ebfa-4154-92e8-2c3ae048dff1.png)

![2_nginx](https://user-images.githubusercontent.com/114786664/194129755-98ccb6ec-ec52-42a1-8f1b-ffa9dd4e440d.png)

* The public IP address can be retrieved using the command below:
>` curl -s http://169.254.169.254/latest/meta-data/public-ipv4 `

## STEP 2 — installing mysql

* Install MySQL using Ubuntu’s package manager ‘apt’:
>` sudo apt install mysql-server `
- Log in to the MySQL console
>` sudo mysql `

![3_mysql](https://user-images.githubusercontent.com/114786664/194130084-fd9222ba-6364-41b3-8730-eaddf7f864ce.png)


* Set a password for the root user, using mysql_native_password as default authentication method. I defined the user’s password as Mukumuku.1
>` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'; `
- Exit the MySQL shell with: ` mysql> exit `

* Start an interactive script that removes some insecure default settings and lock down access to your database system.
>` sudo mysql_secure_installation `
- When asked if to validate password plugin, I chose "No". Enabling "validating password plugin" allows you to define password strength as either low, medium or strong. It is safe to leave this feature disabled, but always use strong, unique passwords for databases.

* Select and confirm a password for the MySQL root user (not to be confused with the system root user).
- The database root user is an administrative user with full privileges over the database system. Define a strong password here for additional security measures.
- For the rest of the questions, press Y and hit the ENTER key at each prompt.
- Test if you are able to log in to your MySQL console: 
>` sudo mysql -p `
- Exit the MySQL console: 
>` mysql> exit `

## STEP 3 — installing php

- Unlike installing php on apache that embed the php interpreter, Nginx require an external program to  handle php. The result of this is better performance. To this effect, php-fpm (php fastCGI process manager) will be installed. It accept php request from Nginx for processing.
- In addition to the above requirement, php-mysql will be installed which allows php to communicate with mysql database.

* Both packages istalled with a single command:
>` sudo apt install php-fpm php-mysql `

* Install the above listed requirements in a single run:
>` sudo apt install php libnginx2-mod-php php-mysql `
- After installation, confirm the version of the php: 
>` php -v `

## STEP 4 — Configuring Nginx to use PHP Processor

- Here, we will set up a domain called 'project_lemp'
* Create a web root directory for 'project_lemp' using the mkdir command
` mkdir /var/www/project_lemp `

* Assign ownership of the directory to your current system user:
>` sudo chown -R $USER:$USER /var/www/project_lemp `

* Create and open a new configuration file in Nginx’s sites-available directory using nano editor;
>` sudo nano /etc/nginx/sites-available/project_lemp `

- Paste the bare-bones configuration, Ctrl+O to save and Ctrl+L to exit nano.


* Use the ln command to activate the configuration by linking it to the config file in nginxs's sites available directory.
>` sudo ln /etc/nginx/sites-available/project_lemp etc/nginx/sites-enabled/ `

* Configuration should be checked for syntax error using this code:
> ` sudo nginx -t `

![4_nginx_enabled](https://user-images.githubusercontent.com/114786664/194137469-f91f9179-7b51-49fd-84ef-8039998644b7.png)
 
 * We need to disable the default Nginx host and reload it using the follwing commands respectively:
 > ` sudo unlink /etc/nginx/site_enabled/default `
 
 > ` sudo systemctl reload nginx `

 * Now the web browser will display the below diagram, meaning the nginx is working well.
 
 ![5_nginx_web](https://user-images.githubusercontent.com/114786664/194147327-99c3fc5a-f931-47e8-8019-a7c091cc28e3.png)
 
 ## STEP 5 — Testing PHP with Nginx
* To test if nginx can handle request from .php file, we can create a php file in the project_lemp directory using nano,

> ` sudo nano /var/www/projectLEMP/info.php `
 
* Input a valid php code:

``` 
<?php
phpinfo();
```
* The web browser should display a detailed php page as shown below.

![6_php](https://user-images.githubusercontent.com/114786664/194138069-fb5978ae-c8cf-4c9b-8316-e4a4b5a49f09.png)

**NB: The info.php file should be removed after confirmation as it contains sensitive info about the server. This can be done using the "rm" command.**

 ## STEP 6 — Retrieving data from mysql database with PHP
 
* In this step you will create a test database (DB) with simple "To do list", and configure access to it. The Nginx website can then query data from the DB and display it.

![8_example_database](https://user-images.githubusercontent.com/114786664/194138293-b63f90d4-d0b2-45f5-917e-eaf4696ff8c9.png)

![8_output](https://user-images.githubusercontent.com/114786664/194138381-cb1c473e-9e73-4d1e-833a-92a5320f2802.png)

![9_web_item](https://user-images.githubusercontent.com/114786664/194138542-0031d7a5-7913-4ee4-b413-5712a71b9b2c.png)

