# **To-Do Web Application on MERN Web Stack**

This Project covers the implementation of developing a To-Do app using the MERN Stack from start to finish and deploying to **AWS** (Amazon Web Services). The MERN Stack is a Technology Stack. A Technology stack comprises of framework/tools that are used in building a software application. The MERN Acronym stands for MongoDB, ExpressJS, React, Node.

- MongoDB - MongoDB is a document-based NoSQL database that stores data in form of documents. It allows for storage of data in semi-structured and unstuctured formats.

- ExpressJS - ExpessJS is a server side web framework that is built for NodeJS. It comes with a lot of features out of the box and it enables the development process to be more faster and efficient.

- React - React is a Front-end Framework/Library built with Javascript for building user interfaces or client-facing apps.

- NodeJS - NodeJS is a javascript runtime that is built on the v8 engine, it enables you to run javascript on your local machine rather than on the browser.

## **Create an EC2 instance on your AWS Dashboard**

You have to create a Linux virtual server on AWS. There are several features and services that AWS offer but for this project, I'll be using the ec2 instance. If you're using windows, you have to download a CLI (Command Line Interface) such as Git, Putty, Termius, MobaXterm that is Linux-enabled so you can run basic linux commands. I watched this [video](https://www.youtube.com/watch?v=xxKuB9kJoYM&list=PLtPuNR8I4TvkwU7Zu0l0G_uwtSUXLckvh&index=7) to create a Linux virtual server, also i need to add that the Linux distro we're using is Ubuntu.

When we're done creating the server, we first have to add permissions to our private key using this command.

![1](https://user-images.githubusercontent.com/47898882/125817420-2a9b775b-eea9-4ef8-822d-84283e0e30b6.JPG)

If that works out successfully, you can then connect to the server using the ssh command.

![2](https://user-images.githubusercontent.com/47898882/125817444-b289df98-98e1-4089-989c-37bea1a79429.JPG)

Now that our Virtual Server is up and running we can start building our application.

# **Backend configuration**

- Update and Upgrade the _apt_ package manager in order to have the newest updated version

```
$ sudo apt update
$ sudo apt upgrade
```

## **Install NodeJS on Server**

- Download the location of the NodeJS software on ubuntu's repository. It'll be stored in the `etc/apt/sources.list.d` directory.

```
$ curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

- Install NodeJS using the _apt_ command

```
$ sudo apt install nodejs -y
```

- Verify the version of Node installed using `node -v`

```
$ node -v
```

## **Application Code Setup**

- Create a Directory using the called Todo using the _mkdir_ command and enter into the folder using the _cd_ command

```
$ mkdir Todo
$ cd Todo
```

- In that directory, initialize your project with `git init`, this will create a file called _package.json_ which will house all the global and development dependencies that would be installed. Follow all of the prompts until the end.

```
$ npm init
```

## **Install ExpressJS & Other Dependencies**

- Install Express using the _npm_ command. NPM known as Node Package Manager is used for installing software on NodeJs. This package manager houses all of the dependencies that enable NodeJS to function.

```
$ npm install express
```

- Create a File called index.js in that same directory using the `touch` command

```
$ touch index.js
```

- Install another dependency known as `dotenv`. Dotenv is a package used for creating environmental variables. It also adds a layer of security, so important information can be hidden.

```
$ npm install dotenv
```

- Use th `vi` command to enter into the index.js file and write into the file. _vi_ also known as vim, it is a text editor in linux for writing code. The configuration for the virtual host will be written as below. You can use _i_ to insert, :_wq_ to write(save) into the file and quit.

```
$ const express = require('express');
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

- Port has been set to `5000` in the index.js file. This is the port the server is going to run on. It can be can be tested on using the `node index.js` command

![3](https://user-images.githubusercontent.com/47898882/125827128-ea13b2b1-dd12-438e-ae9b-f71cf9c5ac31.JPG)

- Before we can test on the browser, the security group for our virtual server has to be updated to allow connections into port 5000. To achieve this, edit inbound connections and add port 5000 as one of the ports where http connection is allowed from.

![4](https://user-images.githubusercontent.com/47898882/125827838-ca6fbae6-6483-4629-ad58-4b9b08e77f6a.JPG)

- Test on our browser using the public IP address or DNS name. `http://<Public-IP-Address>:5000 or http://DNS-name:5000`. "Welcome to Express" should be seen on the screen.

## **Create Routes(Endpoints)**

- Create http endpoints for the app that allows creation of a new task(POST), displaying all tasks(GET) and deleting all tasks(DELETE). To achieve this, create another folder called `routes`.

```
$ mkdir routes
```

- Create a file and name it api.js

```
$ touch api.js
```

- Use the _vi_ command to enter into the file and write into the file

```
$ const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

## **Create Models (Database Creation)**

- To create the database, MongoDB is the NoSQL database we'll be using. It stores data in form of documents. `mongoose` is a package built for mongodb that makes working with mongodb easier. Install `mongoose` uisng the npm command

```
$ npm install mongoose
```

- Create a folder called _models_ and add a file into it called _todo.js_

```
$ mkdir models && cd models && touch todo.js
```

- Use th `vi` command to write into the todo.js file

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

- Update the api.js file to make use of the newly created model

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

## **Create Database in the Cloud**

- Thankfully, there's an offering by MongoDB Atlas that allows us to spin up new clusters and create databases for our applications. To achieve this create an account on MongoDB Atlas, Sign up and create a new cluster. Follow the follwoing steps below

![5](https://user-images.githubusercontent.com/47898882/125833414-8c3607ba-4ebe-4dc5-81ac-e1d3ac7cc12d.JPG)

![6](https://user-images.githubusercontent.com/47898882/125833416-332c511c-a09a-4c7c-a5cf-2c84322babcf.JPG)

![7](https://user-images.githubusercontent.com/47898882/125833408-77930f52-201d-4701-a3ab-b9a25667afc0.JPG)

- Click on the connect button above to connect your newly created cluster. Then connect to your application.

![8](https://user-images.githubusercontent.com/47898882/125834142-7c0fb4b0-7e23-4e1b-8ee6-c4a2e9a05d43.JPG)

- Copy the conncetion string as we'll be using it in our code. Make sure you remember the `password` you used in creating the cluster, also specify a database name when pasting the connection string in your code

![9](https://user-images.githubusercontent.com/47898882/125834145-02720d5f-f15d-4583-8eb6-fb5fed5c0416.JPG)

- Create a file in the Todo directory called .env and use _vi_ to write into the file

```
$ touch .env
$ vi .env
```

- Paste the connection sting into that file and give it a vaariable name called `DB`. That is the name we'll reference when updating the index.js file

```
DB = mongodb+srv://rotimi:<password>@cluster0.ynr2h.mongodb.net/myFirstDatabase?retryWrites=true&w=majority
```

- Update Index.js to make use of the newly created `DB` environmental variable.

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

- In our code we connect to the database by referencing the .env file using `process.env.DB`. Run `node index.js` and you should see **Database connected successfully**

![10](https://user-images.githubusercontent.com/47898882/125835753-453d46e0-9c0c-460b-880f-0b567c655554.JPG)

## **Test Endpoints using Postman**

- Our Database has been created and we've inserted our connection string in our code, so mongoose can connect to the database. Let us test using postman if we can add, display tasks from the database.

**POST REQUEST**
![11](https://user-images.githubusercontent.com/47898882/125837623-f6a22eea-641e-4d71-bd63-fbd63dbf99e6.JPG)

**GET REQUEST**
![12](https://user-images.githubusercontent.com/47898882/125837620-bd620868-40a8-4eb3-85b7-2bcdf7101067.JPG)

# **Create Frontend Using React**

- To get started with react run `npx create-react-app client ` in the Todo directory

- Install both _concurrently_, _nodemon_ & _axios_ as they are dependencies needed for the app to run. Concurrently allows us to run more than one command at a time on the terminal while nodemon allows changes to be made to the application without restarting the server. Axios on the other hand will help us consume our REST API we created in the backend section.

```
$ npm install concurrently nodemon axios
```

- In the Todo directory, open the package.json folder and update.

```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```

![13](https://user-images.githubusercontent.com/47898882/125838791-89f27a3c-1622-4625-9202-02fa76b68b80.JPG)

- Change directory into client and update the package.json file to have the key value pair `"proxy": "http://localhost:5000"`

- Go back to the the Todo directory and run the `npm run dev `command.

```
$ npm run dev
```

**Note** - You have to configure your outbound rules to allow connection to port 3000 as react by default uses this port for testing.

## **Creating your React Components**

- Change Directory into _client_, and then into _src_ and create a directory called _components_, enter into that directory and create three files `Todo.js`, `ListTodo.js` & `Input.js`

```
$ cd client && cd src && mkdir components && touch Todo.js ListTodo.js Input.js
```

- Input these pieces of code into the files.

**Input.js**

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

**ListTodo.js**

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

**Todo.js**

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

- Change directory back to the `src` and make changes to the `App.js`, `App.css` & `index.css` files

**App.js**

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

**App.css**

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

**index.css**

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

- Change back to the Todo directory and run the command `npm run dev`. You should see have your application running with a User Interface:

![14](https://user-images.githubusercontent.com/47898882/125841056-2465e6bb-6264-4a07-9e3b-3ab895a023c2.JPG)

- Add more Tasks to the todo list at your own discretion.

**This is how to implement a MERN Stack Application and deploy to AWS EC2 from start to finish**.
