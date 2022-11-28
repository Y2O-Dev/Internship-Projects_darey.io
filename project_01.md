## Web Stack (LAMP) Implementation in AWS

A web stack is a compilation of technologies, often needed for web or mobile  applications especially implementing websites. Its a solution stack that comprised of frameworks, programming languages, patterns, servers, libraries, softwares etc used by it developers. Its otherwise called web application stack.

**LAMP**
- Linux
- Apache
- MySQL
- PHP

## Steps
1. Lauching virtual server (EC2) on AWS 
2. Installing apache and updating the firewall
3. Installing mysql
4. Installing php
5. Creating a virtual host for your website using apache
6. Enable php on the website

## Step 1 – Launching A Virtual Server (EC2) on AWS 

* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user with administrative privilege on the root account for security best practices
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

![ubuntu instance](https://user-images.githubusercontent.com/114786664/194324470-d837624d-ff27-4d47-95be-f857b0c80ba8.png)

### Connecting to the EC2 terminal
* change directory to the directory wherein your key pair is located;

` cd ~/Downloads `

* Change premissions for the private key file (.pem),

` sudo chmod 0400 <private-key-name>.pem `

* ssh into the ec2 terminal using the bash shell.

` ssh -i <private-key-name>.pem ubuntu@<Public-IP-address> `

![ssh ubuntu](https://user-images.githubusercontent.com/114786664/194324824-8bcf9052-f96c-48f6-b743-62d05887d410.png)

* Use the public IP on the instance for remote login on the Linux server.
* For remote server login, I used linux bash.

## Step 2 – Installing Apache and Updating Firewall
* Install Apache using Ubuntu’s package manager ‘apt’:

- update a list of packages in package manager

>` sudo apt update `

- run apache2 package installation

>` sudo apt install apache2 `

- Verify that Apache is running

>` sudo systemctl status apache2 `

![apache2](https://user-images.githubusercontent.com/114786664/194326860-3c6d2c88-62ea-4fad-a3f6-5bea6c64d731.png)

* To allow our web server to receive traffic from web users, edit inbound rule on the EC2 security group. Allow HTTP/TCP traffic on port 80 from anywhere.

* Check if you can access the web server locally from the ubuntu shell
```
> curl http://localhost:80
or
> curl http://127.0.0.1:80
```
* To test how our Apache server will receive requests from the internet, open a web browser and access the folowing URL:

>` http://<Public-IP-Address>:80 `

![Apache](https://user-images.githubusercontent.com/114786664/194326890-9be15787-59a4-4566-9527-1c76a9b820f3.png)

* Getting the IP address by checking the instance metadata via the command:

>` curl -s http://169.254.169.254/latest/meta-data/public-ipv4 `

## STEP 3 — Installing MySQL

* Install MySQL using Ubuntu’s package manager ‘apt’:
>` sudo apt install mysql-server `
- Log in to the MySQL console
>` sudo mysql `

![mysql page](https://user-images.githubusercontent.com/114786664/194327432-76612612-e2b0-4600-8709-cc83e2694d0b.png)

* Set a password for the root user, using mysql_native_password as default authentication method. I defined the user’s password as PassWord.1 using the following command:

>` ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1'; `

- Exit the MySQL shell with: 

>` mysql> exit `

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

## STEP 4 — Installing PHP

- In addition to installing the 'php package', you'll need to install 'php-mysql'(a PHP module that allows PHP to communicate with MySQL-based databases), and 'libapache2-mod-php'(to enable Apache to handle PHP files).

* Install the above listed requirements in a single run:

>` sudo apt install php libapache2-mod-php php-mysql `

- After installation, confirm the version of the php: 

>` php -v `

To verify the setup with a PHP script, it is best to set up (an Apache Virtual Host](https://httpd.apache.org/docs/2.4/vhosts/) to hold your website’s files and folders.

## STEP 5 — Creating a Virtual Host for the Website Using Apache

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

* use the *ls* command to check the newly created configuration file in Apaches's sites available directory.

>` sudo ls /etc/apache2/sites-available `

![confg files](https://user-images.githubusercontent.com/114786664/194328149-c1debfa5-62c7-4a1a-b591-433bced3653e.png)

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

>``` sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html ``` 

* Open your browser and try to access your website using the public IP address

>` http://<Public-IP-Address>:80 ` 

![web browser display](https://user-images.githubusercontent.com/114786664/194328421-186f76ca-9eba-4b44-ba62-cdabb38e85a1.png)

## STEP 6 — Enable PHP on the website

* There is need to modify the default directory index settings on apache to enable the loading of the .php file instead of the .html file. The */etc/apache2/mods-enabled/dir.conf* file will be modified to change the order in which the index.php file is listed within the DirectoryIndex directive:

`sudo vim /etc/apache2/mods-enabled/dir.conf`

```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

* Saving and closing the file, there is need to reload Apache so the changes take effect

`sudo systemctl reload apache2`

* Create a new file named *index.php* inside your custom web root folder */var/www/projectlamp/*:

`vim /var/www/projectlamp/index.php`

```
<?php
phpinfo();
```

![php page](https://user-images.githubusercontent.com/114786664/194328693-5431349a-f8f8-4c0d-93bc-f78177e8fbc3.png)

* It’s best to remove the file you created as it contains sensitive information about your PHP environment -and your Ubuntu server.

`sudo rm /var/www/projectlamp/index.php`
