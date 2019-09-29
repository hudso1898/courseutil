# Matt Hudson - Exploration 2 Writeup

**Backend - NodeJS, ExpressJS and MongoDB**

## Code

Using NodeJS, I created a NodeJS server utilizing the concepts I covered in this exploration. There are 3 mini applications running on the server:

1) File Retrieval Application - simple application that retrieves text files from the server. This uses Node/Express with the https, fs, and url modules.

2) Email Application - sends an email to me (mrh4hd@mail.missouri.edu) with whatever name/text the user enters in a simple HTML form. Very useful for
feedback forms and such! I used the `nodemailer` module, which I installed using `npm`. 

3) Database Application - Simple Node/Express application that uses MongoDB. Users can enter their name, age, and email into the database. This page also shows
all of the entries in the database.

Originally, I wrote my code using just Node; later, I refactored the code using Express middleware functions. This shortened my application greatly!

My code is running live [on my EC2 server](https://www.hudso.dev:8080).

(Note the port is **8080** not 8000). https://www.hudso.dev:8080

The application code is also located in the [code](./code) directory.

## What framework did you choose and why?

For this exploration, I decided to look into the back end of the MEAN stack (MongoDB, Express.js, and Node.js). Originally, I was going to learn about just Node.js and
MongoDB; however, I added Express.js since Express works with Node.js to serve wevb applications. I needed to explore back-end technologies, since my first exploration
centered on Angular, which worked in the front end. The big aspect of Angular I still wanted to learn was how to work with persistant data; the front-end aspects of Angular
only work on the client-side, but can't interact with much of the server-side from what I explored. So, learning these backend frameworks (at least, what they are, basic
usage, etc.) can help me understand how to build a complete web application in the MEAN stack, not just a LAMP stack.

I mainly explored Node, with a little bit of Express and MongoDB. I did this because Node is the framework that both Express and MongoDB use. Therefore, I wanted
to built my foundation in Node, then explore a little bit into Express and Mongo.

## What did you learn about this framework?

## Node.js

### [0] History of NodeJS

NodeJS was created by *Ryan Dahl* in 2009, 13 years after the first server-side javascript framework was launched. Dahl cited his reasoning for making Node.js as bypassing
the limitations of the Apache web server, which could only hold so many concurrent connections (more on this later):

> Dahl criticized the limited possibilities of the most popular web server in 2009, Apache HTTP Server, to handle a lot of concurrent connections (up to 10,000 and more) and 
> the most common way of creating code (sequential programming), when code either blocked the entire process or implied multiple execution stacks in the case of simultaneous 
> connections.

Another important tool with NodeJS is **npm**, a package manager for Node.js. A package manager is an application that handles retrieving and installing software packages for
some computer system, such as *apt-get* or *yum* in Linux distributions. *npm* serves this function for Node.js, allowing developers to easily download, upgrade and use
node.js libraries. npm was released in 2010.

Originally, node.js was only for Linux/Mac systems. In 2011, a Windows version was released. In 2012, Dahl gave the project to Issac Schuleter, the creater of npm. In 2014, the
project changed hands again, but this sparked conflict over the project. Later that year, Fedor Indunty started a *fork* of Node.js (fork = branching off a project, but to
create an entirely separate and independent branch) as *io.js*. In 2015, the Node.js Foundation was formed, and both Node.js and io.js communities joined in this foundation.

Later that year, both projects were merged into *Node 4.0*, and io.js development was discontinued. Node is still under development to this day, and remains one of the most
popular, if not *the* most popular, server-side javascript framework.

### [1] What is NodeJS?

Node.js is *a server-side javascript framework built for asyncronous handling of various server requests*. Let's break this down:

**Server-side javascript framework** - as we've seen, Javascript typically runs on the *client* machine. This is done on normal web pages, and even in Angular when it's
compiled. Ultimately, the 'grunt work' of Javascript is done by the device accessing the content, not at the server. So, the difference is that Node.js runs on the
*server* and not on the client. 

**Asyncronous handling** - Imagine a relay race, where you have a team of 5 people. The first person runs 100m to the end of the track, then runs back. They tag the second
person, then they can start running, and this process continues. This models the normal flow of execution of systems like Apache and PHP. In this case, each 'runner' is a
request for something like a web page or a file. Apache can only process one request at a time, and the same for PHP. While this can still happen very fast, this becomes
a problem when talking about *scalability*, or how effective the application is when it is used more and more.

Node.js handles requests *asycronously*. In this analogy, all five runners can run at the same time, meaning they all finish 5x as fast! The beauty of node.js is scalability.
Large applications can handle thousands, tens of thousands, etc. of requests so quickly because of this feature. It's very memory efficient, and takes a lot of wasted waiting
time (i.e. for Node to wait for the Linux filesystem to retrieve a file) out of handling requests.

**Various server requests** - node.js is a general language for *serving* items on a server. This includes HTTP(S) requests, so Node.js can act as a web server. Additionally,
since node.js is a server-side language like PHP, it can serve dynamic web pages just like PHP can! In fact, node can serve just about anything as long as there's a module
 for it. It can serve files through FTP, interact with a database (such as MySQL and MongoDB), and more. I've seen Discord bots use a Node.js library (I noticed someone did
 an exploration on Discord.js a few weeks ago), and so I've had some prior exposure to this.

 In short, Node.js is javascript that runs on the server, that handles events and serves content in a very efficient manner.

### [2] Installing NodeJS and npm ; Serving Content

We went over this method in class; the basic principle is to use apt-get or the node.js official website to download node.js and npm. You can then use `node` to run the
node server. Additionally, you can use `pm2` to enable the node service to run in the background; this way, I can set up a node server on my EC2 instance.

By default, Node.js serves on port 8000 or port 8080. In this way, if we want node.js to serve web pages, we need to type `:8000` in our web browser. Otherwise, we'll
access our apache2 web server instead.

### [3] Modules

Modules in Node.js are essentially the same as Javascript libraries. They are a set of related functions you include in your application that pertain to some specific
service and/or function. To include modules in your Node.js application, you use the `require()` function to load the module into a `var`:
```javascript
var module = require('moduleName');
```

In essence, modules return objects, of which you can access data through the `var` you create in this way. So, you can make your own modules as well like this:
```javascript
exports.methodName = function() {
	return 'foo';
};
```

If you put this into a file named 'module.js', you can require this module and access this function from other Node.js applications:
```javascript
var module = require('./module.js');
var response = module.methodName(); // response will contain 'foo'
```

This allows us to separate related functions in a larger Node application. Node has many built-in modules as well, including those that enable Node to serve web page
content, such as `http`. Other modules enable other functions, and this is where `npm` comes in. You can use npm to install libraries which you can access in this way.

### [4] HTTP Module

The `http` module is the module we can use to serve web content. We access this module by importing the 'http' module in our Node.js application. To use the module, we define
an HTTP server using the `http.createServer()` method. To that method, we pass a function that takes in two parameters: request to represent data from the request, and
response to represent the response body the server will write to. In that method, we can define how the server will handle an HTTP request. We also add the `listen()` method
on this resulting object to define what port the server should be listening on.

We should also use the `res.writeHead()` method to define the HTTP header (we saw this when looking at REST). This can define the Content-Type of our response; note this
includes many other things than just html.

```javascript
var http = require(http);
http.createServer(function (req,res) {
	// read in data through req, write data through res
	var url = req.url;
	// define the header to be 200 (response OK) and content type as html
	res.writeHead(200, {'Content-Type': 'text/html'});
	// write text to the response body
	res.write("Hello there from " + url);
	res.end(); // end the response body

}).listen(8000); // listen on port 8000
```

### [5] HTTPS Module

To use HTTPS, we simply import the `https` module instead of the http module. However, this includes a few extra steps to provide the security for HTTPS.
First, we need to set up HTTPS just like for Apache and get our private, public and intermediate keys. Once that is finished, we need to specify an options
variable we sent to `createServer()`. This object will contain our keys that enable HTTPS. We use the `fs` module to read in the keys from the filesystem. Note
the `readFileSync()` method reads files syncronously, as opposed to in parallel (asyncronously). This is because these key files are very important, and should
be handled as such to ensure the integrity of the data.

```javascript
var https = require('https');
var fs = require('fs');
const options = {
	key: fs.readFileSync('/path/to/private/key');
	cert: fs.readFileSync('/path/to/public/key');
	ca: fs.readFileSync('/path/to/ca/key');
}
https.createServer(options, handlerFunction).listen(8080);
```
### [6] FS Module

The `fs` module contains useful methods to manipulate the filesystem of the host machine. A great benefit of this module is that there is no need to handle resource
closing - it's all done behind the scenes and abstracted away. These methods are important and used to handle files:

- `fs.readFile('filepath', handlerFunction(err, data))` - reads in a file from the filesystem. The second argument contains a function that has two parameters:
err to represent any error messages, and data that contains the content of the file. In this function, you can use this data such as writing the data to the response.

- `fs.open(fileName, mode, handlerFunction(err, data))` - opens a file from the filesystem. The `mode` parameter specifies what to do to the file, for example 'r' to read
and 'w' to write. This can also be used to create new files in the filesystem.

- `fs.append(fileName, content, errorHandlerFunction(err))` - appends 'content' to the file specified. You also specify an error handler function to this method.

- `fs.writeFile(fileName, content, errorHandlerFunction(err))` - same as append, but this overwrites any previous content if it exists.

- `fs.rename(oldFilePath, newFilePath, errorHandlerFunction(err))` - renames a file in the file system.

- `fs.unlink(fileName, errorHandlerFunction(err))` - unlink is used to delete files in the file system.

Together, these methods allow us to implement full CRUD operators on the filesystem.

### [7] URL Module

The `url` module allows Node.js to manipulate objects in the URL. This allows us to adjust page content based on the components of the URL. The most important
method in the URL module is `url.parse(url, true)`. (true sets a flag to return an object rather than a string). We can access those properties through the object
this method returns, and perform operations based on the URL.

```javascript
// URL: https://www.hudso.dev/home
var url = url.parse(req.url, true); // req is from http.createServer
res.write("The pathname is " + url.pathname); // The pathname is /home
```
### [8] Events

NodeJS is built around *Events*, which are actions that occur in the system the server needs to respond to. For example, an HTTP(S) request is an event. Each event
needs a *handler function*, which runs each time the event is triggered. The event handler function uses the data associated with the event to construct a response to
the event. You can even create your own events in NodeJS using the `events` built-in module. This module is critical to many modules in NodeJS; for example,
Discord.js uses events to allow bots to respond when they are sent a message, or when a user joins a server, etc.

### [9] Email Module

The `nodemailer` module allows node to send emails. The `createTransport()` method initializes an email service with your credentials. For this example,
you can use the 'gmail' service and link that to your personal gmail account. You then create an options object that contains from, to, subject, and text/html body.
You can then send mail using the `sendMail()` method on the transporter created by createTransport().

A few notes about this:

	- You install nodemailer through npm: npm install nodemailer
	- I used my gmail account to send mail, but I had to enable 'less secure apps' for this to work.

## Express.js (A brief overview)

### [0] What is Express.js?

Express.js is a framework for Node.js. Express is, in fact, just a module of Node.js! It is built to make it easier to create web servers with Node, building on
top of the http and https modules built into Node. It was made originally in 2010, and is currently owned by IBM. However, IBM gave Express to the Node.js Foundation
as well.

The point of Express is to make developing web applications easier when using Node. Node has many options for many different services; however, the http(s) module
is not complete to deal with many HTTP(S) requests and website workflow. Express is the framework which enables greater modularity, and for controlling the
back-end of the web application.

### [1] Express.js Basic Usage - Middleware Methods

To initialize an express application, import the express module, and create a new variable containing the application.

```javascript
var express = require('express');
const app = express();
```

Various methods on the `app` object allow us to define different routes and how to handle specific requests such as `GET`, `POST`, `PUT`, `DELETE`, etc.

```javascript
app.get("/", (req,res) => {
	// handle get requests for the root directory (index) here!
});
app.post("/email", (req,res) => {
	// handle post requests in email here!
})
```

This kind of code is called **Middleware**, since it sits 'in the middle' of the transaction and has access to both the request *and* response. So, you can define
different handling of GET and POST requests. You can also use `post()` to handle POST requests, just as PHP would! However, this is much more intuitive and useful
than PHP, and can do more than just serve web content using other Node.js methods!

You can also use the `express.static()` method to serve static content, i.e. files on the system. I left my file server application in basic Node, but I could
upgrade it to use Express using this method.

One more thing: You can also use wildcard expressions in middleware handlers to match certain URLs! Wildcard expressions (regular expressions, really) allow the
application to decide which handler to use based on the structure of the URL. This allows us to implement many routing concepts we've covered!

## MongoDB

### [0] What is MongoDB?

MongoDB is a database application, which is used to store data on the server. However, MongoDB differs from MySQL: it is a **NoSQL** database, meaning it does
not use any SQL. Instead, it is *document-driven*, meaning it uses a syntax more akin to JSON. Instead of creating tables and records, you create *collections*
and *documents*.

MongoDB is also based on the Cloud, meaning the data does not live on the EC2 instance. Instead, the server connects to this cloud database through a *driver*.
In a Node server, this driver is the mongodb module, which provides an interface to the database.

### [1] Using MongoDB in Node/Express

To get started, I visited https://www.mongodb.com/ to create an account with the free tier storage. I won't need more than that, and I don't want to pay for anything
extra anyways! Once there, I accessed through the web portal to create a user and allow remote access from my instance.

Using MongoDB is very easy in a node server. I installed the mongodb module using `npm install mongodb --save`. Then, I imported the module just like any other
module using the `require()` method. However, in this case, I need the `MongoClient` module from the greater `mongodb` module.
```javascript
var mongoClient = require('mongodb').MongoClient;
```

Then, I can connect to the database using the `connect()` method.

```javascript
mongoClient.connect(url, (err, db) => {
	if (err) {
		// handle connection errors here
	}
	// do database manipulations here using the db object
});
```

### [2] Databases, Collections and Documents

Recall that MongoDB is a *document-based* database system. *Documents* are the same as entries or *records* in a MySQL database. However, instead of using SQL
to access the database, documents are treated essentially the same as JSON objects, i.e. key value pairs. So, there's no need to define or create databases and collections
before inserting data; they are created automatically.

### [3] Creating Databases/Collections/Documents

To create entries in a collection, we create a Javascript object contianing the data we want to add. Then, we pass this object into the `collections().insertOne()` method.
We can also use the `insertMany()` method to insert many documents into the collection. MongoDB doesn't require us to create the Database/Collection separately. So, on
the first entry, the database and collection will be created automatically by MongoDB.

```javascript
var document = {name: 'Matt Hudson', age: 22, email: 'mrh4hd@mail.missouri.edu'}
var databaseObject = db.db("myDatabase");
databaseObject.collection('students').insertOne(document, (err, result) => {
	// check for results here
});
```

### [4] Reading from a Database/Collection

Instead of doing a `SELECT * FROM table` like in SQL, we use the `find({}).toArray()` method in MongoDB. The first object is an object that defines search criteria. An empty
object is the same as a `SELECT *` in MySql. Then, the result will contain the documents in the collection as objects. You can then iterate through each object.

```javascript
results.forEach(function(doc) {
	// do some stuff with doc here
});
```

You can specify search criteria to `find()` in order to select based on some criteria.

## What resources did you utilize?

- [W3Schools NodeJS Tutorial](https://www.w3schools.com/nodejs/default.asp)

- [W3Schools MongoDB Tutorial](https://www.w3schools.com/nodejs/nodejs_mongodb.asp)

W3Schools is the bread and butter of learning web development, and I have used W3Schools tutorials to learn everything from HTML, CSS, Javascript, PHP, MySql, and much much
more. I always trust W3Schools as a starting point for learning a new language or framework, since it steps through the basics with examples you can try directly in the
browser. I also liked that W3Schools groups together NodeJS and MongoDB, so it's easy to learn these two in conjunction with each other.

- [NodeJS API Documentation](https://nodejs.org/api/)

API Documentation, I have found, is a mixed bag. Sometimes API documentation can be enlightening; however, sometimes the documentation is written too abstractly to be
accessible to beginners.

- [NodeJS Wikipedia History Page](https://en.wikipedia.org/wiki/Node.js#History)

Good general read on the history of NodeJS, and helped me understand why it was made and what it brought to the web server table for server-side javascript.

- [Short tutorial on Node.JS HTTPS](https://aghassi.github.io/ssl-using-express-4/)

A short article explaining the usage of the HTTPS module in Node.js and how to set up a HTTPS server in Node.

- [Bootstrap Classes](https://www.w3schools.com/bootstrap/)

Useful lookup for various bootstrap classes. I used these to create my demo applications.

- [Nodemailer Blog Post](https://monkey.work/blog/2018/01/21/node-js/)

A small blog post that helped me debug the email application using nodemailer.

- [ExpressJS Official Website](https://expressjs.com/)

Official page of Express.js, to learn about the framework and some basic syntax.

- [This Express tutorial](https://scotch.io/tutorials/use-expressjs-to-get-url-and-post-parameters)

I used this to figure out how to get GET and POST parameters in an Express.js app.post() method.


## What do you still want to learn about this framework?

The big aspect of these frameworks I still want to explore is how to integrate these services with a front-end framework like **Angular**. I touched on Angular services,
and I feel combining these two aspects together can create a fully-functional web application; For this exploration, I wanted to explore the basics of Node/Express/Mongo, so
I can implement a full-stack application later on in the course.

I explored a little bit into this, as I used some HTML templates in my application using the `fs` module. Express also has some functionality for routing and handling
various routes in the URL, and for handling different cases; however, it still lacks the SPA feel I had with my Angular applications. If/when I look into combining
these frameworks, I can make a really useful and aesthetically beautiful web application!

I also want to work more with Express, and play around with how Express works more. I only use basic middleware methods, so there's a lot left for me to learn.

Finally, I only looked into Create and Update methods in MongoDB, so I'd like to look into updating/deleting at some point. This, and also looking into how to specify
conditions for queries (I only used the `SELECT *` equivalent in MongoDB).

## What problems did you run into?

One problem I encountered was when I was creating my email application. First, I forgot to install the `nodemailer` module, which is what I used to send email. I used `npm`
to install this module once I realized I needed to install the module. Also, I recieved errors when trying to send emails. To get more information, I added
`res.write(err.response)` to get more information. I noticed I had to allow less secure apps in gmail in order for this to work. Also, I needed to unlock the Captcha as well.

Another problem was figuring out I needed to use port 8080 instead of port 8000. I configured my node server in class using Express, which listened for HTTPS. When trying
to connect on port 8000, I got an SSL error. This is because the web browser was trying to use HTTPS, but the Node server was still using HTTP. Therefore, the browser refused
to load any content. I followed the steps to configure HTTPS on port 8080 and it worked just fine.

One last problem was when my Node.js service stopped unexpectadly. If there's a syntax error in the JS, the service won't run in pm2. I used an online javascript syntax
checker [here](https://esprima.org/demo/validate.html) to check this syntax.