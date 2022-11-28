## MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

MEAN is a free and open-source JavaScript software stack for building dynamic web sites and web applications. 
A variation known as MERN used in the previous project (4) replaces Angular with React. 
The MEAN stack is made up of the following four components:

**MEAN**
* MongoDB
* Express.js
* AngularJS
* Node.js

Since all its components are in JavaScript, both front-end and back-end environment can be written in one language.

### Steps
1. Lauching virtual server (EC2) on AWS 

2. Back-end configuration
 * Node.js Installation
 * Application Code Setup
 * ExpressJs Installation & Creation of Route Folder
 * Models creation

3. Database configuration
 
4. Front-end configuration
 * Running a React App

## Step 1 – Launching A Virtual Server (EC2) on AWS 

* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user with administrative privilege on the root account for security best practices
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

### Connecting to the EC2 terminal
* change directory to the directory wherein your key pair is located;

` cd ~/Downloads `

* Change premissions for the private key file (.pem),

` sudo chmod 0400 <private-key-name>.pem `

* ssh into the ec2 terminal using the bash shell.

` ssh *i <private-key-name>.pem ubuntu@<Public-IP-address> `

* Use the public IP on the instance for remote login on the Linux server.
* For remote server login, I used linux bash.

## Step 2 – Back-End Configuration
### Installation of Node

* update a list of packages in package manager

> ` sudo apt update ; sudo apt upgrade -y `

* Add certificate using this code:

```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

* For installation of NodeJS:

> ` sudo apt install -y nodejs `

!Diag.....

* verification of node installation:

> ` node *v ` 

## Step 3 – Database Configuration
### MongoDB Installation

* MongoDB is a NoSQL database program that store data in the form of JSON-like

* Input the following code:

> ` sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6`

* To install MongoDB, I input the code below:

> ` sudo apt install -y mongodb`

* To start the server:

> ` sudo service mongodb start `

* Verify that  server is running

> ` sudo systemctl status mongodb `

![Screenshot from 2022-10-13 17-25-59](https://user-images.githubusercontent.com/114786664/196739051-082978a1-0f64-4bb7-be14-4f7ac5de449a.png)

* Next is to install *npm* which is a package manager for node

> ` sudo apt install -y npm`

* then, a body-parser package will be installed. This help in processing JSON files passed in request to the server

> ` sudo npm install body-parser ` 

* Create a *Books* directory, cd into it, and intialized npm project with the following code:

> ` mkdir Books ; cd $_ ; npm init `

* After initializing, create a *server.js* file. 

> ` vi server.js`

* Paste, save and quit the vim editor.

### ExpressJs Installation & Creation of Route Folder

* We'll be making use of mongoose which provide a schema based solution for modelling application server. 

* Install express mongoose using:

> ` sudo npm install express mongoose `

* Next is to create a folder named *apps*,  cd into it, and create a file named *route.js* in a test editor.

` mkdir apps && cd $_ && vi route.js `

* Paste the code, save, and quit the editor.

* Still in the *apps* folder, create another folder named *models*, cd into it, and create a file named *book.js* with a text editor

` mkdir models && cd models && vi book.js `

* Paste, save, and exit the text editor.

## Step 4 – Access the routes with AngularJS

* Change the directory back to ‘Books’: 

`cd ../.. `

* Create a folder named *public*, and open a *script.js* file 

* Paste, save, and exit the text editor.

* In public folder, create a file named index.html;

> `vi index.html `

* Paste, save, and exit the text editor.

* Change the directory back up to Books

* Start the server by running this command:

> ` node server.js `

* The server is now running, and can be connected to at port 3300

* The Book web application can be access from the web browser using the public ip address.

![Screenshot from 2022-10-13 17-53-21](https://user-images.githubusercontent.com/114786664/196739470-6e36927f-5763-4823-9188-9c4e34889591.png)

![Screenshot from 2022-10-13 18-04-41](https://user-images.githubusercontent.com/114786664/196739541-3d1de5c1-f8b9-42bc-8ebf-e67a96ba98d3.png)

![Screenshot from 2022-10-13 18-05-27](https://user-images.githubusercontent.com/114786664/196739943-e56d5010-dc90-473f-9301-b075c1bc8ccc.png)

