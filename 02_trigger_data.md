Trigger Data
===

In this tutorial we will build upon the Pet Finder app and allow it to communicate
to another app, the Pet Finder History. As it’s name suggests, it will contain a
list of pet names that have been recently searched for.

1. Create a Livestax App
---

As we did in the previous app, we will start off by creating a skeleton Livestax
app with the Livestax JavaScript and the Livestax Theme from the CDN.

```diff
+ <!DOCTYPE html>
+ <html>
+   <head>
+     <script src="//assets.livestax.com/livestax-0.2.0.min.js"></script>
+     <link href="//assets.livestax.com/theme-0.0.1.css" rel="stylesheet" type="text/css" media="all">
+   </head>
+   <body>
+   </body>
+ </html>
```

2. Communicate Data - Part One
---

We will use Livestax to trigger a named action and broadcast the pet name when
a name is searched for. In `main.js` of the Pet Finder app, after the DOM
manipulation code, we will insert the following code:

```diff
...
$.getJSON("http://tutorial-pet-service.herokuapp.com/?name=" + petName, function(pet) {
...
});
+Livestax.trigger("newpet", "petName");
```

[More information on Triggers](http://developers.livestax.com/js_api?v=0.4.0#javascript-api-version-040-communicating-with-other-apps-events)

3. Receive and Present Data - Part One
---

As before, in this app we will include the [jQuery](http://www.jquery.com) Library to easily manipulate
the DOM and `main.js` to separate any JavaScript from the HTML. A `<div>` tag
will be used as a container for the list of names we receive, with the
class `js-pet-names` to easily target it within JavaScript, and the class
`list-group` to apply the list group style from the
[Livestax Theme](http://livestax.github.io/theme).

```diff
...
<link href="//assets.livestax.com/theme-0.0.1.css" rel="stylesheet" type="text/css" media="all">
+<script src="//code.jquery.com/jquery-2.1.1.min.js"></script>
+<script src="js/main.js"></script>
...
+<div class="col-md-12">
+  <div class=" list-group js-pet-names">
+  </div>
+</div>
```

Next, we want the app to prepend the pet data to the `<div>` tag when the app
receives the `pet-finder.newpet` trigger. This will be wrapped in a
`$(document).ready()` function so that the code will only be called when the
page is ready.

```diff
+$(document).ready(function() {
+  Livestax.on("pet-finder.newpet", function(petName) {
+    $(".js-pet-names").prepend("<a href=’#’ class=’list-group-item’>" + petName + "</a>");
+  });
+});
```

4. Communicate Data - Part Two
---

We now have a history of searched pet names, but we also want to be able to
redisplay the information on that name in the Pet Finder app. To do this, we
will use a [Key Value Store](http://developers.livestax.com/js_api?v=0.4.0#javascript-api-version-040-communicating-with-other-apps-key-value-store)
where another app can access it. The links are rendered after the JavaScript has
been loaded, therefore, we need to bind the click capture function to a parent
of the link. Here, we will use `document.body` as the parent and will communicate
the Pet Name for the Pet Finder app to utilise to update it’s view.

```diff
$(document).ready(function() {
...
});

+$(document.body).on("click", ".js-pet-names a", function(event) {
+  var clickedLink, petName;
+  clickedLink = event.target;
+  petName = clickedLink.text;
+  $(".js-pet-names a").removeClass("active");
+  $(clickedLink).addClass("active");
+  Livestax.store.set("getpet", petName);
+});
```

You may notice that we remove and add the "active" CSS class using the `removeClass()` and
`addClass()` functions respectively. This is because the Livestax Theme uses this class to
present some user feedback to the User to what is active at this point in time.

5. Extracting DOM Manipulation
---

When writing code, it is common practice to extract common functionality to
functions/methods where they can be reused multiple times. As we want to get a pet and
update the DOM when we submit directly in the app **and** when data from the Pet Finder
History app is received, this would be a prime candidate to refactor out into a function.
So, lets start by moving the code to a `updatePetDetails()` function.

```diff
+function updatePetDetails(petName) {
+  $.getJSON("http://tutorial-pet-service.herokuapp.com/?name=" + petName, function(pet) {
+    $(".js-pet-name").html(pet.name);
+    $(".js-pet-type").html(pet.type);
+    $(".js-pet-img").attr("src", pet.image_url);
+  });
+};
```

Now we can replace the code in the `click()` function to call the `updatePetDetails()` function as follows:

```diff
...
var petName = $('.js-pet-name-input').val();
-$.getJSON("http://tutorial-pet-service.herokuapp.com/?name=" + petName, function(pet) {
-  $(".js-pet-name").html(pet.name);
-  $(".js-pet-type").html(pet.type);
-  $(".js-pet-img").attr("src", pet.image_url);
-});
+updatePetDetails(petName);
Livestax.trigger("newpet", petName);
...
```
To improve user feedback, when the view has been updated, the input box should empty allowing the user
to enter another name. To do this, we will set the value of the input to an empty string.

```diff
$.getJSON("http://tutorial-pet-service.herokuapp.com/?name=" + petName, function(pet) {
  $(".js-pet-name").html(pet.name);
  $(".js-pet-type").html(pet.type);
  $(".js-pet-img").attr("src", pet.image_url);
+ $(".js-pet-name-input").val("");
});
```

6. Receive and Present Data - Part Two
---

When using the [Key Value Store](http://developers.livestax.com/js_api?v=0.4.0#javascript-api-version-040-communicating-with-other-apps-key-value-store), the app needs to
watch the store using the `Livestax.store.watch()` function we can then connect this directly to the
updatePetDetails() function.

```diff
...
$(".js-find-pet-submit").click(function() {
...
});

+Livestax.store.watch("pet-finder-history.getpet", updatePetDetails);
...
```

And now we have two apps that can communicate with each other and transmit information to each other.

Further Development
===

UI Considerations
---

When this app is added, it displays an empty page, which could look as if it was broken to a
user. Therefore, it would be helpful for a notice to be shown to user if there is nothing in
the history. The [Livestax Theme](http://livestax.github.io/theme/) provides a notices component
that can be easily dropped in.

```diff
...
<div class="col-md-12">
+  <div class="notice">
+    <div class="media-badge media-badge-lg media-badge-info-inverse img-circle">
+      <span class="media-badge-container">
+        <i class="fa fa-flag"></i>
+      </span>
+    </div>
+    <h2 class="text-info"><strong>Empty History</strong></h2>
+    <p>No Pet names have been searched for, search for a name in the Pet Finder App to create a History.</p>
+  </div>
</div>
...
```

This now shows at all times, which is confusing, therefore, we’ll hide it when an item is prepended to the list.

```diff
...
Livestax.on("pet-finder.newpet", function(petName) {
+ $(".notice").hide();
  $(".js-pet-names").prepend("<a href=’#’ class=’list-group-item’>" + petName + "</a>");
});
...
```
