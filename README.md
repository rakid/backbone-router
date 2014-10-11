# MarionetteRouter :: Routing Marionette with style \o/
----

```Backbone.MarionetteRouter``` wraps the ```Backbone.Marionette``` router to simplify it's use and bring new functionnalities

It's structure and API is inspired by routers in the node.js frameworks Meteor and ExpressJS.

Added functionnalities compared to the ```Backbone.Marionette``` router are :

 * Multiple controllers for a same path
 * Aliasing a route to another
 * Declaring routes to be executed only when a user is either logged in or not, or in both cases

## General use
----

Declaring routes goes through executing a simple method : ```Backbone.MarionetteRouter.map();```

This method takes a function as it's only parameter which will be executed in the router's context to access the internal API easily. A route consists of a unique name and an object to describe the route's action.

Let's just jump right in with an example :

```javascript
// Create a marionette app instance
var App = new Backbone.Marionette.Application();

// Start route declarations
Backbone.MarionetteRouter.map(function() {
  // Declare a route named 'home'
  this.route("home", {
    // The url to which the route will respond
    "path": "/",
    // Method to be executed when the given path is intercepted
    "action": function() {
      // Do something fantastic \o/
    }
  });

  // Declare other routes...
});

// Wait for document ready event
$(function() {
  // Start the marionette app
  App.start();

  // Start the router passing the marionette app instance
  Backbone.MarionetteRouter.start(App);
});
```

**Important:** Routes have to be declared before ```Backbone.Marionette``` and ```Backbone.MarionetteRouter``` are started.

## Redirecting

Once the app and router are started, the easiest way of redirecting the user to a route is by using the route name :

```javascript
CM.Router.go("home");
```

Route parameters can be passed in as an ```Array``` in the second parameter.

## Route declaration parameters
----

The ```path``` and ```action``` parameters are the base of a route. But a few more parameters exist to extend the control of the route.

```javascript
// Definition object for a route named 'user_edit'
{
  // Path with an 'id' parameter
  "path": "/user/:id/edit",

  // Route will only be executed if the user is logged in
  "authed": true,

  // Execute triggers before the 'action' controller
  "before": [
    "core:display",
    "users:display"
  ],

  // Main controller for the route
  "action": function(userId) {
    CM.vent.trigger("users:display_edit", [userId]);
  },

  // Execute triggers after the 'action' controller
  "after": [
    "core:post_analysis"
  ]
}
```

## Events distribution (Triggers)

```Backbone.MarionetteRouter``` uses the ```Marionette``` global event aggregator : ```App.vent```

This parameter can be overridden using any ```Backbone.Events``` instance.

```javascript
var App = new Backbone.Marionette.Application();

// Create a custom event aggregator
var myDispatcher = _.extend({}, Backbone.Events);

// Pass the custom object to the Router
Backbone.MarionetteRouter.dispatcher = myDispatcher;

App.start();
Backbone.MarionetteRouter.start(App);
```

All the triggers declared in the ```before``` and ```after``` parameters will be executed using the given event aggregator.

## Declaring particular routes for logged in/logged out users

Each route can receive an ```authed``` boolean parameter to declare if the route should be interpreted when the user is logged in or not.

```javascript
Backbone.MarionetteRouter.map(function() {
  // Declare secure route
  this.route("secure_route", {
    "path": "/admin/users",
    "authed": true,
    "action": function() {
      // Display list of users
    }
  });
});
```
To make a route be interpreted in both cases (i.e. when the user is logged in or logged out),
simply leave out the ```authed``` parameter in the route declaration.

**Important**

Only the server has the authority to tell if a connected client is a logged in user or not.
So for this to be achieved, the server has to print out a small piece of JavaScript to tell the router the current client's state :

```php
<script type="text/javascript" src="backbone.marionetterouter.js"></script>
<script type="text/javascript">
BackboneMarionette.authed = <?php if ($_SESSION['logged_in']): ?>true<?php else: ?>false<?php endif; ?>;
</script>
```