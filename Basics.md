## Creating a Node Server

```js // core modules that node.js ships with: http, https, fs (file system), path, os (these are some modules, not all)
const http = require('http');

const server = http.createServer((req, res) => {
});

server.listen(3000);
```

createServer() method turns your computer into an HTTP server. Has a function as an arg which has two args, req and res. Takes in a request, and returns a response.
Listens on a specified port. 

## Node Lifecycle & Event Loop

Node.js keeps on looping as long as there are event listeners registered. For example, the request listener function in createServer(). Unless process.exit() is called
in our request listener, our server will keep looping to listen for requests (which is what we want it to do). 

## Requests

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // console.log(req);
  console.log(req.url, req.method, req.headers);
});

server.listen(3000);
```
We can read through some of the data from the request by accessing the request parameter's fields (here, we just console.log it and do nothing with it). 

## Sending Responses

```js
const http = require('http');

const server = http.createServer((req, res) => {
    res.setHeader('Content-Type', 'text/html'); /**  attaches a header to our response, which passes some meta-info that declares the type
    of our response (in this case, it's html). Content-Type is a supported header. Header: metadata. */
    res.write('<html>'); // write some data to the response
    res.write('<head><title>My First Page</title></head>');
    res.write('<body><h1>Hello world!</h1></body>')
    res.write('</html>')
    res.end();
});
server.listen(3000);
```
Sends back some header info with some HTML response body. Express is leveraged to make this process less verbose. 

## Routing Requests

Lets try doing things based on which route was entered. 

*Note: GET requests are automatically sent when you click a link or URL. A POST request has to be set up by you. 

```js
const http = require('http');

const server = http.createServer((req, res) => {
    const url = req.url;
    if (url === '/') {
        res.write('<html>'); // write some data to the response
        res.write('<head><title>Enter Message</title></head>');
        res.write('<body><form action="/message" method="POST"><input name="message"><button type="submit">Send</button></form></body>')
        res.write('</html>')
        res.end();
        return res.end();
    }
    res.setHeader('Content-Type', 'text/html');
    res.write('<html>'); 
    res.write('<head><title>My First Page</title></head>');
    res.write('<body><h1>Hello world!</h1></body>')
    res.write('</html>')
    res.end();
});

server.listen(3000);
```

Presents the user with a form when the user enters the site on the home route (localhost:3000/). Once the user submits the form, the url becomes /message, and the
if statement is not executed (other body of statements executes). 

## Redirecting Requests

```js
const http = require('http');
const fs = require('fs');

const server = http.createServer((req, res) => {
    const url = req.url;
    const method = req.method;
    if (url === '/') {
        res.write('<html>'); // write some data to the response
        res.write('<head><title>Enter Message</title></head>');
        res.write('<body><form action="/message" method="POST"><input name="message"><button type="submit">Send</button></form></body>')
        res.write('</html>')
        res.end();
        return res.end();
    }
    /** We want to achieve two things:
     * 1. Redirect the user to localhost:3000/
     * 2. Create a new file and store the message the user entered in it. (Here, we will only get DUMMY_TEXT, but we work on parsing request
     * bodies in the next step)
     */
    if (url === '/message' &&  method === 'POST'){  // only enter this if statement if we have a POST request to /message. 
        fs.writeFileSync('message.txt', 'DUMMY_TEXT');
        // res.writeHead(); // allows us to write some metainformation in one go, but we could also do:
        res.statusCode = 302;
        res.setHeader('Location', '/');
        return res.end();
    } 
    res.setHeader('Content-Type', 'text/html');
    res.write('<html>'); 
    res.write('<head><title>My First Page</title></head>');
    res.write('<body><h1>Hello world!</h1></body>')
    res.write('</html>')
    res.end();
});

server.listen(3000);
```

## Parsing Request Bodies

Requests are sent in chunks so that we can work on the request without waiting for the full request to be read. To organize these chunks, we use a buffer. 
A buffer is like a bus stop, which is a construct which allows us to hold multiple chunks and work with them before they are released. 

```js
const http = require('http');
const fs = require('fs');

const server = http.createServer((req, res) => {
    const url = req.url;
    const method = req.method;
    if (url === '/') {
        res.write('<html>'); // write some data to the response
        res.write('<head><title>Enter Message</title></head>');
        res.write('<body><form action="/message" method="POST"><input name="message"><button type="submit">Send</button></form></body>')
        res.write('</html>')
        res.end();
        return res.end();
    }
    /** We want to achieve two things:
     * 1. Redirect the user to localhost:3000/
     * 2. Create a new file and store the message the user entered in it. (Here, we will only get DUMMY_TEXT, but we work on parsing request
     * bodies in the next step)
     */
    if (url === '/message' &&  method === 'POST'){  // only enter this if statement if we have a POST request to /message. 
        // allows us to listen to events (event listener), here, we are listening for data whenever a new chunk is ready to be read
        const body = [];
        req.on('data', (chunk) => {
            console.log(chunk);
            body.push(chunk); // node will keep pushing these chunks into our body until it's done. 
        }); 
        // the end listener is fired once the incoming request data is done being parsed
        req.on('end', () => {
            // to interact with these chunks, we now need to buffer them
            const parsedBody = Buffer.concat(body).toString();
            console.log(parsedBody);
            const message = parsedBody.split('=')[1];
            fs.writeFileSync('message.txt', message); // now we will get a file with our text in it
        })
        res.statusCode = 302;
        res.setHeader('Location', '/');
        return res.end();
    } 
    res.setHeader('Content-Type', 'text/html');
    res.write('<html>'); 
    res.write('<head><title>My First Page</title></head>');
    res.write('<body><h1>Hello world!</h1></body>')
    res.write('</html>')
    res.end();
});

server.listen(3000);
```
The output will look like this in the terminal (from the console.logs in the code):

![image](https://user-images.githubusercontent.com/104043093/164801602-9699d434-160a-4b90-926f-c0419b690b41.png)


## Understanding Event Driven Code Execution

The order of execution of code is not necessarily the order in which it is written. 

Example:
```js
req.on('end', () => {
            // to interact with these chunks, we now need to buffer them
            const parsedBody = Buffer.concat(body).toString();
            console.log(parsedBody);
            const message = parsedBody.split('=')[1];
            fs.writeFileSync('message.txt', message); // now we will get a file with our text in it
        })
        res.statusCode = 302;
        res.setHeader('Location', '/');
        return res.end();
```
Even though we send a response with res.end(), it does not mean that our event listeners are dead. They will still execute even if the response has already gone out.
This means that if we do something in the event listener that influences the response, that we should move the response code into the event listener since we have
that dependency in this case. So, refactoring:

```js
req.on('end', () => {
            // to interact with these chunks, we now need to buffer them
            const parsedBody = Buffer.concat(body).toString();
            console.log(parsedBody);
            const message = parsedBody.split('=')[1];
            fs.writeFileSync('message.txt', message); // now we will get a file with our text in it
            res.statusCode = 302;
            res.setHeader('Location', '/');
            return res.end();
        })
```
Node.js sometimes executes passed in functions asynchronously. In such cases, Node.js won't immediately run the function. Instead, when it encounters that line,
it will add an event listener internally (and subsequently manages all the event listeners internally). In this case, the 'end' listener will execute once
Node.js is finished parsing the request. To summarize, the order in which the code is written does not necessarily correlate to the order of execution for this reason.
So, now with our modified code, the original 'HTML Hello world' page will be executed first, because Node.js moves on to the next lines (the 'hello world' lines).
We will now get an error because we called res.end() in the 'hello world' code block, and then it tried executing the 'end' event listener code, an error will be 
thrown because we should not be sending any responses after res.end() is called. To avoid the error, we should return the 'end' listener code so that the 'hello world'
doesn't get executed once the 'end' event listener is parsed by Node. Refactored:

```js
        return req.on('end', () => {
            // to interact with these chunks, we now need to buffer them
            const parsedBody = Buffer.concat(body).toString();
            console.log(parsedBody);
            const message = parsedBody.split('=')[1];
            fs.writeFileSync('message.txt', message); // now we will get a file with our text in it
            res.statusCode = 302;
            res.setHeader('Location', '/');
            return res.end();
        })
    } 
    res.setHeader('Content-Type', 'text/html');
    res.write('<html>'); 
    res.write('<head><title>My First Page</title></head>');
    res.write('<body><h1>Hello world!</h1></body>')
    res.write('</html>')
    res.end();

```

## Blocking and Non-Blocking Code

writeFileSync() is a method that blocks code execution until the file is created. If we are writing a huge file, it would slow down our server. We should use
writeFile(). It contains a third argument, a callback, which can handle any errors. Refactored: 

```js
fs.writeFile('message.txt', message, err => {
                res.statusCode = 302;
                res.setHeader('Location', '/');
                return res.end();
            });
```

## Node.js - Looking Behind the Scenes

#### Single Thread, Event Loop & Blocking Code

https://nodejs.dev/learn/the-nodejs-event-loop

## Using the Node Modules System

We should refactor our code to split up our logic. We do this by making multiple files for each segment of logic. For example, a routes.js file for all the routes.
There are two ways to then wire it up from there. Now, our routes.js folder looks like this:

```js
const requestHandler = (req, res) => {
  const url = req.url;
  const method = req.method;
  if (url === "/") {
    res.write("<html>");
    res.write("<head><title>Enter Message</title></head>");
    res.write(
      '<body><form action="/message" method="POST"><input name="message"><button type="submit">Send</button></form></body>'
    );
    res.write("</html>");
    res.end();
    return res.end();
  }

  if (url === "/message" && method === "POST") {
    const body = [];
    req.on("data", (chunk) => {
      console.log(chunk);
      body.push(chunk);
    });

    return req.on("end", () => {
      const parsedBody = Buffer.concat(body).toString();
      console.log(parsedBody);
      const message = parsedBody.split("=")[1];
      fs.writeFile("message.txt", message, (err) => {
        res.statusCode = 302;
        res.setHeader("Location", "/");
        return res.end();
      });
    });
  }
  res.setHeader("Content-Type", "text/html");
  res.write("<html>");
  res.write("<head><title>My First Page</title></head>");
  res.write("<body><h1>Hello world!</h1></body>");
  res.write("</html>");
  res.end();
};

module.exports = requestHandler;
```

And then import in app.js:
```js
const http = require('http');
const routes = require('./routes');

const server = http.createServer(routes);

server.listen(3000);

```

Can also export as an object for acessing multiple properties within the file:
```js
module.exports = {
  handler: requestHandler,
  someText: 'Some hard coded text'
}
```

