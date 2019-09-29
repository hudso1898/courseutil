# Matt Hudson - Exploration 2 Sources
 
## Documentation Links

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

## Components I explored

**NodeJS**
	
	- What is Node.js? (A server-side javascript framework for asyncronous serving of requests, more than just HTTP(S))
	- History of Node.js
	- NodeJS Modules
	- HTTP(S) Modules, working with request and response objects
	- FS (Filesystem) module to retrieve and manipulate resources
	- URL Module (manipulating parts of the URL)
	- Events
	- Nodemailer Module to send email through a node.js service

**ExpressJS**

	- What is Express.js? (A Node.js framework for building web applications)
	- Installing and importing Express.js into node
	- Creating an express app object and serving the application
	- Using app.get() and app.post() to respond to GET and POST requests for specific URLs
	- Using Middleware

**MongoDB**
	
	- What is MongoDB? (document-based NoSQL cloud-based database)
	- Creating a MongoDB Account
	- Accessing the MongoDB Web-Based Atlas
	- Connecting to MongoDB through Node.js
	- Creating databases and collections
	- Inserting documents into a collection
	- Finding documents in a collection
	- Building an application to Insert/Find documents and build dynamic HTML

I combined all of these components together to build sample applications deployed on my EC2 instance. 

## Troubleshooting

One problem I encountered was when I was creating my email application. First, I forgot to install the `nodemailer` module, which is what I used to send email. I used `npm`
to install this module once I realized I needed to install the module. Also, I recieved errors when trying to send emails. To get more information, I added
`res.write(err.response)` to get more information. I noticed I had to allow less secure apps in gmail in order for this to work. Also, I needed to unlock the Captcha as well.

Another problem was figuring out I needed to use port 8080 instead of port 8000. I configured my node server in class using Express, which listened for HTTPS. When trying
to connect on port 8000, I got an SSL error. This is because the web browser was trying to use HTTPS, but the Node server was still using HTTP. Therefore, the browser refused
to load any content. I followed the steps to configure HTTPS on port 8080 and it worked just fine.

One last problem was when my Node.js service stopped unexpectadly. If there's a syntax error in the JS, the service won't run in pm2. I used an online javascript syntax
checker [here](https://esprima.org/demo/validate.html) to check this syntax.

## GitHub Repository Link

[GitHub Repository Link](https://github.com/Mizzou-CSIT2830-CS7830-F19/exploration2-hudso1898)

## Running Code Link

[link to my running code on my instance](https://www.hudso.dev:8080)