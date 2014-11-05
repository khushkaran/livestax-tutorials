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
