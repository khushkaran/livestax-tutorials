Getting Started
===

In this tutorial, we are going to develop a web application that sends a name
to a Pet Finder Service, and returns an image for us to use in the application.
We will then use this data to populate the page with it and the other details
provided.

1. Create the HTML Page
---
Firstly, we need to create the basic requirements of a HTML page, for
more information on HTML, visit the
[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML).
```diff
+ <!DOCTYPE html>
+ <html>
+   <head>
+   </head>
+   <body>
+   </body>
+ </html>
```
2. Add the Livestax JavaScript
---
Next, we include the Livestax JavaScript file that will make our app
compatible with Livestax. This needs to be done before any other
JavaScript is called, therefore, we’ll place this within the `<head>` tag.
```diff
...
<html>
  <head>
+   <script src="//dz44vc6ose3il.cloudfront.net/livestax-0.2.0.min.js"></script>
  </head>
  <body>
...
```

3. Add the Livestax Theme
---
In order to easily style your application, we will include the Livestax
theme in CSS format in the `<head>` tag. To find out more about CSS, visit the
[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference).

The [Livestax theme] is based upon the Bootstrap
front-end framework, a collection of highly curated generic CSS classes that help arrange
elements quickly and easily. To find out more about Bootstrap you can view the
documentation by clicking [here](http://getbootstrap.com/).
```diff
...
<head>
  <script src="//dz44vc6ose3il.cloudfront.net/livestax-0.2.0.min.js"></script>
+ <link href="//dz44vc6ose3il.cloudfront.net/theme-0.0.1.css" rel="stylesheet" type="text/css" media="all" />
</head>
...
```

4. Add UI Elements
---
Now we are going to begin adding functionality to your Pet Finder application.
Using the [Livestax theme] we are going to add
elements such as an input box, input label, image placeholder, button and a few
headers. There are many HTML elements available, visit the
[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
 for a more exhaustive list. All these elements will be placed within the `<body>` tag
as we want them visible to the user.

We need to have an input box so that the user can enter a name of their potential pet.
With this we should also display what this box is for, so we’ll also add a input label.
```diff
...
<body>
+ <div class="form-group">
+   <label class="control-label" for="name">Pet Name</label>
+   <input type="text" name="name" class="form-control">
+ </div>
</body>
...
```

To submit this name, we will need a button that the user can click.
```diff
...
  <input type="text" name="name" class="form-control">
+ <a href="#" class="btn btn-primary">Find Pet</a>
</div>
...
```

We want to be able to display the name the user submitted, the type of pet we received
and finally an image of their beloved pet. So we will insert 2 header tags, `<h1>` and
`<h2>`, and a image tag `<img>`.
```diff
...
  <a href="#" class="btn btn-primary">Find Pet</a>
+ <h1></h1>
+ <h2></h2>
+ <img src="" class="img-responsive" alt="Pet Image" />
</div>
...
```

Unfortunately, we cannot see any content for these when the page loads, this is because
simply, there is nothing there. It is common practice to provide some placeholders, so
we’ll add that now.
```diff
...
-<h1></h1>
+<h1>Pet Name</h1>
-<h2></h2>
+<h2>Pet Type</h2>
-<img src="" class="img-responsive" alt="Pet Image" />
+<img src="http://placehold.it/350x350" class="img-responsive" alt="Pet Image" />
...
```

Please note: we are using a placeholder image from [placehold.it](http://www.placehold.it)
as it allows us to perform something very simple, however, there are many other ways to
insert a placeholder image.

Bootstrap provides us with a [grid system](http://getbootstrap.com/css/#grid) that allows
us to easily arrange our HTML elements, they are simple classes applied to `<div>` tags.
We want each group of elements to take up the full width of the page, therefore we will
wrap each section in div tags with the class `col-md-12` applied.
```diff
+<div class="col-md-12">
  <div class="form-group">
  ...
+</div>

+<div class="col-md-12">
  <a href="#" class="btn btn-primary">Find Pet</a>
+</div>

+<div class="col-md-12">
  <h1>Pet Name</h1>
  ...
+</div>
```

5. Request Data from API
===
Now, let us make a request to the Pet Finder Service API. For your information, the URL for the service is
http://tutorial-pet-service.herokuapp.com and it takes a parameter called `name`. In order to make a request,
we are going to use the [jQuery](http://jquery.com/) library. There are many alternative libraries that you
could use instead. Let’s add the jQuery framework from the CDN within the `<head>` tags.

```diff
...
<head>
  <script src="//dz44vc6ose3il.cloudfront.net/livestax-0.2.0.min.js"></script>
  <link href="//dz44vc6ose3il.cloudfront.net/theme-0.0.1.css" rel="stylesheet" type="text/css" media="all" />
+ <script src="//code.jquery.com/jquery-2.1.1.min.js"></script>
</head>
...
```

The text input and anchor HTML elements will need to be targeted via JavaScript, so we add CSS classes
to the elements for them to be easily identified. As a convention, we prefix these CSS classes with `js-`.

```diff
...
<div class="col-md-12">
  <div class="form-group">
-   <input type="text" name="name" class="form-control">
+   <input type="text" name="name" class="form-control js-pet-name-input">
...
<div class="col-md-12">
- <a href="#" class="btn btn-primary">Find Pet</a>
+ <a href="#" class="btn btn-primary js-find-pet-submit">Find Pet</a>
...
```

We now create a `main.js` file, that will provide a location for our custom JavaScript. This file will be
included into our `index.html` file via a `<script>` tag within the `<head>` element.

```diff
...
<head>
  <script src="//dz44vc6ose3il.cloudfront.net/livestax-0.2.0.min.js"></script>
  <link href="//dz44vc6ose3il.cloudfront.net/theme-0.0.1.css" rel="stylesheet" type="text/css" media="all" />
  <script src="//code.jquery.com/jquery-2.1.1.min.js"></script>
+ <script src="js/main.js"></script>
</head>
...
```

So, within our `main.js`, lets add the jQuery function that will be called when the web page has been loaded.
```diff
+$(document).ready(function() {
+});
```

Then, we set up an event handler for the click event of the "Find Pet" button.
```diff
$(document).ready(function() {
+ $(".js-find-pet-submit").click(function() {
+ });
});
```

Now, we need to capture the text within the pet name input box.
```diff
...
$(".js-find-pet-submit").click(function() {
+ var petName = $('.js-pet-name-input').val();
});
...
```

Finally, lets make our request to the Pet Finder Service API.
```diff
...
var petName = $('.js-pet-name-input').val();
+$.getJSON("http://tutorial-pet-service.herokuapp.com/?name=" + petName, function(pet) {
+});
...
```

6. Render Data to the DOM
===
Now we are getting the data from the Pet Finder, which is a JSON object, we can update the page with this
data. An example JSON response (when the `name` parameter "sam" is passed to the service) is as follows:
```json
{
  "name":"sam",
  "type":"horse",
  "image_url":"http://tutorial-pet-service.herokuapp.com/images/returnable/horse.jpg"
}
```

Firstly, as in the previous step, we need to add targeting classes to the elements that will change, that is the 2 header
and image elements.
```diff
...
-<h1>Pet Name</h1>
+<h1 class="js-pet-name">Pet Name</h1>
-<h2>Pet Type</h2>
+<h2 class="js-pet-type">Pet Type</h2>
-<img src="http://placehold.it/350x350" class="img-responsive" alt="Pet Image" />
+<img src="http://placehold.it/350x350" class="img-responsive js-pet-img" alt="Pet Image" />
...
```

Now we can easily manipulate these elements, so lets switch to `main.js` and add the code that will use the data we are
getting back, this will be placed within the `getJSON()` block.
```diff
...
$.getJSON("http://tutorial-pet-service.herokuapp.com/?name=" + petName, function(pet) {
+ $(".js-pet-name").html(pet.name);
+ $(".js-pet-type").html(pet.type);
+ $(".js-pet-img").attr("src", pet.image_url);
});
...
```

And that’s it, you now have an app that you can use within Livestax to find a pet from a name it is passed.

[Livestax theme]: http://livestax.github.io/theme/
