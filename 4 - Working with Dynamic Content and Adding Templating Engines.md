## Sharing Data Across Requests and Users

Data can be shared across files. In admin.js, we tweak our file to export two objects: router and products.
![image](https://user-images.githubusercontent.com/104043093/169668419-3c24bca0-be4a-40bd-8533-57389ce60662.png)

In app.js, we tweak our middleware function: ```js app.use('/admin', adminData.routes); //accessing the adminData routes object ```

And in shop.js, we console.log() the exported products object contained in admin.js. As we can see, data can be shared between files.

![image](https://user-images.githubusercontent.com/104043093/169668454-52a8f618-277c-4dfd-bf5a-bb7d8e34b447.png)




## Templating Engines - An Overview

## Installing and Implementing Pug

## Outputting Dynamic Content

## Converting HTML Files to Pug

## Adding a Layout

## Finishing the Pug Template

## Working with Handlebars

## Converting our Project to Handlebars

## Adding Layout to Handlebars

## Working with EJS

## Working on the Layout with Partials

## Wrap Up

