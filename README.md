# express-mongo-lab
Lab9


Install MongoDb locally
Time to get into some document db stuff. First of all download MongoDb server from here.

If you haven't change anything during install, it should also install a thing called MongoDb Compass Community.

This is a great tool to inspect, change, add or remove data in collections in MongoDb. You can connect to the local instance by using default address and port like on this image below or connect to any other server.



To connect locally just press connect. Inside you can see some default collections and you can play around. We will need MongoDb Compass a bit later.

2. Connect to MongoDb through Express app
In this tutorial I will be using my favorite editor Visual Studio Code. You will also need Nodejs and Docker installed. In my case I’m using Windows, so I got Docker for Windows from here.

Now run following command:

```
mkdir test-mongo-app && cd test-mongo-app && npm init -y && code .
```

Time to install dependencies. We will need express and mongoose packages.
npm i express mongoose
Create file called server.js inside root folder.

Also don't forget to change your package.json to run server.js file at start.
```
{
  "name": "test-mongo-app",
  "version": "1.0.0",
  "description": "",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1",
    "mongoose": "^5.6.1"
  }
}
```

Good. Let's create a basic express app with two routes. One for reading Users from the database and second is for adding dummy user data to it.

First of all check if everything works with just express server.
```
// server.js
const express = require("express");
const app = express();

const PORT = 8080;

app.get("/", (req, res) => {
  res.send("Hello from Node.js app \n");
});

app.listen(PORT, function() {
  console.log(`Listening on ${PORT}`);
});
```


You can run npm start to test it. If you see the message "Listening on 8080" everything is ok. Also open http://localhost:8080 and check if you can see the hello message.

There is a nice thing called nodemon. It auto rebuilds our project when any changes happened in source code. Let's use it! 😀
npm install --save-dev nodemon
Add a new command in package.json. So we use it for development.
```
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
```
Now use run npm run dev while development instead of npm start.
```
npm run dev
```
You will notice difference in the console, because now nodemon is watching for any changes in your project and if needs, rebuild it. Change something in server.js and you will notice 😉

Now create folder src in root of the project. Here we will add all the rest files.

Let's create a User model for mongoose. Create file names User.model.js
```
// User.model.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  username: {
    type: String
  }
});

const User = mongoose.model("User", userSchema);

module.exports = User;
```
Good! Here we defined a model for our document db. User model has only one field username which is a string. Enough for now :)

Let's add a file called connection.js for connection to the database.
```
// connection.js
const mongoose = require("mongoose");
const User = require("./User.model");

const connection = "mongodb://localhost:27017/mongo-test";

const connectDb = () => {
  return mongoose.connect(connection);
};

module.exports = connectDb;
```
Please notice that mongo-test will be the name of our database (cluster).

Now modify a bit server.js and start the app. You should see message in the console that MongoDb is connected.
```
// server.js
const express = require("express");
const app = express();
const connectDb = require("./src/connection");

const PORT = 8080;

app.get("/users", (req, res) => {
  res.send("Get users \n");
});

app.get("/user-create", (req, res) => {
  res.send("User created \n");
});

app.listen(PORT, function() {
  console.log(`Listening on ${PORT}`);

  connectDb().then(() => {
    console.log("MongoDb connected");
  });
});

```
Yeah! 🎉 We connected Express app with local MongoDb instance!

3. Implement read and write to MongoDb
We should implement two routes for reading and adding new users.
Open server.js file and first of all import our model on the top:
```
// server.js
const User = require("./src/User.model");
// ...
Then implement both routes below like this:
// server.js
app.get("/users", async (req, res) => {
  const users = await User.find();

  res.json(users);
});

app.get("/user-create", async (req, res) => {
  const user = new User({ username: "userTest" });

  await user.save().then(() => console.log("User created"));

  res.send("User created \n");
});
// ...
```
Be attentive here we are using async/await pattern. If you are curious about this find it here.

Basically we implemented two routes /users and /user-create. ✋ Yeah, yeah, I know that create should be done through the POST http verb but just to make testing easier and escape configuring seed method for db.

Now it's time to test! 🔍 Open in browser this link http://localhost:8080/user-create to create a dummy user record in db. Open this link http://localhost:8080/users to get all users as JSON in browser.
