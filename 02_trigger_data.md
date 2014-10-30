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

4. Communicate Data - Part Two
---

We now have a history of searched pet names, but we also want to be able to
redisplay the information on that name in the Pet Finder app. To do this, we
will use a [Key Value Store](https://github.com/livestax/docs#key-value-store)
where another app can access it. The links are rendered after the JavaScript has
been loaded, therefore, we need to bind the click capture function to a parent
of the link. Here, we will use `document.body` as the parent and will communicate
the Pet Name for the Pet Finder app to utilise to update it’s view.

```javascript
$(document.body).on("click", ".js-pet-names a", function(event) {
  var clickedLink, petName;
  clickedLink = event.target;
  petName = clickedLink.text;
  $(".js-pet-names a").removeClass("active");
  $(clickedLink).addClass("active");
  Livestax.store.set("getpet", petName);
});
```

You may notice that we remove and add the "active" CSS class using the `removeClass()` and
`addClass()` functions respectively. This is because the LiveStax Theme uses this class to
present some user feedback to the User to what is active at this point in time.

[See code changes](https://github.com/livestax/tutorial-pet-finder-history/commit/df7816cdaf89d4a4c737d00ea2074bf1403d1b2e)

5. Extracting DOM Manipulation
---

When writing code, it is common practice to extract common functionality to
functions/methods where they can be reused multiple times. As we want to get a pet and
update the DOM when we submit directly in the app **and** when data from the Pet Finder
History app is received, this would be a prime candidate to refactor out into a function.
So, lets start by moving the code to a `updatePetDetails()` function.

```javascript
function updatePetDetails(petName) {
  $.getJSON("http://tutorial-pet-service.herokuapp.com/?name=" + petName, function(pet) {
    $(".js-pet-name").html(pet.name);
    $(".js-pet-type").html(pet.type);
    $(".js-pet-img").attr("src", pet.image_url);
  });
};
```

Now we can replace the code in the `click()` function to call the `updatePetDetails()` function as follows:

```javascript
...
var petName = $('.js-pet-name-input').val();
updatePetDetails(petName);
Livestax.trigger("newpet", petName);
...
```
To improve user feedback, when the view has been updated, the input box should empty allowing the user
to enter another name. To do this, we will set the value of the input to an empty string.

```javascript
$(".js-pet-name-input").val("");
```

[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/7ff06d952bf3501cf191f99d2ea31ee4297b8840)

6. Receive and Present Data - Part Two
---

When using the [Key Value Store](https://github.com/livestax/docs#key-value-store), the app needs to
watch the store using the `Livestax.store.watch()` function we can then connect this directly to the
updatePetDetails() function.

```javascript
Livestax.store.watch("pet-finder-history.getpet", updatePetDetails);
```

And now we have two apps that can communicate with each other and transmit information to each other.

[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/1de72ba53291d6cb78b1cb1f171d5638fc6a4cc6)