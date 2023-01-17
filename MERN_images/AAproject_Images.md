# SIMPLE TO-DO APPLICATION ON MERN WEB STACK

###  Implement a web solution based on MERN (MongoDB ExpressJS ReactJS NodeJS) stack in AWS Cloud

**1. MongoDB:** A document-based, No-SQL database used to store application data in a form of documents.

**2. ExpressJS:** A server side Web Application framework for Node.js.

**3. ReactJS:** A frontend framework developed by Facebook. It is based on JavaScript, used to build User Interface (UI) components.

**4. Node.js:** A JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser.

### Schematics of the stack

![Schematics of the stack](mern_schematic.PNG)

*As shown above, a user interacts with the ReactJS UI components at the application front-end residing in the browser. This frontend is served by the application backend residing in a server, through ExpressJS running on top of NodeJS.
Any interaction that causes a data change request is sent to the NodeJS based Express server, which grabs data from the MongoDB database if required, and returns the data to the frontend of the application, which is then presented to the user.*

Deployed EC2 Instance of t2.nano family with Ubuntu Server 20.04 LTS (HVM) image.

**STEP 1: BACKEND CONFIGURATION** 

*Update ubuntu*

`sudo apt update`
![SS for update](image01.PNG)

*Upgrade ubuntu*

`sudo apt upgrade`
![SS for upgare](image02.PNG)

*Got the location of [Node.js](https://github.com/nodesource/distributions#deb) software*

`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`
![SS for to grab NodeJs](image03.PNG)

*Install Node.js with the command below. **Note:** The command installs both nodejs and npm. **NPM** is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.*

`sudo apt-get install -y nodejs`

*Verify the node installation with the command below*

`node --version`

Verify the npm installation with the command below

`npm -version`

*Create a new directory for your To-Do project*

`mkdir Todo`

***TIP:** In order to see some more useful information about files and directories, you can use following combination of keys `ls -lih` – it will show you different properties and size in human readable format. You can learn more about different useful keys for `ls` command with `ls --help`.*

*Navigate  into Todo directory with below command*

`cd Todo`

*Next, you will use the command `npm init` to initialise your project, so that a new file named package.json will be created. This file will normally contain information about your application and the dependencies that it needs to run. Follow the prompts after running the command. You can press **Enter** several times to accept default values, then accept to write out the package.json file by typing **yes***

`npm init`

![SS for NodeJs installation, Todo directory creation and initialising npm](image04.PNG)

*Run the command `ls` to confirm that you have package.json file created*

**STEP 2: INSTALL EXPRESSJS**

*Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. Therefore it simplifies development, and abstracts a lot of low level details. For example, Express helps to define routes of your application based on HTTP methods and URLs.*

*Install Express using npm*

`npm install express`

*Create a file index.js with the command below*

`touch index.js`

*Run `ls` to confirm that your index.js file is successfully created*

*Install the dotenv module*

`npm install dotenv`

![SS for ExpressJs installation, and dotenv](image05.PNG)

*Open the index.js file with the command below*

`vi index.js`

*Type the code below into it and save ( `:w` to save in vim and use `:qa` to exit vim)*
```
const express = require('express');
require('dotenv').config();
const app = express();
const port = process.env.PORT || 5000;
app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});
app.use((req, res, next) => {
res.send('Welcome to Express');
});
app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```
*Start server to see if it works. Open your terminal in the same directory as your index.js file and type. If every thing goes well, you should see Server running on port 5000 in your terminal.*

`node index.js`

*Open this port in EC2 Security Groups. Created an inbound rule to open TCP port 80, you need to do the same for port 5000*

*Open up your browser and try to access your server’s Public IP or Public DNS name followed by port 5000*
![access server via public IP and opened port number](Espress.PNG)

![SS to start server](image06.PNG)



#### There are three actions that our To-Do application needs to be able to do:
```
  1. Create a new task
	2. Display list of all tasks
	3. Delete a completed task
```
#### Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.
#### For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes

***Tip:** While starting server, you can open multiple shells in Putty or Linux/Mac to connect to the same EC2*

*Navigate into the Todo directory and also create directory route and access the it*
`cd Todo`

`mkdir routes && cd routes`

*Now, create a file api.js with the command below*

`touch api.js`

*Open the file with the command below*

`vi api.js`

*Copy below code in the file (api.js)*

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
**MODELS**
#### Since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model. A model is at the heart of JavaScript based applications, and it is what makes it interactive. We will also use models to define the database schema . This is important so that we will be able to define the fields stored in each Mongodb document. Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties. To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.

*Change directory back Todo folder with cd .. and install Mongoose*

`npm install mongoose`

*Create a new folder models*

`mkdir models`

*Change directory into the newly created ‘models’ folder with*

`cd models`

*Inside the models folder, create a file and name it todo.js*

`touch todo.js`

*Open the file created with vim todo.js then paste the code below in the file*

`vi todo.js`

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

![SS for previous steps](image07.PNG)

*Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model. In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit*

`cd routes && vi api.js`

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
![SS for routes, api](image088.PNG)

**MONGODB DATABASE**
#### We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS), so to make life easy, you will need to sign up for a shared clusters free account, which is ideal for our use case. [Sign up](https://www.mongodb.com/atlas-signup-from-mlab) here. Follow the process, select AWS as the cloud provider, and choose a region near you. Complete a get started checklist as shown on the image below

![SS for mongodb](mongo01.PNG)

*Allow access to the MongoDB database from anywhere (Not secure, but it is ideal for testing). As in the image below, make sure you change the time of deleting the entry from 6 Hours to 1 Week*

![SS for mongodb](mongo02.PNG)

*Create a MongoDB database and collection inside mLab*

![SS for mongodb](mongo03.PNG)

![SS for mongodb](mongo04.PNG)

*In the index.js file, we specified process.env to access environment variables, but we have not yet created this file. So we need to do that now. Create a file in your Todo directory and name it .env*

`touch .env && vi env`

*Add the connection string to access the database in it, just as below. Ensure to update `<username>, <password>, <network-address>` and `<database>` according to your setup*

DB = 'mongodb+srv://:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'

![SS for env](image088.PNG)

*Here is how to get your connection string*

![SS for mongodb](mongo05.PNG)

![SS for mongodb](mongo06.PNG)

![SS for mongodb](mongo07.PNG)

*update the index.js to reflect the use of .env so that Node.js can connect to the database. Simply delete existing content in the file, and update it with the entire code below. Use `%d` to delete content from vim editor *

`vim index.js`

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();
const app = express();
const port = process.env.PORT || 5000;
//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));
//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;
app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});
app.use(bodyParser.json());
app.use('/api', routes);
app.use((err, req, res, next) => {
console.log(err);
next();
});
app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```
![SS for NodeJs installation, Todo directory creation and initialising npm](image099.PNG)

*Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file*

*Start your server using the command*

`node index.js`

![SS to start node](image010.PNG)

![SS to start node](image011.PNG)

*You should see a message ‘Database connected successfully’, which reveals that backend is configured. Now we are going to test it*

*Testing Backend Code without Frontend using RESTful API*

#### So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code.
#### Download and install postman on your machine. You should test all the API endpoints and make sure they are working. For the endpoints that require body, you should send JSON back with the necessary fields since it’s what we setup in our code. Open your Postman, 
#### Create a POST request to the API `http://<PublicIP-or-PublicDNS>:5000/api/todos`. This request sends a new task to our To-Do list so the application could store it in the database.
#### Note: make sure your set header key Content-Type as application/json
#### Create a GET request to your API on http://<PublicIP-or-PublicDNS>:5000/api/todos. This request retrieves all existing records from out To-do application (backend requests these records from the database and sends it us back as a response to GET request).

![SS for postman](postman1.PNG)

![SS for postman](postman1a.PNG)

![SS for postman](postman2.PNG)

**FRONTEND CREATION**

#### Create a user interface for a Web client (browser) to interact with the application via API. To start out with the frontend of the To-do app, we will use the create-react-app command to scaffold our app

*In the same root directory as your backend code, which is the Todo directory, run below command as it will create a new folder in your Todo directory called client, where we will add all the react cod*

`npx create-react-app client`

![SS to start server](image06.PNG)

*Before testing the react app, there are some dependencies that need to be installed*

`npm install concurrently --save-dev`

*Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.*

`npm install nodemon --save-dev`

*In Todo folder open the package.json file. Change the highlighted part of the below screenshot and replace with the code below*

```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
![SS to start node](todo01.PNG)

*Change directory to ‘client’ and open package.json via vim*

`cd client && vi package.json`

*Add the key value pair in the package.json file `"proxy": "http://localhost:5000`"*

*The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like `http://localhost:5000` rather than always including the entire path like `http://localhost:5000/api/todos`. Now, ensure you are inside the Todo directory, and simply do*

`npm run dev`

![SS to start node](image012.PNG)

*Now app should open and start running on `localhost:3000`. In order to be able to access the application from the Internet you have to open TCP port 3000 on EC2 by adding a new Security Group rule*

![SS to start node](image013.PNG)

![SS to start node](image014.PNG)

*One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component. From your Todo directory run*

`cd client`

*m
*ove to the src directory*

`cd src`

*Inside your src folder create another folder called components*

`mkdir components`

*Move into the components directory with*

`cd components`

*Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js.*

`touch Input.js ListTodo.js Todo.js`

*Open Input.js file and past the below code into it then save*

`vi Input.js`

```
import React, { Component } from 'react';
import axios from 'axios';
class Input extends Component {
state = {
action: ""
}
addTodo = () => {
const task = {action: this.state.action}
if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }
}
handleChange = (e) => {
this.setState({
action: e.target.value
})
}
render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}
export default Input
```

*To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios. Move to the src folder*

`npm install axios`

![SS to start node](image015.PNG)

![SS to start node](image016.PNG)

**FRONTEND CREATION (CONTINUED)**

*Go to ‘components’ directory and open ListTodo.js file,*

`cd src/components`

`vi ListTodo.js`

*Then paste the following code then save*

```
import React from 'react';
const ListTodo = ({ todos, deleteTodo }) => {
return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}
export default ListTodo
```
![SS to start node](image017.PNG)

`vi Todo.js`

*Then in your Todo.js file you write the following code then save*

```
import React, {Component} from 'react';
import axios from 'axios';
import Input from './Input';
import ListTodo from './ListTodo';
class Todo extends Component {
state = {
todos: []
}
componentDidMount(){
this.getTodos();
}
getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}
deleteTodo = (id) => {
axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))
}
render() {
let { todos } = this.state;
return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )
}
}
export default Todo;
```

*Some little adjustment is required on the react code. Delete the logo and adjust our App.js to look like this. Move to the src folder and Make sure that you are in the src folder and run*

`vi App.js`

*Paste below code into it then save it*

![SS to start node](image018.PNG)

```
import React from 'react';
import Todo from './components/Todo';
import './App.css';
const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}
export default App;
```

*In the src directory open the App.css*

`vi App.css`

*Paste below code into App.css*

```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}
input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}
input:focus {
outline: none;
}
button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}
button:focus {
outline: none;
}
ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}
li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}
@media only screen and (min-width: 300px) {
.App {
width: 80%;
}
input {
width: 100%
}
button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}
@media only screen and (min-width: 640px) {
.App {
width: 60%;
}
input {
width: 50%;
}
button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```
*In the src directory open the index.css*

`vim index.css`

*Paste below code into it and save*

```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}
code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```


*Go to the Todo directory and when in the directory run below command*

`npm run dev`

![SS to start node](image019.PNG)

*Assuming no errors when saving all these files, our To-Do app should be ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all your tasks.*

![SS to start node](todo.PNG)

*Assuming no errors when saving all these files, our To-Do app should be ready and fully functional with the functionality discussed earlier: creating a task, deleting a task and viewing all your tasks.*

![SS to start node](React1.PNG)
