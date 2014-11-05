Store Data
===

In this tutorial we will improve the Pet Finder History app by allowing it to store
the history. At the moment, every time the app is reloaded, the previous history is
lost as the data is not stored anywhere. For the purposes of this tutorial, we will
use [Node.js](http://nodejs.org/) as the server-side technology and [Redis](http://redis.io)
as the Data store, however, this is only one of many suitable options. To follow
along you will need to install these two technologies to your personal computer,
[Node.js](http://nodejs.org/download/) has a very simple installer which can be
obtained by visiting their website. Redis is installed following the instructions
on their [website](http://redis.io/download), they also have a link to a Windows
port of Redis maintained by the Microsoft Open Tech group.

1. Convert to Node.js App
---

After you have installed Node.js, you will have a command available to you in your terminal called "npm", this is short for the [Node Package Manager](https://npmjs.org/), it allows you to keep track of any dependencies that your app uses. To allow us to easily setup a lightweight web application, we will install [Express](http://expressjs.com/) using the "npm" command. In the terminal type in the following command:

```shell
npm install express --save
```

This command will instruct NPM to install express onto your machine and also save a reference to it into a file called `package.json`. `package.json` should include all the dependencies required to run your app wherever it may be, on whatever machine using the `npm install` command. Another package we will require is "ejs", a templating engine that will help us bring together HTML and Node.js.

```shell
npm install ejs --save
```

To start our Node.js app we will create a new file called `app.js` and add the code to create an instance of express and set it to the variable `app`.

```javascript
var express, app;
express = require('express');
app = express();
```

Now lets configure the app's settings such as, port, view engine and where the static files are located.

```javascript
app.set('port', (process.env.PORT || 5000));
app.set('view engine', 'ejs');
app.use(express.static(__dirname + '/public'));
```

Here, the port of this app is configured to either the environment variable named `PORT` or 5000 if there is no such variable set. The view engine is set to "ejs", the templating engine that we installed earlier and the static files directory is set to "/public".

```javascript
app.all('/', function(request, response) {
  response.render('index');
});

app.listen(app.get('port'));
```

We now have created a route at "/" that accepts both GET and POST requests and we will render the "index" view. Also the app will listen for connections on the app port we set earlier.

The last thing we need to do is to move our static "js" directory into a new "public" directory and move our index.html into a new "views" directory and change the extension to ".ejs". Now we can use the command `node app.js` and visit [http://localhost:5000](http://localhost:5000) to see our app.

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/e46e76310c91ba165cbd33968d7c0723dc9b7c57)
