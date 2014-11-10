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

2. Signed Request - Part One
---

In LiveStax, to gain more information on your app, e.g. the current instance, whether the user is an admin, etc, the app needs to use POST. Once it does, LiveStax will POST to it rather using the usual GET request, within the POST request, a `signed_request` parameter is also sent. In Express, we need to use another package to parse the incoming request body, [Body Parser](https://github.com/expressjs/body-parser).

```shell
npm install body-parser --save
```

Firstly, we'll need to require the Body Parser and tell the app to use it using the `urlencoded` scheme:

```javascript
var express, app, bodyParser;
...
bodyParser  = require('body-parser');
...
app.use(bodyParser.urlencoded( {
  extended: true
}));
```

Now that the app will be able to parse the incoming request, all we need to do is set it to a variable so that we can use it.

```javascript
var signedRequest = request.body.signed_request;
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/8eaa34ffb00eb3621e10fb511ac97d7fbe57a850)

3. Signed Request - Part Two
---

The `signed_request` parameter that we have parsed is in fact a [JSON Web Token](http://jwt.io) that needs to be decoded. In LiveStax, each signed request is encoded using the app's secret, so that we can verify the signed request. As we will need this app secret to verify the token, we are going to use another package to store the app secret. So let's install [dot-env](https://github.com/supershabam/dot-env):

```shell
npm install dot-env --save
```

dot-env works by looking for a `.env.json` file and merging it into `process.env`, therefore lets create the file (replace "TODO" with your own app secret):

```javascript
{
  "APP_SECRET": "TODO"
}
```

We need to ensure that dot-env is started as soon as possible, so at the top of `app.js` let's add the following line:

```javascript
require('dot-env');
```

So, now we can use the environment variables set in our `.env.json`. Next we need to decode the signed request, which we'll do by using the recommended package by the team behind [JWT](http://jwt.io), [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken):

```shell
npm install jsonwebtoken --save
```

Now we can use the package to verify the signed request using the `verify` function:

```javascript
var express, app, bodyParser, jwt, appSecret;
...
jwt = require('jsonwebtoken');
appSecret = process.env.APP_SECRET;
...
var signedRequest = request.body.signed_request;
jwt.verify(signedRequest, appSecret, function(err, decoded) {
});
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/8dd19f94aa9fd5e144fc8e62a0dd4b5831265acc)