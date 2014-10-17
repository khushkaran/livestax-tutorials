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

```html
<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
  </body>
</html>
```
[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/5454cd6d8b5ca4322387bf238bccf2d5d88b1b61)

2. Add the LiveStax JavaScript to the HTML Page
---
Next, for us to interact with LiveStax and to allow the app to become
visible on LiveStax we need to include the LiveStax JavaScript file.
This needs to be done before any other JavaScript is called, therefore,
we’ll place this within the `<head>` tag.

```html
<script src="http://livestax.com/assets/livestax-0.1.0.js"></script>
```
[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/0cd754a6c68ca091fd3bf7a7637194f4ded34392)

3. Add the LiveStax Theme
---
In order to easily style your application, we will include the LiveStax
theme in CSS format in the `<head>` tag. To find out more about CSS, visit the
[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference).

The [LiveStax theme](http://livestax.github.io/theme/) is based upon the Bootstrap
front-end framework, a collection of highly curated generic CSS classes that help arrange
elements quickly and easily. To find out more about Bootstrap you can view the
documentation by clicking [here](http://getbootstrap.com/).

```html
<link href="http://dz44vc6ose3il.cloudfront.net/theme.css" rel="stylesheet" type="text/css" media="all">
```
[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/f061853a9dc2c0d7a63fac6094c56ecfa7486490)

4. Add UI Elements
---
Now we are going to begin adding functionality to your Pet Finder application.
Using the [LiveStax theme](http://livestax.github.io/theme/) we are going to add
elements such as an input box, input label, image placeholder, button and a few
headers. There are many HTML elements available, visit the
[Mozilla Developer Network ](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
for a more exhaustive list. All these elements will be placed within the `<body>` tag
as we want them visible to the user.

We need to have an input box so that the user can enter a name of their potential pet.
With this we should also display what this box is for, so we’ll also add a input label.

```html
<div class="form-group">
  <label class="control-label" for="name">Pet Name</label>
  <input type="text" name="name" class="form-control">
</div>
```

To submit this name, we will need a button that the user can click.

```html
<a href="#" class="btn btn-primary">Find Pet</a>
```

We want to be able to display the name the user submitted, the type of pet we received
and finally an image of their beloved pet. So we will insert 2 header tags, `<h1>` and
`<h2>`, and a image tag `<img>`.

```html
<h1></h1>
<h2></h2>
<img src="" class="img-responsive" alt="Pet Image">
```

Unfortunately, we cannot see any content for these when the page loads, this is because
simply, there is nothing there. It is common practice to provide some placeholders, so
we’ll add that now.

```html
<h1>Pet Name</h1>
<h2>Pet Type</h2>
<img src="http://placehold.it/350x350" class="img-responsive" alt="Pet Image">
```
Please note: we are using a placeholder image from [placehold.it](http://www.placehold.it)
as it allows us to perform something very simple, however, there are many other ways to
insert a placeholder image.

Bootstrap provides us with a [grid system](http://getbootstrap.com/css/#grid) that allows
us to easily arrange our HTML elements, they are simple classes applied to `<div>` tags.
We want each group of elements to take up the full width of the page, therefore we will
wrap each section in div tags with the class `col-md-12` applied.

```html
<div class="col-md-12">
  <div class="form-group">
  ...
</div>

<div class="col-md-12">
  <a href="#" class="btn btn-primary">Find Pet</a>
</div>

<div class="col-md-12">
  <h1>Pet Name</h1>
  ...
</div>
```

[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/ed6946949fb69e1944274777dee6e8b89415c33c)
