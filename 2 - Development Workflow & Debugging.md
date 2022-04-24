## Understanding npm Scripts

npm - node package manager (shipped with node.js)

'npm init' can be ran in the terminal, which will create a package.json file in your project. It is a configuration file. Within the file, there is a scripts configuration, and you can add your own scripts here. A 'start' script can be added with the json notation: ```"start": "node app-js" ```. Then, npm start can be ran to run your node server. 'start' is common practice in industry so that when the code is shared, users don't have to search for the entry file. This is just an example. 

'start' is a reserved script name. For custom scripts, you must run npm run 'script_name'. 

## Installing 3rd Party Packages

``` npm install package_name ```

ex: ``` npm install nodemon ```

## Using Nodemon for Autorestarts

Modify the start script to ```"start": "nodemon app.js" ```

*note (from course), you can share projects without the node_modules folder (where they are stored) and you can run npm install in a project to then re-create that node_modules folder. This allows you to share only your source code, hence reducing the size of the shared project vastly. (This works because install reads from your packages.json file and installs all the dependencies in it).
The attached course code snippets also are shared in that way, hence you need to run npm install in the extracted packages to be able to run my code!
I showed that nodemon app.js would not work in the terminal or command line because we don't use local dependencies there but global packages.
You could install nodemon globally if you wanted (this is NOT required though - because we can just run it locally): npm install -g nodemon would do the trick. Specifically the -g flag ensures that the package gets added as a global package which you now can use anywhere on your machine, directly from inside the terminal or command prompt.*

