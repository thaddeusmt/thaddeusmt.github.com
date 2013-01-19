---
layout: post
title: "Backbone.js: Bind Callback to Successful Model Fetch"
date: 2013-01-15 07:48
updated: 2013-01-17 15:58
comments: true
categories: [JavaScript, jQuery, Backbone.js]
keywords: javascript,callbacks,asynchronous,backbone,deferred,js
description: "Three ways to bind callbacks to succesful asynchronous Backbone.js Model fetches"
---

I am learning the [Backbone.js JavaScript framework](http://backbonejs.org/) right now, and ran into a problem. Backbone [Collections](http://backbonejs.org/#Collection) and [Models](http://backbonejs.org/#Model) can be set up to asynchronously load data from the server with the [`fetch()`](http://backbonejs.org/#Model-fetch) method, which fires off an Ajax `sync` call. You can bind a callback method to a Collection's [`reset`](http://backbonejs.org/#Collection-reset) event, which Backbone provides. It was my impression that the [`change`](http://backbonejs.org/#Model-change) event on a Model would do the same thing. But when I bound to the Model's `change` event in a View, it did not work as expected. Here is the code that was not working:

```
 var Model = Backbone.Model.extend({ url: '/my/url' });

 var View = Backbone.View.extend({
   initialize: function () {
     this.model.on('change', this.render); // attempt to bind to model change event
     this.model.fetch(); // fetching the model data from /my/url
   },
   render: function () {
     console.log(this.model); // this.model has not been populated!
   }
 });

 var myModel = new Model();
 var myView = new View({ model: myModel });
```

NOTE: the `.on()` method was formerly called `.bind()` apparently, so some older online examples use `this.model.bind()`, and this still works, presumably calling the underscore.js `bind()` method?

Well, it turns out I was making a simple mistake about one of JavaScript's trickier points: `this` context in the callback function. A substantial amount of reading later ([this article](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/) in particular was helpful) I realized I needed to pass in the `this` context as a third parameter to the `on()` method. There is actually a whole section on `this` in the [Backbone documentation](http://backbonejs.org/#FAQ-this).

More research showed me that since Backbone version 0.9.9 a new method was added to make it more foolproof to bind callbacks with the right context: `listenTo()`. This method is on the _object that has the callback_, instead of on the object you are binding to. This way it assumes the correct context and you don't need to worry about binding it correctly. Below is the working method of binding to a successful Model load, with the new `listenTo()` syntax.

<!-- more -->

## 1. Bind callback to Backbone.js Model 'change' event

```
 var Model = Backbone.Model.extend({ url: '/my/url' });

 var View = Backbone.View.extend({
   initialize: function () {
     /** here is the fixed, correct way to use `on()` **/
     //this.model.on('change', this.render, this); //**** pass in 'this' as the third parameter
     /** here is the new way to bind using the `listenTO()` convenience method, since Backbone version 0.9.9 **/
     this.listenTo(this.model,'change', this.render); // new bind technique, to change event on the View's model
     this.model.fetch(); // fetching the model data from /my/url
   },
   render: function () {
     console.log(this.model); // this.model has been populated!
   }
 });

 var myModel = new Model();
 var myView = new View({ model: myModel });
```

But before I figured that out I discovered two [other ways](http://stackoverflow.com/questions/9250523/how-to-wait-to-render-view-in-backbone-js-until-fetch-is-complete) to bind callbacks to successful Model and Collection `fetch()` Ajax calls. So I'll share those here as well. Both of them use jQuery functionailty, so if you are using Backbone with anything besides jQuery (like Zepto?), YMMV.

## 2. Pass 'success' callback into to the Model's 'fetch()' method

The second method is to pass a callback via option parameters into the Model's `fetch()` method. `fetch()` passes these parameters on to the `$.ajax()` method, so you can pass in options that [`$.ajax()`](http://api.jquery.com/jQuery.ajax/) accepts. This includes the `success` callback option. The trick here, as with the first technique, is making sure the callback has the correct `this` context. You can do this with the Underscore [`_.bindAll()`](http://underscorejs.org/#bindAll) method, which works some magic that is not quite clear to me (although this [StackOverflow post](http://stackoverflow.com/a/6582122/164439) helped).

```
 var Model = Backbone.Model.extend({ url: '/my/url' });

 var View = Backbone.View.extend({
   initialize: function () {
     _.bindAll(this, "render"); // make sure 'this' refers to this View in the success callback below
     this.model.fetch({ // call fetch() with the following options
       success: this.render // $.ajax 'success' callback
     });
   },
   render: function () {
     console.log(this.model); // this.model has been populated!
   }
 });

 var myModel = new Model();
 var myView = new View({ model: myModel });
```

## 3. Bind with jQuery Deferred 'done()' callback on 'fetch()' method

The final method uses [jQuery's Deferred object](http://api.jquery.com/category/deferred-object/), which is a powerful tool for making JavaScript more synchronous when you need it to be. I got the idea from [this blog](http://api.jquery.com/category/deferred-object/). [This article](http://www.erichynds.com/jquery/using-deferreds-in-jquery/) is also helpful for understanding Deferred. Basically objects that implement Deferred (like `$.ajax()`) return a "promise" object, which takes in callbacks via the `done()` method and queues them up. The queue of callbacks is run when the asynchronous method completes.

```
 var Model = Backbone.Model.extend({ url: '/my/url' });

 var View = Backbone.View.extend({
   initialize: function () {
     var _thisView = this; // need to be able to refer to 'this' context in the callback
     this.model.fetch().done(function () { // queue up this callback to run when fetch() completes
       _thisView.render();
     });
   },
   render: function () {
     console.log(this.model); // this.model has been populated!
   }
 });

 var myModel = new Model();
 var myView = new View({ model: myModel });
```

These last two methods are particularly useful when you only want to bind to the initial Model load event, and not subsequent `change` events (i.e. `change:attribute_name`) which are fired every time `model.set()` is called. For instance maybe you have some initial view rendering you want to do when the model first loads, and then have separate callbacks to update the view when model attributes change.

**[Here is a JsFiddle of all three Backbone Model fetch() bind methods](http://jsfiddle.net/thaddeusmt/DWzBq/) to play with.**

<iframe style="width: 100%; height: 300px" src="http://jsfiddle.net/thaddeusmt/DWzBq/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

## Waiting for multiple models to load with Deferred

Using jQuery Deferreds in Backbone is great for binding _single_ callbacks that depend on _multiple_ Models loading. It is much cleaner than chaining a series of callbacks, and allows the load events to run in parallel. You can easily do multiple-dependency callbacks with the `when()` method, like so:

```
var Model1 = Backbone.Model.extend({ url: '/my/url/id/1' });
var Model2 = Backbone.Model.extend({ url: '/my/url/id/2' });

var View = Backbone.View.extend({
  initialize: function () {
    var _thisView = this;
    // This uses jQuery's Deferred functionality to bind render() so it runs
    // after BOTH models have been fetched
    $.when(this.options.model1.fetch(),this.options.model2.fetch())
      .done(function () {
        _thisView.render();
      });
  },
  render: function () {
    console.log(this.options.model1); // this.options.model1 has been populated!
    console.log(this.options.model2); // this.options.model2 has been populated!
  }
});

// Initialize the Model
var myModel = new Model1();
var myOtherModel = new Model2();

// Initialize the View, passing it the model instances
var myView = new View({
  model1: myModel,
  model2: myOtherModel
});
```

**[Here is a running JsFiddle of binding to two Model fetch() calls](http://jsfiddle.net/thaddeusmt/WQ3uH/)**

<iframe style="width: 100%; height: 300px" src="http://jsfiddle.net/thaddeusmt/WQ3uH/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Deferred is, as I said, quite powerful and I'm happy to have it in my tool belt now (even though I solved my original Backbone issue without it).

(Want to use Deferred style JS on more than just jQuery AJAX objects, or without jQuery at all? Sam Breed created a plugin/mixin to [enable Deferred functionality in Underscore.js](https://github.com/wookiehangover/underscore.Deferred). Might be worth checking out if you are battling with asynchronous issues.)

  