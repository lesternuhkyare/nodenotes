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

## Creating/Serving HTML Pages

In shop.js...
```js
const path = require('path');
const express = require('express');

const router = express.Router();

router.get('/', (req, res, next) => {
  res.sendFile(path.join(__dirname, '../', 'views', 'shop.html')); // returns a path by concatenating the different segments, path.join adds the '/' for you (works on all OS) 
  // dirname holds the absolute path on our OS to this folder
});

module.exports = router;
```
In shop.html...
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Add Product</title>
</head>
<body>
    <header>
        <nav>
            <ul>
                <li><a href="/">Shop</a></li>
                <li><a href="/add-product">Add Product</a></li>
            </ul>
        </nav>
    </header>
    <main>
        <h1>My Products</h1>
        <p>List of all the products...</p>
    </main>
</body>
</html>
```

## Serving Static Files

Serving files statically simple means not handled by the Express router (or other middleware), but instead directly forwarded to the file system. 

In app.js:

![image](https://user-images.githubusercontent.com/104043093/166815388-1f85a826-2923-4b00-b7cd-6d76824c00b5.png)

In add-product.html:

![image](https://user-images.githubusercontent.com/104043093/166815473-397008b4-33e4-47c6-bb09-5599fa03a0c3.png)

File structure:

![image](https://user-images.githubusercontent.com/104043093/166815516-cf7dfc65-5f7e-4c51-b714-e693d6627e44.png)

## Module Summary

Middleware - next() and res()

Middleware functions can be added with the use() method. Middleware functions handle a request and should either call next() or res(). 

Routing

You can filter requests by path and method. If you filter by method, paths are matched exactly, otherwise, the first segment of a URL is matched. You can use express.Router to split your routes across files elegantly. 

Serving Files

Can use sendFile() to your users, e.g and HTML file. If a request is directly made for a file, e.g a css file, you can enable static serving for such files using express.static(). 

