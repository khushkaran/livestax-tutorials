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

```diff
+var express, app;
+express = require('express');
+app = express();
```

Now lets configure the app's settings such as, port, view engine and where the static files are located.

```diff
...
app = express();
+app.set('port', (process.env.PORT || 5000));
+app.set('view engine', 'ejs');
+app.use(express.static(__dirname + '/public'));
```

Here, the port of this app is configured to either the environment variable named `PORT` or 5000 if there is no such variable set. The view engine is set to "ejs", the templating engine that we installed earlier and the static files directory is set to "/public".

```diff
...
app.use(express.static(__dirname + '/public'));
+app.all('/', function(request, response) {
+  response.render('index');
+});

+app.listen(app.get('port'));
```

We now have created a route at "/" that accepts both GET and POST requests and we will render the "index" view. Also the app will listen for connections on the app port we set earlier.

The last thing we need to do is to move our static "js" directory into a new "public" directory and move our index.html into a new "views" directory and change the extension to ".ejs". Now we can use the command `node app.js` and visit [http://localhost:5000](http://localhost:5000) to see our app.

2. Signed Request - Part One
---

In Livestax, to gain more information on your app, e.g. the current instance, whether the user is an admin, etc, the app needs to use POST. Once it does, Livestax will POST to it rather using the usual GET request, within the POST request, a `signed_request` parameter is also sent. In Express, we need to use another package to parse the incoming request body, [Body Parser](https://github.com/expressjs/body-parser).

```shell
npm install body-parser --save
```

Firstly, we'll need to require the Body Parser and tell the app to use it using the `urlencoded` scheme:

```diff
-var express, app;
+var express, app, bodyParser;
...
+bodyParser  = require('body-parser');
...
+app.use(bodyParser.urlencoded( {
+  extended: true
+}));
...
```

Now that the app will be able to parse the incoming request, all we need to do is set it to a variable so that we can use it.

```diff
...
app.all('/', function(request, response) {
+ var signedRequest = request.body.signed_request;
  response.render('index');
});
...
```

3. Signed Request - Part Two
---

The `signed_request` parameter that we have parsed is in fact a [JSON Web Token](http://jwt.io) that needs to be decoded. In Livestax, each signed request is encoded using the app's secret, so that we can verify the signed request. As we will need this app secret to verify the token, we are going to use another package to store the app secret. So let's install [dot-env](https://github.com/supershabam/dot-env):

```shell
npm install dot-env --save
```

dot-env works by looking for a `.env.json` file and merging it into `process.env`, therefore lets create the file (replace "TODO" with your own app secret):

```diff
+{
+  "APP_SECRET": "TODO"
+}
```

We need to ensure that dot-env is started as soon as possible, so at the top of `app.js` let's add the following line:

```diff
+require('dot-env');
...
```

So, now we can use the environment variables set in our `.env.json`. Next we need to decode the signed request, which we'll do by using the recommended package by the team behind [JWT](http://jwt.io), [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken):

```shell
npm install jsonwebtoken --save
```

Now we can use the package to verify the signed request using the `verify` function:

```diff
...
-var express, app, bodyParser;
+var express, app, bodyParser, jwt, appSecret;
...
+jwt = require('jsonwebtoken');
+appSecret = process.env.APP_SECRET;
...
+var signedRequest = request.body.signed_request;
+jwt.verify(signedRequest, appSecret, function(err, decoded) {
+});
```

4. Communicate Signed Request
---

The signed request token is made up of a series of key-value pairs, for more information on the contents of it visit the [Livestax Documentation](http://developers.livestax.com/v0.2.0/docs/signed-request). To save data unique to the instance of the app in question, we are concerned with the `instance_id`. To enable use of the instance ID, we need to make the token available to the template and consquently, it will be available to JavaScript, which can in turn, be decoded as it contains the instance ID.

Firstly, let's render the token to the template, we need to move the `response.render()` function to within the `jwt.verify()` function. This will ensure that the page is only rendered if the token has been verified.

```diff
...
jwt.verify(signedRequest, appSecret, function(err, decoded) {
+ response.render('index', {signed_request: signedRequest});
});
...
```

To display the passed data, we need to add the variable to the template, here we will add it to a `signed-request` data attribute to the body tag.

```diff
...
</head>
+<body data-signed-request="<%= signed_request  %>">
<div class="col-md-12">
...
```

Finally, let's capture the signed request token in the JavaScript so that we can use it at a later date.
```diff
$(document).ready(function() {
+ var signedRequest = $("body").data("signed-request");
  Livestax.on("pet-finder.newpet", function(petName) {
  ...
  };
  ...
};
```

5. POST Required Data
---

In order to save the data from the history app, we need to provide that data to the server, to do this we will POST the data we require; the pet name and the signed request token. So in "main.js", when a name is received, let's POST the data:

```diff
...
Livestax.on("pet-finder.newpet", function(petName) {
...
+ $.post("/addtohistory", {pet_name: petName, signed_request: signedRequest});
});
...
```

Here, we have provided the data required and as per the code need a new route called `/addtohistory`, so let's add that to "app.js".

```diff
...
app.all('/', function(request, response) {
...
});

+app.post('/addtohistory', function(request, response) {
+});

app.listen(app.get('port'));
```

Finally, let's store that received pet name and signed request token in variables so that we can use them in the future.

```diff
...
app.post('/addtohistory', function(request, response) {
+ var petName, signedRequest;
+ petName = request.body.pet_name;
+ signedRequest = request.body.signed_request;
});
...
```

6. Save History
---

We are going to use [Redis](http://redis.io) as the data store for the history data, it is a very simple key-value store. As noted in the introduction of this tutorial, we need to install Redis onto your local machine, instructions can be found on their [website](http://redis.io/download). After this, we need to install a Redis client, a tool that enables our server to talk to the database. On the Redis [website](http://redis.io/clients), there are many provided, we will use the one recommended for Node.js, ["node_redis"](https://github.com/mranney/node_redis). In its repository's README, the instructions provide the package required, however, also state that using a supplementary package, "hiredis", is faster, so that is what we will do:

```shell
npm install hiredis redis --save
```

To connect to Redis, we will use a URL, usually your localhost address (127.0.0.1) and your port (default: 6379), so let's add this variable to the ".env" file:

```diff
...
"APP_SECRET": "TODO",
+"REDIS_URL": "http://127.0.0.1:6379"
...
```

Now, in your "app.js", lets assign the Redis client to a variable using the `createClient()` function with the Redis port and hostname. We will also need to ensure we are using the correct port, this can have issues when using different services such as [RedisToGo](http://redistogo.com/), so we will use the second colon to split the address:

```diff
...
-var express, app, bodyParser, jwt, appSecret;
+var express, app, bodyParser, jwt, appSecret, redisURL, redis;
express = require('express');
...
appSecret = process.env.APP_SECRET;
+redisURL = require('url').parse(process.env.REDIS_URL);
+redis = require('redis').createClient(redisURL.port, redisURL.hostname);
+if (redisURL.auth) {
+  redis.auth(redisURL.auth.split(":")[1]);
+}
app.set('port', (process.env.PORT || 5000));
...
```

Before we save any data, we want to ensure that there is a pet name passed (not NULL or an empty string):

```diff
...
app.post('/addtohistory', function(request, response) {
  ...
  +if (petName) {
  +};
});
```

Now as before, we want to verify the signed request token with the app secret to get our instance ID:

```diff
...
app.post('/addtohistory', function(request, response) {
- var petName, signedRequest;
+ var petName, signedRequest, instanceID;
  ...
  if (petName) {
+   jwt.verify(signedRequest, appSecret, function(err, decoded) {
+     instanceID = decoded.instance_id;
+   });
  };
});
```

Finally, we will use [Redis lists](http://redis.io/commands#list) to save the history of pet names. The `LPUSH` command in Redis prepends an element to a list of the name provided. If there is no list, it will create one and then add the element to it.

```diff
...
jwt.verify(signedRequest, appSecret, function(err, decoded) {
  instanceID = decoded.instance_id;
+  redis.lpush(instanceID, petName);
});
...
```

7. Render History
---

Now that the history has been saved, we want that history to be visible to us, so we want to render the history to the view. Firstly, we want to get the stored pet names from Redis using the `LRANGE` command within [Redis lists](http://redis.io/commands#list) then pass it to the view. To display all the records within a list, we use the beginning index "0" and the last index "-1".

```diff
...
jwt.verify(signedRequest, appSecret, function(err, decoded) {
+  redis.lrange(decoded.instance_id, 0, -1, function(err, petNameList) {
+    response.render('index', {signed_request: signedRequest, pet_names: petNameList});
+  });
});
...
```

The ejs templating engine allows us to use JavaScript within our view, e.g. iterate over arrays, if/else statements, and we can utilize this to present the data to the user. In ejs, JavaScript statements are written within `<%` and `%>` and to directly render the result of a statement `<%=` and `%>`. Firstly, we want to add an iterator of pet names within an `if` statement, on whether there are pet names, inside the list-group div.

```diff
...
<div class="list-group js-pet-names">
+ <% if (pet_names.length > 0) { %>
+   <% for(var i=0; i<pet_names.length; i++) { %>
+     <a href="#" class="list-group-item"><%= pet_names[i] %></a>
+   <% } %>
+ <% } %>
</div>
...
```

So now we can see our history from Redis, however, if you reload the app, we see both the notice and the history. We can fix this with some JavaScript in our "main.js", it will hide the notice if there is history displayed:

```diff
...
var signedRequest = $("body").data("signed-request");
+if ($(".js-pet-names a").length > 0) {
+  $(".notice").hide();
+};
Livestax.on("pet-finder.newpet", function(petName) {
...
});
...
```

8. Clear History
---

Livestax provides a menu in the top right of your app, however, you may want to add additional item to it. Livestax provides a way of doing this using the `Livestax.menu.set()` function, to read more on this visit the [Livestax documentation](http://developers.livestax.com/v0.2.0/docs/app-menu). First let's set the menu item called "Clear History" with an eraser icon in our "main.js":

```diff
...
Livestax.on("pet-finder.newpet", function(petName) {
...
});
+Livestax.menu.set("Clear History", "eraser", function() {
+});
$(document.body).on("click", ".js-pet-names a", function(event) {
...
});
...
```

Note: "eraser" is a icon provided by [FontAwesome](http://fortawesome.github.io/Font-Awesome/icons/), all you have to do is provide the name minus the preceding "fa-", so "fa-eraser" becomes "eraser".

Now we place our logic within the function, we want the following things to happen:

* clear the history for our instance
* remove all the names in the history app
* show the empty history notice

```diff
...
Livestax.menu.set("Clear History", "eraser", function() {
+ $.post("/clearhistory", {signed_request: signedRequest});
+ $(".js-pet-names a").remove();
+ $(".notice").show();
});
...
```

The code above stipulates we need to have a POST route called `/clearhistory` in our "app.js", so let's add it:

```diff
...
app.post('/addtohistory', function(request, response) {
...
});

+app.post('/clearhistory', function(request, response) {
+});

app.listen(app.get('port'));
```

Finally, we need the instance ID to clear the correct history and we'll use the Redis `DEL` [function](http://redis.io/commands/del) to delete the data.

```diff
...
app.post('/clearhistory', function(request, response) {
+ var signedRequest, instanceID;

+ signedRequest = request.body.signed_request;
+ jwt.verify(signedRequest, appSecret, function(err, decoded) {
+   instanceID = request.body.instance_id;
+   redis.del(instanceID);
+ });
});
...
```

9. Confirmation
---

We are now successfully clearing the history, however, as this is a destructive action, it would be helpful to provide the users with a confirmation message to confirm whether or not they are sure. There are two methods of achieving this, either the [Livestax Dialog API](http://developers.livestax.com/v0.2.0/docs/dialogs) or the [Livestax Flash API](http://developers.livestax.com/v0.2.0/docs/flash-messages). Both methods are described below.

To reduce the duplication of code, we will extract the clear history functionality into its own function.

```diff
...
+var clearHistory = function() {
+ $.post("/clearhistory", {signed_request: signedRequest});
+ $(".js-pet-names a").remove();
+ $(".notice").show();
+};

-Livestax.menu.set("Clear History", "eraser", function() {
- $.post("/clearhistory", {signed_request: signedRequest});
- $(".js-pet-names a").remove();
- $(".notice").show();
-});
+Livestax.menu.set("Clear History", "eraser", clearHistory);
...
```



## Dialog

Firstly, we need to create the dialog data that will be passed through to the API call.

```diff
...
-Livestax.menu.set("Clear History", "eraser", clearHistory);
+Livestax.menu.set("Clear History", "eraser", function() {
+ var dialogData = {
+   title: "Are you sure?",
+   message: "Are you sure you want to clear your history? This is an irreversible action and cannot be undone.",
+   buttons: [
+     {
+       title: "Yes",
+       callback: clearHistory
+       type: "danger"
+     },
+     {
+       title: "Cancel",
+       callback: function(){},
+       type: "ok"
+     }
+   ]
+ };
+});
...
```

Our `dialogData` object contains the title, message and the buttons we want the dialog to display. So now our `clearHistory` function is called on the click of "Yes". Now that our data is ready, all we need to do is show the dialog using the `show()` function.

```diff
...
Livestax.menu.set("Clear History", "eraser", function() {
  var dialogData = {
    ...
  };
+ Livestax.dialog.show(dialogData);
});
...
```

## Flash

Firstly, we need to create the flash data that will be passed through to the API call.

```diff
...
-Livestax.menu.set("Clear History", "eraser", clearHistory);
+Livestax.menu.set("Clear History", "eraser", function() {
+ var flashData = {
+   message: "Sure you want to clear your history?",
+   dismiss: function() {},
+   confirm: clearHistory
+ };
+});
...
```

Our `flashData` object contains the message and the buttons we want the flash message to display. So now our `clearHistory` function is called on the confirm of the flash message. Now that our data is ready, all we need to do is show the flash message using the `danger()` function, as the result will be destructive. It is important to note, that there are other types available.

```diff
...
Livestax.menu.set("Clear History", "eraser", function() {
  var flashData = {
    ...
  };
+ Livestax.flash.danger(flashData);
});
...
```
