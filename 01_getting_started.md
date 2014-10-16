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
[Mozila Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML).

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
<script src=”http://livestax.com/assets/livestax-0.1.0.js”></script>
```
[See code changes](https://github.com/livestax/tutorial-pet-finder/commit/0cd754a6c68ca091fd3bf7a7637194f4ded34392)
