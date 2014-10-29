Trigger Data
===

In this tutorial we will build upon the Pet Finder app and allow it to communicate
to another app, the Pet Finder History. As it’s name suggests, it will contain a
list of pet names that have been recently searched for.

1. Create a LiveStax App
---

As we did in the previous app, we will start off by creating a skeleton LiveStax
app with the LiveStax JavaScript and the LiveStax Theme from the CDN.

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="http://livestax.com/assets/livestax-0.1.0.js"></script>
    <link href="http://dz44vc6ose3il.cloudfront.net/theme.css" rel="stylesheet" type="text/css" media="all">
  </head>
  <body>
  </body>
</html>
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/372e570736044976fc041389e5e3c8df4c01e2f5)

2. Communicate Data - Part One
---

We will use LiveStax to trigger a named action and broadcast the pet name when
a name is searched for. In `main.js` of the Pet Finder app, after the DOM
manipulation code, we will insert the following code:

```javascript
Livestax.trigger("newpet", "petName");
```

[More information on Triggers](https://github.com/livestax/docs#trigger)

[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/136046591087b3312cded431e46fe6e3289a7cfe)

3. Receive and Present Data - Part One
---

As before, in this app we will include the [jQuery](http://www.jquery.com) Library to easily manipulate
the DOM and `main.js` to separate any JavaScript from the HTML. A `<div>` tag
will be used as a container for the list of names we receive, with the
class `js-pet-names` to easily target it within JavaScript, and the class
`list-group` to apply the list group style from the
[LiveStax Theme](http://livestax.github.io/theme).

```html
<script src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
<script src="js/main.js"></script>
...
<div class="col-md-12">
  <div class=" list-group js-pet-names">
  </div>
</div>
```

Next, we want the app to prepend the pet data to the `<div>` tag when the app
receives the `pet-finder.newpet` trigger. This will be wrapped in a
`$(document).ready()` function so that the code will only be called when the
page is ready.

```javascript
$(document).ready(function() {
  Livestax.on("pet-finder.newpet", function(petName) {
    $(".js-pet-names").prepend("<a href=’#’ class=’list-group-item’>" + petName + "</a>");
  });
});
```

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/4c1b19d305ac92027d4a48c5fd1379f87162adef)
