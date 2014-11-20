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

4. Communicate Signed Request
---

The signed request token is made up of a series of key-value pairs, for more information on the contents of it visit the [LiveStax Documentation](https://github.com/livestax/docs#signed-request). To save data unique to the instance of the app in question, we are concerned with the `instance_id`. To enable use of the instance ID, we need to make the token available to the template and consquently, it will be available to JavaScript, which can in turn, be decoded as it contains the instance ID.

Firstly, let's render the token to the template, we need to move the `response.render()` function to within the `jwt.verify()` function. This will ensure that the page is only rendered if the token has been verified.

```javascript
response.render('index', {signed_request: signedRequest});
```

To display the passed data, we need to add the variable to the template, here we will add it to a `signed-request` data attribute to the body tag.

```javascript
<body data-signed-request="<%= signed_request  %>">
```

Finally, let's capture the signed request token in the JavaScript so that we can use it at a later date.
```javascript
var signedRequest = $("body").data("signed-request");
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/d853a9a021fa18070981538988262c551454b828)

5. POST Required Data
---

In order to save the data from the history app, we need to provide that data to the server, to do this we will POST the data we require; the pet name and the signed request token. So in "main.js", when a name is received, let's POST the data:

```javascript
...
$.post("/addtohistory", {pet_name: petName, signed_request: signedRequest});
...
```

Here, we have provided the data required and as per the code need a new route called `/addtohistory`, so let's add that to "app.js".

```javascript
app.post('/addtohistory', function(request, response) {
});
```

Finally, let's store that received pet name and signed request token in variables so that we can use them in the future.

```javascript
...
var petName, signedRequest;
petName = request.body.pet_name;
signedRequest = request.body.signed_request;
...
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/abb4e90429222fa98f59c01082d43926a7396cd2)

6. Save History
---

We are going to use [Redis](http://redis.io) as the data store for the history data, it is a very simple key-value store. As noted in the introduction of this tutorial, we need to install Redis onto your local machine, instructions can be found on their [website](http://redis.io/download). After this, we need to install a Redis client, a tool that enables our server to talk to the database. On the Redis [website](http://redis.io/clients), there are many provided, we will use the one recommended for Node.js, ["node_redis"](https://github.com/mranney/node_redis). In its repository's README, the instructions provide the package required, however, also state that using a supplementary package, "hiredis", is faster, so that is what we will do:

```shell
npm install hiredis redis --save
```

To connect to Redis, we will use a URL, usually your localhost address (127.0.0.1) and your port (default: 6379), so let's add this variable to the ".env" file:

```javascript
...
"REDIS_URL": "http://127.0.0.1:6379"
...
```

Now, in your "app.js", lets assign the Redis client to a variable using the `createClient()` function with the Redis port and hostname. We will also need to ensure we are using the correct port, this can have issues when using different services such as [RedisToGo](http://redistogo.com/), so we will use the second colon to split the address:

```javascript
...
var express, app, bodyParser, jwt, appSecret, redisURL, redis;
...
redisURL = require('url').parse(process.env.REDIS_URL);
redis = require('redis').createClient(redisURL.port, redisURL.hostname);
if (redisURL.auth) {
  redis.auth(redisURL.auth.split(":")[1]);
}
...
```

Before we save any data, we want to ensure that there is a pet name passed (not NULL or an empty string):

```javascript
...
var petName, signedRequest;
petName = request.body.pet_name;
signedRequest = request.body.signed_request;
if (petName) {
};
```

Now as before, we want to verify the signed request token with the app secret to get our instance ID:

```javascript
var petName, signedRequest, instanceID;
...
if (petName) {
  jwt.verify(signedRequest, appSecret, function(err, decoded) {
  instanceID = decoded.instance_id;
  });
}
```

Finally, we will use [Redis lists](http://redis.io/commands#list) to save the history of pet names. The `LPUSH` command in Redis prepends an element to a list of the name provided. If there is no list, it will create one and then add the element to it.

```javascript
...
redis.lpush(instanceID, petName);
...
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/fd9072b57de1d2d5c530d7f2ebcde08d62f64e1d)

7. Render History
---

Now that the history has been saved, we want that history to be visible to us, so we want to render the history to the view. Firstly, we want to get the stored pet names from Redis using the `LRANGE` command within [Redis lists](http://redis.io/commands#list) then pass it to the view. To display all the records within a list, we use the beginning index "0" and the last index "-1".

```javascript
redis.lrange(decoded.instance_id, 0, -1, function(err, petNameList) {
  response.render('index', {signed_request: signedRequest, pet_names: petNameList});
});
```

The ejs templating engine allows us to use JavaScript within our view, e.g. iterate over arrays, if/else statements, and we can utilize this to present the data to the user. In ejs, JavaScript statements are written within `<%` and `%>` and to directly render the result of a statement `<%=` and `%>`. Firstly, we want to add an iterator of pet names within an `if` statement, on whether there are pet names, inside the list-group div.

```javascript
...
<% if (pet_names.length > 0) { %>
  <% for(var i=0; i<pet_names.length; i++) { %>
    <a href="#" class="list-group-item"><%= pet_names[i] %></a>
  <% } %>
<% } %>
...
```

So now we can see our history from Redis, however, if you reload the app, we see both the notice and the history. We can fix this with some JavaScript in our "main.js", it will hide the notice if there is history displayed:

```javascript
...
if ($(".js-pet-names a").length > 0) {
  $(".notice").hide();
};
...
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/8aec1c0e0c8d566a88b3b4f6cbdbf23d7f0c6c37)

8. Clear History
---

LiveStax provides a menu in the top right of your app, however, you may want to add additional item to it. LiveStax provides a way of doing this using the `Livestax.menu.set()` function, to read more on this visit the [LiveStax documentation](https://github.com/livestax/docs#menu). First let's set the menu item called "Clear History" with an eraser icon in our "main.js":

```javascript
Livestax.menu.set("Clear History", "eraser", function() {
});
```

Note: "eraser" is a icon provided by [FontAwesome](http://fortawesome.github.io/Font-Awesome/icons/), all you have to do is provide the name minus the preceding "fa-", so "fa-eraser" becomes "eraser".

Now we place our logic within the function, we want the following things to happen:

* clear the history for our instance
* remove all the names in the history app
* show the empty history notice

```javascript
...
$.post("/clearhistory", {signed_request: signedRequest});
$(".js-pet-names a").remove();
$(".notice").show();
...
```

The code above stipulates we need to have a POST route called `/clearhistory` in our "app.js", so let's add it:

```javascript
app.post('/clearhistory', function(request, response) {
});
```

Finally, we need the instance ID to clear the correct history and we'll use the Redis `DEL` [function](http://redis.io/commands/del) to delete the data.

```javascript
...
var signedRequest, instanceID;

signedRequest = request.body.signed_request;
jwt.verify(signedRequest, appSecret, function(err, decoded) {
  instanceID = request.body.instance_id;
  redis.del(instanceID);
});
...
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/5ef5a509203ab174fb4e4158ed390595b37abbfd)

9. Confirmation Dialog
---

We are now successfully clearing the history, however, as this is a destructive action, it would be helpful to provide the users with a dialog to confirm whether or not they are sure. We can do this very simply using the `Livestax.dialog.show()` function, further information is available in the [LiveStax documentation](https://github.com/livestax/docs#dialogs).

Firstly, we need to create the dialog data that will be passed to the `show()` function:

```javascript
var dialogData = {
  title: "Are you sure?",
  message: "Are you sure you want to clear your history? This is an irreversible action and cannot be undone.",
  buttons: [
    {
      title: "Yes",
      callback: function(){
        $.post("/clearhistory", {signed_request: signedRequest});
        $(".js-pet-names a").remove();
        $(".notice").show();
      },
      type: "danger"
    },
    {
      title: "Cancel",
      callback: function(){},
      type: "ok"
    }
  ]
};
```

Our `dialogData` object contains the title, message and the buttons we want the dialog to display. Notice our functions that cleared the history have moved into a function in the `callback` of the yes button. Now that our data is ready, all we need to do is show the dialog using the `show()` function.

```javascript
...
Livestax.dialog.show(dialogData);
...
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/2aaa7e09a78f86d68cc6d9a8a8bee58df2975458)