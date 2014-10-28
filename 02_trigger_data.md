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
