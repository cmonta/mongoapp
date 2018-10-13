# Build a Simple Beginner App with Node, Bootstrap & MongoDB

[by James Hibbard](https://www.sitepoint.com/build-simple-beginner-app-node-bootstrap-mongodb/)

If you’re just getting started with Node.js and want to try your hand at building a web app, things can often get a little overwhelming. Once you get beyond the “Hello, World!” tutorials, much of the material out there has you copy-pasting code, with little or no explanation as to what you’re doing or why.

This means that, by the time you’ve finished, you’ve built something nice and shiny, but you also have relatively few takeaways that you can apply to your next project.

In this tutorial, I’m going to take a slightly different approach. Starting from the ground up, I’ll demonstrate how to build a no-frills web app using Node.js, but instead of focusing on the end result, I’ll focus on a range of things you’re likely to encounter when building a real-world app. These include routing, templating, dealing with forms, interacting with a database and even basic authentication.

This won’t be a JavaScript 101. If that’s the kind of thing you’re after, look here. It will, however, be suitable for those people who feel reasonably confident with the JavaScript language, and who are looking to take their first steps in Node.js.

## What We’ll Be Building
We’ll be using Node.js and the Express framework to build a simple registration form with basic validation, which persists its data to a MongoDB database. We’ll add a view to list successful registration, which we’ll protect with basic HTTP authentication, and we’ll use Bootstrap to add some styling. The tutorial is structured so that you can follow along step by step. However, if you’d like to jump ahead and see the end result, the code for this tutorial is also available on GitHub.

## Basic Setup
Before we can start coding, we’ll need to get Node, npm and MongoDB installed on our machines. I won’t go into depth on the various installation instructions, but if you have any trouble getting set up, please leave a comment below, or visit our forums and ask for help there.

### Node.js
Many websites will recommend that you head to the official Node download page and grab the Node binaries for your system. While that works, I would suggest that you use a version manager instead. This is a program which allows you to install multiple versions of Node and switch between them at will. There are various advantages to using a version manager, for example it negates potential permission issues which would otherwise see you installing packages with admin rights.

If you fancy going the version manager route, please consult our quick tip: Install Multiple Versions of Node.js Using nvm. Otherwise, grab the correct binaries for your system from the link above and install those.

### npm
npm is a JavaScript package manager which comes bundled with Node, so no extra installation is necessary here. We’ll be making quite extensive use of npm throughout this tutorial, so if you’re in need of a refresher, please consult: A Beginner’s Guide to npm — the Node Package Manager.

### MongoDB
MongoDB is a document database which stores data in flexible, JSON-like documents.

The quickest way to get up and running with Mongo is to use a service such as mLabs. They have a free sandbox plan which provides a single database with 496 MB of storage running on a shared virtual machine. This is more than adequate for a simple app with a handful of users. If this sounds like the best option for you, please consult their quick start guide.

You can also install Mongo locally. To do this, please visit the official download page and download the correct version of the community server for your operating system. There’s a link to detailed, OS-specific installation instructions beneath every download link, which you can consult if you run into trouble.

#### Create a MongoDB container

```
docker run -p 27017:27017 -v $(pwd)/mongodb/db:/data/db --name my-mongo-dev -d mongo mongod
```

### A MongoDB GUI
Although not strictly necessary for following along with this tutorial, you might also like to install Compass, the official GUI for MongoDB. This tool helps you visualize and manipulate your data, allowing you to interact with documents with full CRUD functionality.

At the time of writing, you’ll need to fill out your details to download Compass, but you won’t need to create an account.

### Check that Everything Is Installed Correctly
To check that Node and npm are installed correctly, open your terminal and type:

```
node -v
```

followed by:

```
npm -v
```

This will output the version number of each program (8.9.4 and 5.6.0 respectively at the time of writing).

If you installed Mongo locally, you can check the version number using:

```
mongo --version
```

This should output a bunch of information, including the version number (3.6.2 at the time of writing).

### Check the Database Connection Using Compass
If you have installed Mongo locally, you start the server by typing the following command into a terminal:

```
mongod
```

Next, open Compass. You should be able to accept the defaults (server: localhost, port: 27017), press the CONNECT button, and establish a connection to the database server.

![MongoDB Compass connected to localhost:27107](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/01/1515662920mongo-compass.jpg "MongoDB Compass connected to localhost:27107")

*MongoDB Compass connected to localhost*

Note that the databases admin and local are created automatically.

### Using a Cloud-hosted Solution
If you’re using mLabs, create a database subscription (as described in their quick-start guide), then copy the connection details to the clipboard. This should be in the form:

```
mongodb://<dbuser>:<dbpassword>@ds251827.mlab.com:51827/<dbname>
```

When you open Compass, it will inform you that it has detected a MongoDB connection string and asks if you would like to use it to fill out the form. Click Yes, noting that you might need to adjust the username and password by hand. After that, click CONNECT and you should be off to the races.

![MongoDB Compass connected to mLabs](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/01/1515664188mongo-compass-mlabs.jpg "MongoDB Compass connected to mLabs")

*MongoDB Compass connected to mLabs*

Note that I called my database sp-node-article. You can call yours what you like.

## Initialize the Application

With everything set up correctly, the first thing we need to do is initialize our new project. To do this, create a folder named demo-node-app, enter that directory and type the following in a terminal:

```
npm init -y
```

This will create and auto-populate a package.json file in the project root. We can use this file to specify our dependencies and to create various npm scripts, which will aid our development workflow.

### Install Express
Express is a lightweight web application framework for Node.js, which provides us with a robust set of features for writing web apps. These features include such things as route handling, template engine integration and a middleware framework, which allows us to perform additional tasks on request and response objects. There is nothing you can do in Express that you couldn’t do in plain Node.js, but using Express means we don’t have to re-invent the wheel and reduces boilerplate.

So let’s install Express. To do this, run the following in your terminal:

```
npm install --save express
```

By passing the --save option to the npm install command, Express will be added to the dependencies section of the package.json file. This signals to anyone else running our code that Express is a package our app needs to function properly.

### Install nodemon
nodemon is a convenience tool. It will watch the files in the directory it was started in, and if it detects any changes, it will automatically restart your Node application (meaning you don’t have to). In contrast to Express, nodemon is not something the app requires to function properly (it just aids us with development), so install it using:

```
npm install --save-dev nodemon
```

This will add nodemon to the dev-dependencies section of the package.json file.

### Create Some Initial Files

We’re almost through with the setup. All we need to do now is create a couple of initial files before kicking off the app.

In the demo-node-app folder create an app.js file and a start.js file. Also create a routes folder, with an index.js file inside. After you’re done, things should look like this:

```
.
├── app.js
├── node_modules
│   └── ...
├── package.json
├── routes
│   └── index.js
└── start.js
```

Now, let’s add some code to those files.

In app.js:

```javascript
const express = require('express');
const routes = require('./routes/index');

const app = express();
app.use('/', routes);


module.exports = app;
```

Here, we’re importing both the express module and (the export value of) our routes file into the application. The require function we’re using to do this is a built-in Node function which imports an object from another file or module. If you’d like a refresher on importing and exporting modules, read Understanding module.exports and exports in Node.js.

After that, we’re creating a new Express app using the express function and assigning it to an app variable. We then tell the app that, whenever it receives a request from forward slash anything, it should use the routes file.

Finally, we export our app variable so that it can be imported and used in other files.

In start.js:

```javascript
const app = require('./app');

const server = app.listen(3000, () => {
  console.log(`Express is running on port ${server.address().port}`);
});
```

Here we’re importing the Express app we created in app.js (note that we can leave the .js off the file name in the require statement). We then tell our app to listen on port 3000 for incoming connections and output a message to the terminal to indicate that the server is running.

And in routes/index.js:

```javascript
const express = require('express');

const router = express.Router();

router.get('/', (req, res) => {
  res.send('It works!');
});

module.exports = router;
```

Here, we’re importing Express into our routes file and then grabbing the router from it. We then use the router to respond to any requests to the root URL (in this case http://localhost:3000) with an “It works!” message.

### Kick off the App
Finally, let’s add an npm script to make nodemon start watching our app. Change the scripts section of the package.json file to look like this:

```javascript
"scripts": {
  "watch": "nodemon ./start.js"
},
```

The scripts property of the package.json file is extremely useful, as it lets you specify arbitrary scripts to run in different scenarios. This means that you don’t have to repeatedly type out long-winded commands with a difficult-to-remember syntax. If you’d like to find out more about what npm scripts can do, read Give Grunt the Boot! A Guide to Using npm as a Build Tool.

Now, type npm run watch from the terminal and visit http://localhost:3000.

You should see “It works!”

### Basic Templating with Pug
Returning an inline response from within the route handler is all well and good, but it’s not very extensible, and this is where templating engines come in. As the Express docs state:

>A template engine enables you to use static template files in your application. At runtime, the template engine replaces variables in a template file with actual values, and transforms the template into an HTML file sent to the client.

In practice, this means we can define template files and tell our routes to use them instead of writing everything inline. Let’s do that now.

Create a folder named views and in that folder a file named form.pug. Add the following code to this new file:

```javascript
form(action="." method="POST")
  label(for="name") Name:
  input(
    type="text"
    id="name"
    name="name"
  )

  label(for="email") Email:
  input(
    type="email"
    id="email"
    name="email"
  )

  input(type="submit" value="Submit")
  ```

As you can deduce from the file ending, we’ll be using the pug templating engine in our app. Pug (formerly known as Jade) comes with its own indentation-sensitive syntax for writing dynamic and reusable HTML. Hopefully the above example is easy to follow, but if you have any difficulties understanding what it does, just wait until we view this in a browser, then inspect the page source to see the markup it produces.

If you’d like a refresher as to what JavaScript templates and/or templating engines are, and when you should use them, read An Overview of JavaScript Templating Engines.

### Install Pug and Integrate It into the Express App
Next, we’ll need to install pug, saving it as a dependency:

```
npm i --save pug
```

Then configure app.js to use Pug as a layout engine and to look for templates inside the views folder:

```javascript
const express = require('express');
const path = require('path');
const routes = require('./routes/index');

const app = express();

app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'pug');

app.use('/', routes);

module.exports = app;
```

You’ll notice that we’re also requiring Node’s native Path module, which provides utilities for working with file and directory paths. This module allows us to build the path to our views folder using its join method and __dirname (which returns the directory in which the currently executing script resides).

### Alter the Route to Use Our Template
Finally, we need to tell our route to use our new template. In routes/index.js:

```javascript
router.get('/', (req, res) => {
  res.render('form');
});
```

This uses the render method on Express’s response object to send the rendered view to the client.

So let’s see if it worked. As we’re using nodemon to watch our app for changes, you should simply be able to refresh your browser and see our brutalist masterpiece.

### Define a Layout File for Pug
If you open your browser and inspect the page source, you’ll see that Express only sent the HTML for the form: our page is missing a doc-type declaration, as well as a head and body section. Let’s fix that by creating a master layout for all our templates to use.

To do this, create a layout.pug file in the views folder and add the following code:

```pug
doctype html
html
  head
    title= `${title}`

  body
    h1 My Amazing App

    block content
```
The first thing to notice here is the line starting title=. Appending an equals sign to an attribute is one of the methods that Pug uses for interpolation. You can read more about it here. We’ll use this to pass the title dynamically to each template.

The second thing to notice is the line that starts with the block keyword. In a template, a block is simply a “block” of Pug that a child template may replace. We’ll see how to use it shortly, but if you’re keen to find out more, read this page on the Pug website.

### Use the Layout File from the Child Template
All that remains to do is to inform our form.pug template that it should use the layout file. To do this, alter views/form.pug, like so:

```pug
extends layout

block content
  form(action="." method="POST")
    label(for="name") Name:
    input(
      type="text"
      id="name"
      name="name"
    )

    label(for="email") Email:
    input(
      type="email"
      id="email"
      name="email"
    )

    input(type="submit" value="Submit")
```

And in routes/index.js, we need to pass in an appropriate title for the template to display:

```javascript
router.get('/', (req, res) => {
  res.render('form', { title: 'Registration form' });
});
```

Now if you refresh the page and inspect the source, things should look a lot better.

## Dealing with Forms in Express

Currently, if you hit our form’s Submit button, you’ll be redirected to a page with a message: “Cannot POST /”. This is because when submitted, our form POSTs its contents back to / and we haven’t defined a route to handle that yet.

Let’s do that now. Add the following to routes/index.js:

```javascript
router.post('/', (req, res) => {
  res.render('form', { title: 'Registration form' });
});
```

This is the same as our GET route, except for the fact that we’re using router.post to respond to a different HTTP verb.

Now when we submit the form, the error message will be gone and the form should just re-render.

### Handle Form Input
The next task is to retrieve whatever data the user has submitted via the form. To do this, we’ll need to install a package named body-parser, which will make the form data available on the request body:

```
npm install --save body-parser
```

We’ll also need to tell our app to use this package, so add the following to app.js:

```javascript
const bodyParser = require('body-parser');
...
app.use(bodyParser.urlencoded({ extended: true }));
app.use('/', routes);

module.exports = app;
```

Note that there are various ways to format the data you POST to the server, and using body-parser’s urlencoded method allows us to handle data sent as application/x-www-form-urlencoded.

Then we can try logging the submitted data to the terminal. Alter the route handler like so:

```javascript
router.post('/', (req, res) => {
  console.log(req.body);
  res.render('form', { title: 'Registration form' });
});
```

Now when you submit the form, you should see something along the lines of:

```
{name: 'Jim', email: 'jim@example.com'}
```

![Form output logged to terminal](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/01/1515667076form-output-logged-to-terminal.jpg "Form output logged to terminal")

*Form output logged to terminal*

### A Note about Request and Response Objects
By now you’ve hopefully noticed the pattern we’re using to handle routes in Express.

```javascript
router.METHOD(route, (req, res) => {
  // callback function
});
```

The callback function is executed whenever somebody visits a URL that matches the route it specifies. The callback receives a req and res parameter, where req is an object full of information that is coming in (such as form data or query parameters) and res is an object full of methods for sending data back to the user. There’s also an optional next parameter which is useful if you don’t actually want to send any data back, or if you want to pass the request off for something else to handle.

Without getting too deep into the weeds, this is a concept known as middleware (specifically, router-level middleware) which is very important in Express. If you’re interested in finding out more about how Express uses middleware, I recommend you read the Express docs.

### Validating Form Input
Now let’s check that the user has filled out both our fields. We can do this using express-validator module, a middleware that provides a number of useful methods for the sanitization and validation of user input.

You can install it like so:

```
npm install express-validator --save
```

And require the functions we’ll need in routes/index.js:

```javascript
const { body, validationResult } = require('express-validator/check');
```

We can include it in our route handler like so:

```javascript
router.post('/',
  [
    body('name')
      .isLength({ min: 1 })
      .withMessage('Please enter a name'),
    body('email')
      .isLength({ min: 1 })
      .withMessage('Please enter an email'),
  ],
  (req, res) => {
    ...
  }
);
```

As you can see, we’re using the body method to validate two properties on req.body — namely, name and email. In our case, it’s sufficient to just check that these properties exist (i.e. that they have a length greater than one), but express-validator offers a whole host of other methods that you can read about on the project’s home page.

In a second step, we can call the validationResult method to see if validation passed or failed. If no errors are present, we can go ahead and render out a “Thanks for registering” message. Otherwise, we’ll need to pass these errors back to our template, so as to inform the user that something’s wrong.

And if validation fails, we’ll also need to pass req.body back to the template, so that any valid form inputs aren’t reset:

```javascript
router.post(
  '/',
  [
    ...
  ],
  (req, res) => {
    const errors = validationResult(req);

    if (errors.isEmpty()) {
      res.send('Thank you for your registration!');
    } else {
      res.render('form', {
        title: 'Registration form',
        errors: errors.array(),
        data: req.body,
      });
    }
  }
);
```

Now we have to make a couple of changes to our form.pug template. We firstly need to check for an errors property, and if it’s present, loop over any errors and display them in a list:

```pug
extends layout

block content
  if errors
    ul
      for error in errors
        li= error.msg
  ...
```

If the li= looks weird, remember that pug does interpolation by following the tag name with an equals sign.

Finally, we need to check if a data attribute exists, and if so, use it to set the values of the respective fields. If it doesn’t exist, we’ll initialize it to an empty object, so that the form will still render correctly when you load it for the first time. We can do this with some JavaScript, denoted in Pug by a minus sign:

```
-data = data || {}
```

We then reference that attribute to set the field’s value:

```javascript
input(
  type="text"
  id="name"
  name="name"
  value=data.name
)
```

That gives us the following:

```pug
extends layout

block content
  -data = data || {}

  if errors
    ul
      for error in errors
        li= error.msg

  form(action="." method="POST")
    label(for="name") Name:
    input(
      type="text"
      id="name"
      name="name"
      value=data.name
    )

    label(for="email") Email:
    input(
      type="email"
      id="email"
      name="email"
      value=data.email
    )

    input(type="submit" value="Submit")
```

Now, when you submit a successful registration, you should see a thank you message, and when you submit the form without filling out both field, the template should be re-rendered with an error message.

## Interact with a Database
We now want to hook our form up to our database, so that we can save whatever data the user enters. If you’re running Mongo locally, don’t forget to start the server with the command mongod.

### Specify Connection Details
We’ll need somewhere to specify our database connection details. For this, we’ll use a configuration file (which should not be checked into version control) and the dotenv package. Dotenv will load our connection details from the configuration file into Node’s process.env.

Install it like so:

```
npm install dotenv --save
```

And require it at the top of start.js:

```javascript
require('dotenv').config();
```

Next, create a file named .env in the project root (note that starting a filename with a dot may cause it to be hidden on certain operating systems) and enter your Mongo connection details on the first line.

If you’re running Mongo locally:

```
DATABASE=mongodb://localhost:27017/node-demo-application
```

If you’re using mLabs:

```
mongodb://<dbuser>:<dbpassword>@ds251827.mlab.com:51827/<dbname>
```

Note that local installations of MongoDB don’t have a default user or password. This is definitely something you’ll want to change in production, as it’s otherwise a security risk.

### Connect to the Database
To establish the connection to the database and to perform operations on it, we’ll be using Mongoose. Mongoose is an ORM for MongoDB, and as you can read on the project’s home page:

>Mongoose provides a straight-forward, schema-based solution to model your application data. It includes built-in type casting, validation, query building, business logic hooks and more, out of the box.

What this means in real terms is that it creates various abstractions over Mongo, which make interacting with our database easier and reduce the amount of boilerplate we have to write. If you’d like to find out more about how Mongo works under the hood, be sure to read our Introduction to MongoDB.

To install Mongoose:

```
npm install --save mongoose
```

Then, require it in start.js:

```javascript
const mongoose = require('mongoose');
```

The connection is made like so:

```javascript
mongoose.connect(process.env.DATABASE, { useMongoClient: true });
mongoose.Promise = global.Promise;
mongoose.connection
  .on('connected', () => {
    console.log(`Mongoose connection open on ${process.env.DATABASE}`);
  })
  .on('error', (err) => {
    console.log(`Connection error: ${err.message}`);
  });
```

Notice how we use the DATABASE variable we declared in the .env file to specify the database URL. We’re also telling Mongo to use ES6 Promises (these are necessary, as database interactions are asynchronous), as its own default promise library is deprecated.

This is what start.js should now look like:

```javascript
require('dotenv').config();
const mongoose = require('mongoose');

mongoose.connect(process.env.DATABASE, { useMongoClient: true });
mongoose.Promise = global.Promise;
mongoose.connection
  .on('connected', () => {
    console.log(`Mongoose connection open on ${process.env.DATABASE}`);
  })
  .on('error', (err) => {
    console.log(`Connection error: ${err.message}`);
  });

const app = require('./app');
const server = app.listen(3000, () => {
  console.log(`Express is running on port ${server.address().port}`);
});
```

When you save the file, nodemon will restart the app and, if all’s gone well, you should see something along the lines of:

```
Mongoose connection open on mongodb://localhost:27017/node-demo-application
```

### Define a Mongoose Schema
MongoDB can be used as a loose database, meaning it’s not necessary to describe what data will look like ahead of time. However, out of the box it runs in strict mode, which means it’ll only allow you to save data it knows about beforehand. As we’ll be using strict mode, we’ll need to define the shape our data using a schema. Schemas allow you to define the fields stored in each document along with their type, validation requirements and default values.

To this end, create a models folder in the project root, and within that folder, a new file named Registration.js.

Add the following code to Registration.js:

```javascript
const mongoose = require('mongoose');

const registrationSchema = new mongoose.Schema({
  name: {
    type: String,
    trim: true,
  },
  email: {
    type: String,
    trim: true,
  },
});

module.exports = mongoose.model('Registration', registrationSchema);
```

Here, we’re just defining a type (as we already have validation in place) and are making use of the trim helper method to remove any superfluous white space from user input. We then compile a model from the Schema definition, and export it for use elsewhere in our app.

The final piece of boilerplate is to require the model in start.js:

```javascript
...

require('./models/Registration');
const app = require('./app');

const server = app.listen(3000, () => {
  console.log(`Express is running on port ${server.address().port}`);
});
```

### Save Data to the Database
Now we’re ready to save user data to our database. Let’s begin by requiring Mongoose and importing our model into our routes/index.js file:

```javascript
const express = require('express');
const mongoose = require('mongoose');
const { body, validationResult } = require('express-validator/check');

const router = express.Router();
const Registration = mongoose.model('Registration');
...
```

Now, when the user posts data to the server, if validation passes we can go ahead and create a new Registration object and attempt to save it. As the database operation is an asynchronous operation which returns a Promise, we can chain a .then() onto the end of it to deal with a successful insert and a .catch() to deal with any errors:

```javascript
if (errors.isEmpty()) {
  const registration = new Registration(req.body);
  registration.save()
    .then(() => { res.send('Thank you for your registration!'); })
    .catch(() => { res.send('Sorry! Something went wrong.'); });
} else {
  ...
}

...
```

Now, if you enter your details into the registration form, they should be persisted to the database. You can check this using Compass (making sure to hit the refresh button in the top left if it’s still running).

![Using Compass to check that our data was saved to MongoDB](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2018/01/1515669040mongo-compass-with-data.jpg "Using Compass to check that our data was saved to MongoDB")

*Using Compass to check that our data was saved to MongoDB*

### Retrieve Data from the Database
To round the app off, let’s create a final route, which lists out all of our registrations. Hopefully you should have a reasonable idea of the process by now.

Add a new route to routes/index.js, as follows:

```javascript
router.get('/registrations', (req, res) => {
  res.render('index', { title: 'Listing registrations' });
});
```

This means that we’ll also need a corresponding view template (views/index.pug):

```pug
extends layout

block content
  p No registrations yet :(
```

Now when you visit http://localhost:3000/registrations, you should see a message telling you that there aren’t any registrations.

Let’s fix that by retrieving our registrations from the database and passing them to the view. We’ll still display the “No registrations yet” message, but only if there really aren’t any.

In routes/index.js:

```javascript
router.get('/registrations', (req, res) => {
  Registration.find()
    .then((registrations) => {
      res.render('index', { title: 'Listing registrations', registrations });
    })
    .catch(() => { res.send('Sorry! Something went wrong.'); });
});
```

Here, we’re using Mongo’s Collection#find method, which, if invoked without parameters, will return all of the records in the collection. Because the database lookup is asynchronous, we’re waiting for it to complete before rendering the view. If any records were returned, these will be passed to the view template in the registrations property. If no records were returned registrations will be an empty array.

In views/index.pug, we can then check the length of whatever we’re handed and either loop over it and output the records to the screen, or display a “No registrations” message:

```pug
extends layout

block content

  if registrations.length
    table
      tr
        th Name
        th Email
      each registration in registrations
        tr
          td= registration.name
          td= registration.email
  else
    p No registrations yet :(
```

## Add HTTP Authentication
The final feature we’ll add to our app is HTTP authentication, locking down the list of successful registrations from prying eyes.

To do this, we’ll use the http-auth module, which we can install using:

```
npm install --save http-auth
```

Next we need to require it in routes/index.js, along with the Path module we met earlier:

```javascript
const path = require('path');
const auth = require('http-auth');
```

Next, let it know where to find the file in which we’ll list the users and passwords (in this case users.htpasswd in the project root):

```javascript
const basic = auth.basic({
  file: path.join(__dirname, '../users.htpasswd'),
});
```

Create this users.htpasswd file next and add a username and password separated by a colon. This can be in plain text, but the http-auth module also supports hashed passwords, so you could also run the password through a service such as Htpasswd Generator.

For me, the contents of users.htpasswd looks like this:

```
jim:$apr1$FhFmamtz$PgXfrNI95HFCuXIm30Q4V0
```

This translates to user: jim, password: password.

Finally, add it to the route you wish to protect and you’re good to go:

```javascript
router.get('/registrations', auth.connect(basic), (req, res) => {
  ...
});
```

## Serve Static Assets in Express
Let’s give the app some polish and add some styling using Twitter Bootstrap. We can serve static files such as images, JavaScript files and CSS files in Express using the built-in express.static middleware function.

Setting it up is easy. Just add the following line to app.js:

```javascript
app.use(express.static('public'));
```

Now we can load files that are in the public directory.

### Style the App with Bootstrap
Create a public directory in the project root, and in the public directory create a css directory. Download the minified version of Bootstrap v4 into this directory, ensuring it’s named bootstrap.min.css.

Next, we’ll need to add some markup to our pug templates.

In layout.pug:

```pug
doctype html
html
  head
    title= `${title}`
    link(rel='stylesheet', href='/css/bootstrap.min.css')
    link(rel='stylesheet', href='/css/styles.css')

  body
    div.container.listing-reg
      h1 My Amazing App

      block content
```

Here, we’re including two files from our previously created css folder and adding a wrapper div.

In form.pug we add some class names to the error messages and the form elements:

```pug
extends layout

block content
  -data = data || {}

  if errors
    ul.my-errors
      for error in errors
        li= error.msg

  form(action="." method="POST" class="form-registration")
    label(for="name") Name:
    input(
      type="text"
      id="name"
      name="name"
      class="form-control"
      value=data.name
    )

    label(for="email") Email:
    input(
      type="email"
      id="email"
      name="email"
      class="form-control"
      value=data.email
    )

    input(
      type="submit"
      value="Submit"
      class="btn btn-lg btn-primary btn-block"
    )
```

And in index.pug, more of the same:

```pug
extends layout

block content

  if registrations.length
    table.listing-table.table-dark.table-striped
      tr
        th Name
        th Email
      each registration in registrations
        tr
          td= registration.name
          td= registration.email
  else
    p No registrations yet :(
```

Finally, create a file called styles.css in the css folder and add the following:

```css
body {
  padding: 40px 10px;
  background-color: #eee;
}
.listing-reg h1 {
  text-align: center;
  margin: 0 0 2rem;
}

/* css for registration form and errors*/
.form-registration {
  max-width: 330px;
  padding: 15px;
  margin: 0 auto;
}
.form-registration {
  display: flex;
  flex-wrap: wrap;
}
.form-registration input {
  width: 100%;
  margin: 0px 0 10px;
}
.form-registration .btn {
  flex: 1 0 100%;
}
.my-errors {
  margin: 0 auto;
  padding: 0;
  list-style: none;
  color: #333;
  font-size: 1.2rem;
  display: table;
}
.my-errors li {
  margin: 0 0 1rem;
}
.my-errors li:before {
  content: "! Error : ";
  color: #f00;
  font-weight: bold;
}

/* Styles for listing table */
.listing-table {
  width: 100%;
}
.listing-table th,
.listing-table td {
  padding: 10px;
  border-bottom: 1px solid #666;
}
.listing-table th {
  background: #000;
  color: #fff;
}
.listing-table td:first-child,
.listing-table th:first-child {
  border-right: 1px solid #666;
}
```

Now when you refresh the page, you should see all of the Bootstrap glory!

## Conclusion
I hope you’ve enjoyed this tutorial. While we didn’t build the next Facebook, I hope that I was nonetheless able to help you get your feet wet in the world of Node-based web apps and offer you some solid takeaways for your next project in the process.

Of course it’s hard to cover everything in one tutorial and there are lots of ways you could elaborate on what we’ve built here. For example, you could check out our article on deploying Node apps and try your hand at launching it to Heroku or now. Alternatively, you might augment the CRUD functionality with the ability to delete registrations, or even write a couple of tests to test the app’s functionality.

Wherever you go from here, I’d be glad to hear any questions or comments below.
