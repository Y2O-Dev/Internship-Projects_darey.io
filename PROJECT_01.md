##Web Stack (LAMP) Implementation in AWS
A web stack is a compilation of technologies, often needed for web or mobile  applications especially implementing websites. Its a solution stack that comprised of frameworks, programming languages, patterns, servers, libraries, softwares etc used by it developers. Its otherwise called web application stack.
**LAMP**
>Linux
>Apache
>MySQL
>PHP

##Steps
1. Lauching virtual server (EC2) on AWS 
2. Installing apache and updating the firewall
3. Installing mysql
4. Installing php
5. Creating a virtual host for your website using apache
6. Enable php on the website

##Step 1 – Lauching virtual server (EC2) on AWS 
* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user with administrative privilege on the root account for security best practices
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

![ubuntu instance](https://user-images.githubusercontent.com/114786664/193413174-37e7eea8-90cf-4ef4-a9c5-c45821b82057.png)

### Connecting to the ec2 terminal
* change directory to the directory wherein your key pair is located;
` cd ~/Downloads `

* Change premissions for the private key file (.pem),
` sudo chmod 0400 <private-key-name>.pem `

* ssh into the ec2 terminal using the bash shell.
` ssh -i <private-key-name>.pem ubuntu@<Public-IP-address> `

![ssh ubuntu](https://user-images.githubusercontent.com/114786664/193413334-9650025a-0e8b-435c-bce4-544cc1e2a169.png)

* Use the public IP on the instance for remote login on the Linux server.
* For remote server login, I used linux bash.

##Step 2 – Installing Apache and updating the firewall

* Install Apache using Ubuntu’s package manager ‘apt’:

- update a list of packages in package manager
>` sudo apt update `
- run apache2 package installation
>` sudo apt install apache2 `
- Verify that Apache is running
>` sudo systemctl status apache2 `
* To allow our web server to receive traffic from web users, edit inbound rule on the EC2 security group. Allow HTTP/TCP traffic on port 80 from anywhere.

* Check if you can access the web server locally from the ubuntu shell
```
> curl http://localhost:80
or
> curl http://127.0.0.1:80
```
* To test how our Apache server will receive requests from the internet, open a web browser and access the folowing URL:
>` http://<Public-IP-Address>:80 `

![apache2](https://user-images.githubusercontent.com/114786664/193413427-55ec9813-e513-4bd4-9f4b-cdcbc87312cd.png)

![Apache](https://user-images.githubusercontent.com/114786664/193413454-e733b819-debe-4f6a-b32c-ae9f871335e4.png)


* Getting the IP address by checking the instance metadata via the command:
>` curl -s http://169.254.169.254/latest/meta-data/public-ipv4 `

## STEP 3 — installing mysql

* Install MySQL using Ubuntu’s package manager ‘apt’:
>` sudo apt install mysql-server `
- Log in to the MySQL console
>` sudo mysql `

![mysql page](https://user-images.githubusercontent.com/114786664/193413511-ee018e5e-2a50-4e0d-8f39-b6fa5f4f38a3.png)

* Set a password for the root user, using mysql_native_password as default authentication method. I defined the user’s password as PassWord.1
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

## STEP 4 — installing php

- In addition to installing the 'php package', you'll need to install 'php-mysql'(a PHP module that allows PHP to communicate with MySQL-based databases), and 'libapache2-mod-php'(to enable Apache to handle PHP files).
* Install the above listed requirements in a single run:
>` sudo apt install php libapache2-mod-php php-mysql `
- After installation, confirm the version of the php: 
>` php -v `

To test the setup with a PHP script, it is best to set up (an Apache Virtual Host](https://httpd.apache.org/docs/2.4/vhosts/) to hold your website’s files and folders.

## STEP 5 — creating a virtual host for your website using apache
- Here, we will set up a domain called 'projectlamp'
* Create a web root directory for 'projectlamp' using the mkdir command
>` mkdir /var/www/projectlamp `

* Assign ownership of the directory to your current system user:
>` sudo chown -R $USER:$USER /var/www/projectlamp `

* Create and open a new configuration file in Apache’s sites-available directory using vim editor;
>` sudo vi /etc/apache2/sites-available/projectlamp.conf `

- press 'i' to enter the insert mode and paste the following bare-bones configuration:

```
<VirtualHost *:80>
    ServerName projectlamp
    ServerAlias www.projectlamp 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/projectlamp
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Press the 'esc' key, followed by :wq to save the text and exit the vim editor page.

* use the ls command to check the newly created configuration file in Apaches's sites available directory.
>` sudo ls /etc/apache2/sites-available `

![confg files](https://user-images.githubusercontent.com/114786664/193413623-1908ee15-f946-41f7-a668-88903373894f.png)


- As seen above, the '000-default' directory is the config file for the server block enabled by default in Apache to serve documents from the /var/www/html directory. If not disabled, it will overwrite the projectlamp config file when one tries to access the website URL using its public IP address. 

* Enable the new virtual host using 'a2ensite' command
>` sudo a2ensite projectlamp `

* Disable apache's default website using 'a2dissite' command
>` sudo a2dissite 000-default `

* To make sure your configuration file doesn’t contain syntax errors, run:
>` sudo apache2ctl configtest `

* Reload Apache so these changes take effect:
>` sudo systemctl reload apache2 `

* Create an index.html file in the projectlamp web directory */var/www/projectlamp/* so that we can test that the virtual host works:
` sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html `.
- Open your browser and try to access your website using the public IP address
` http://<Public-IP-Address>:80 `

![php page](https://user-images.githubusercontent.com/114786664/193413752-eba18906-ec7d-47ea-8369-6766a4d3439b.png)
