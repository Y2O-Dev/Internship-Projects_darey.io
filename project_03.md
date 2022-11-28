# Web Stack (MERN) Implementation in AWS
## SIMPLE TODO APPLICATION ON MERN WEB STACK

MERN is a variation of the MEAN stack (MomgoDB; Express; Angular and Node), but in this case the Angular framework is replaced by React.js. Hence,  the MERN stack is an acronym for he following combination:

* MongoDB
* Express.js
* React.js
* Node.js

Because all its components are open source, it allow for easy construction of a three tier architecture using JSON and JavaScript comprising of: database (MongoDB), front*end (React.js), and back*end/application (Express & Node).

### Steps
1. Lauching virtual server (EC2) on AWS 

2. Back-end configuration
 * Node.js Installation
 * Application Code Setup
 * ExpressJs Installation & Creation of Route Folder
 * Models creation

3. Database configuration

4. Testing Back-end code with RESTful API 
 
5. Front-end configuration
 * Running a React App

## Step 1 – Launching A Virtual Server (EC2) on AWS 

* Create an [AWS](https://aws.amazon.com) account.
* Create an IAM user with administrative privilege on the root account for security best practices
* Launch a new EC2 instance of t2.micro family with Ubuntu Server 20.04 LTS (HVM)

### Connecting to the EC2 terminal
* change directory to the directory wherein your key pair is located;

` cd ~/Downloads `

* Change premissions for the private key file (.pem),

` sudo chmod 0400 <private keyname>.pem `

* ssh into the ec2 terminal using the bash shell.

` ssh -i <private keyname>.pem ubuntu@<Public-IP address> `

* Use the public IP on the instance for remote login on the Linux server.
* For remote server login, I used linux bash.

## Step 2 – Back-End Configuration
### Installation of Node

* update a list of packages in package manager

>` sudo apt update ; sudo apt upgrade -y `

* To get the location of the Node.js on Ubuntu repository:

>` curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - `

* For installation:

>` sudo apt install -y nodejs `

![1_prj3_node](https://user-images.githubusercontent.com/114786664/196696019-f1bad1db-c187-4e41-a4e0-638c1a04182c.png)

* verification of node installation:

>` node -v ` 

### Application Code Setup

* Create a new folder that will contain all setup for the project, and change directory into the newly created folder using the following command:

>` mkdir Todo && cd $_`

* The next action  is to initialized the project. This will create a *package.json* file which will contain information about the application and its dependecies needed to run.

>` npm init`

![2_proj3](https://user-images.githubusercontent.com/114786664/196696684-09d37bda-8368-4dfd-80b5-41cd4448bd6a.png)

* To check for the presence of the package.json file using: >` ls -l ` 

### ExpressJs Installation & Creation of Route Folder

* Installation is done using the below code:

> ` npm install express `

* Next is to create a file named *index.js*: ` touch index.js; ls`

* Istallation of dotenv moodule: ` npm install dotenv `

* opening of the index.js file in a text editor, and pasting the code below: `vi index.js `

```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access*Control*Allow*Origin", "\*");
res.header("Access*Control*Allow*Headers", "Origin, X*Requested*With, Content*Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```
* since we're using Vim editor, save and quit using **:qa**

* Next is starting the server on port:5000: `public ip:5000 `

* The url will only go through after modification of the EC2 instance security group

![4_prj3](https://user-images.githubusercontent.com/114786664/196697536-6382339a-0724-4106-b88e-17a2fc20f12b.png)

![5_proj3](https://user-images.githubusercontent.com/114786664/196697729-62a4d4eb-5733-4ccc-b5c8-a6872e27ebee.png)

> **Routes Directory Creation**

* Create a routes directory, cd into it and create an *api.js* file:

>` mkdir routes && cd $_ && vi api.js`

* The below code should be paste into the file

```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

**Models Creation**

* Change dir back to the Todo dir. ` cd ..`

* We'll have to install mongoose so as to create a schema for our DB:

* ` npm install mongoose `

* Create a models directory, cd into it, and open a *todo.js* file in the text editor: 

` mkdir models && cd $_ && vi todo.js `

```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```

* Save the file and quit the text editor with *:wq*

* Next is to update the *api.js* file in routes directory. Open the file using

` vi api.js `

* Delete the existing code using %d shortcut, and paste the below code:

```
const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```

* Save and quit the editor.

### Database Configuration

**MongoDB Database**

* To store data on our site, database (MongoDB) is needed. No need of installation but through MLab that provides MongoDB DBaaS solution.

* We'll need to sign up at [MongoDB](https://www.mongodb.com/atlas-signup-from-mlab) for a cluster free account.

* Next is to create a MongoDB database and collection inside the mLab

* in the *index.js* file, *process.env* was specified to access environmental variable, but yet to create the file. To create the file,

` vi .env `

* The below connection string should be paste in the file. This allow access to the DB.

` DB = 'mongodb+srv://<username>:<password>@<network*address>/<dbname>?retryWrites=true&w=majority' `

* Save and quit vim

* The username, password, network-address, and database should be changed accordingly. 

* Next is to modify the *index.js* file by deleting the existing code and pasting the old and new code respectively. This is to reflect the use of *.env*, and give Node a connection access to the DB.

` vi  index.js` to open the file

* _:%d_ to delete

*  Paste the new code, save and quit vim.

* Start server using the command:

` node index.js `

![6_prj3](https://user-images.githubusercontent.com/114786664/196698821-652e7157-1097-4353-be0a-ef6538489f38.png)

### Testing Backend Code using RESTful API
  
This is required to test the back*end code prior to configuration of the front*end UI. This will give us assurance that our back*end code are functioning properly.

* **Postman** will be used to test the API using CRUD operations

![7b_prjt3](https://user-images.githubusercontent.com/114786664/196700123-d87950a3-e672-4a69-b54e-5ff767fd07e2.png)

![7c_proj3](https://user-images.githubusercontent.com/114786664/196700160-b23e138f-a117-4641-966c-96cbc28edf31.png)

## Front-end configuration

* At this point, its time to create a user interphase that will interract with the back-end application through an API. The installation will be at the Todo dir unless otherwise stated.

* Starting the front-end Todo app by first creating a React client which will scaffold the app and create a directory that will contain all the react code. 

` npx create-react-app client `

### Running a React App

* Prior to running the React app, some dependencies need be installed:

> **Concurrently**: To run more than one command at the same time from a single terminal window

` npm install concurrently --save-dev `

> **Nodemon**: To run and monitor the server

` npm install nodemon --save-dev `

> **Modification** of the *package.json* file in the Todo folder

### Configure Proxy in *package.json* file in the clients directory

* cd to client and open the *package.json* file in vim:

` cd clients && vi package.json `

* Add the key value pair in the *package.json* file *"proxy": "http://localhost:5000"*

* cd back to the Todo directory and run the following code:

` npm run dev `

* The app should open and start running on localhost:3000
  
![8b_proj3](https://user-images.githubusercontent.com/114786664/196702057-72af7427-54c9-46b5-9479-30c161dd2f25.png)

![8_proj3](https://user-images.githubusercontent.com/114786664/196702202-350bb072-0f4d-4f38-8d9c-72a9842861d8.png)

### Creating a React Components

* cd from *Todo*  to *clients* and subsequently *src* folder.

* Inside *src*, create another folder named *components* and cd into it.

* Inside components directory create three files *Input.js*, *ListTodo.js* and *Todo.js*.

* Open the *Input.js* file

* All these can be done in a single line of command.

``` 
cd Todo/clients/src && mkdir components && cd $_ 
&& touch Input.js ListTodo.js Todo.js && vi Input.js
```

* Paste the new code into the opened file, save and quit the editor

* Change directory back to the clients folder:

` cd ../..` 

* At the current working directory, install Axios:

` npm install axios ` 

* cd back to *components* folder, open the *ListTodo.js* file and paste the code:

` cd src/components ; vi ListTodo.js `

* Next is to open the *Todo.js* file, and paste the code

* Adjustment need be made to the App.js code. To do this, cd into the *src* folder:

` cd .. && vi App.js`
*  Paste the code, save , and quit the editor.

* Also open the App.css file, paste the css code, save, and quit the editor.

* Still in the *src* directory, open the *index.css* file, paste the code, save, and exit the vim editor.

* Finally go back to the Todo folder with this relative path:
` cd ../.. `

* Run the below code:
` npm run dev `

* Assuming all instructions followed to the letter, the front*end webpage should be fully functional.

![9_proj3_final](https://user-images.githubusercontent.com/114786664/196709270-f04c863d-666e-4c30-a2b4-7323eaba7632.png)
