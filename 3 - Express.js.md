## Installing Express.js

```npm install --save express```

```js
const http = require('http');

const express = require('express');
const app = express();

const server = http.createServer(app);
server.listen(3000);
```

## Adding Middleware

Middleware means an incoming request is automatically funneled through Express.js functions. Instead of having one request handler, you will have the option of hooking in a bunch of functions which the request will go through until you send a response. This allows code to be split into multiple blocks/pieces. 

```js
const http = require('http');

const express = require('express');

const app = express();

// allows us to add a new middleware function, accepts an array of request handlers. You can use it to pass one function to handle every incoming req
/**  next() is a function that will be passed to the anonymous function in the use method. next() has to be executed to allow the req to travel to the next middleware*/
app.use((req, res, next) => {
    console.log('In the middleware! ');
    next(); // called to allow the req to travel to the next middleware in line (the use method after this one)
}); 

app.use((req, res, next) => {
    console.log('In another middleware! ');
}); 

const server = http.createServer(app);

server.listen(3000);
```

## How Middleware Works

next() should be called if we don't send a response, otherwise it will just die. 

```js
const http = require('http');

const express = require('express');

const app = express();

// allows us to add a new middleware function, accepts an array of request handlers. You can use it to pass one function to handle every incoming req
/**  next() is a function that will be passed to the anonymous function in the use method. next() has to be executed to allow the req to travel to the next middleware*/
app.use((req, res, next) => {
    console.log('In the middleware! ');
    next(); // called to allow the req to travel to the next middleware in line (the use method after this one)
}); 

app.use((req, res, next) => {
    console.log('In another middleware! ');
    //instead of res.setHeader() and res.write(), we can use send(), which can send a body of any type. It autosets html as the content type in this case. 
    // you can still set your own custom header if you'd like. 
    res.send('<h1>Hello from Express</h1>');
}); 

const server = http.createServer(app);

server.listen(3000);
```

## Express.js - Looking Behind the Scenes

https://github.com/expressjs/express/blob/master/lib/response.js

```js
const express = require('express');

const app = express();

// allows us to add a new middleware function, accepts an array of request handlers. You can use it to pass one function to handle every incoming req
/**  next() is a function that will be passed to the anonymous function in the use method. next() has to be executed to allow the req to travel to the next middleware*/
app.use((req, res, next) => {
    console.log('In the middleware! ');
    next(); // called to allow the req to travel to the next middleware in line (the use method after this one)
}); 

app.use((req, res, next) => {
    console.log('In another middleware! ');
    //instead of res.setHeader() and res.write(), we can use send(), which can send a body of any type. It autosets html as the content type in this case. 
    // you can still set your own custom header if you'd like. 
    res.send('<h1>Hello from Express</h1>');
}); 

// const server = http.createServer(app);

// server.listen(3000);

// can be replaced with app.listen()

app.listen(3000);
```

## Handling Different Routes

```js
const express = require('express');

const app = express();

// the request goes through the code top to bottom. We don't have a next() here because if the route matches, we don't want it to go to the next middleware. 
// The order matters. 
app.use('/add-product', (req, res, next) => {
    console.log('In another middleware! ');
    res.send('<h1>The "Add Product" Page</h1>');
}); 

app.use('/', (req, res, next) => {
    console.log('In another middleware! ');
    res.send('<h1>Hello from Express</h1>');
}); 

app.listen(3000);
```
## Parsing Incoming Requests

``` npm install --save body-parser```

```js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();

app.use(bodyParser.urlencoded({extended: false})) // registers a middleware, will call next to reach our middleware. Before it does that though, it does all the request body
// parsing that we used to do manually in Node.js

app.use('/add-product', (req, res, next) => {
    res.send('<form action="/product" method="POST"><input name="title"><button type="submit">Add Product</button></form>');
}); 

app.use('/product', (req, res, next) => {
    console.log(req.body); // by default, Express doesn't try to parse the incoming body. A request parser needs to be registered. Returns a JS object { title: 'inputted text'}
    res.redirect('/');
})

app.use('/', (req, res, next) => {
    res.send('<h1>Hello from Express</h1>');
}); 

app.listen(3000);
```

## Limiting Middleware Execution to POST Requests

Currently. app.use('/product') executes for GET requests as well. Refactor to: 

```js
app.post('/product', (req, res, next) => {
    console.log(req.body); 
    res.redirect('/');
})
```
## Using Express Router

We want to split our routing code over multiple files. 

![image](https://user-images.githubusercontent.com/104043093/165405699-83dfe899-06ed-4d79-8dea-c2c722481d88.png)

#### admin.js

![image](https://user-images.githubusercontent.com/104043093/165405772-324bc796-bca1-49ad-aa17-97275a8494c2.png)

#### shop.js

![image](https://user-images.githubusercontent.com/104043093/165405823-f11b7eaa-90bc-4247-9e88-d60053032f43.png)

#### app.js

![image](https://user-images.githubusercontent.com/104043093/165405895-e369579c-585e-4cef-8166-4ec2c22da21a.png)

## Adding a 404 Error Page

Inside of app.js...
```js
app.use(adminRoutes);
app.use(shopRoutes);

// set up a catch all middleware at the end that handles erroneous url paths. Will only be executed if no other middleware handles the req (which it shouldn't)
// don't need a path filter, since '/' is the default path anyways
app.use((req, res, next) => {
    res.status(404).send('<h1>Page not found</h1>');
});
```

## Filtering Paths

```js
app.use('/admin',adminRoutes);
```
Now, all the routes in admin.js will first be filtered with the /admin path, and now the /admin path is 'shunted' in front of all the routes in admin.js. 

## Creating HTML Pages

