---
layout: post
title: "Backbone.Marionette Module Anti Pattern"
date: 2013-02-03 16:44
comments: true
tags: [coding, backbone, backbone-marionette, javascript]
---

I have recently been using [Backbone.Marionette](http://marionettejs.com/) in anger while building [Raygun.io](http://raygun.io). I was struggling to find posts on how to use modules correctly. So of course I ended up using them poorly and have since discovered a better way to use them. I am going to show both the first way that I used Marionette and the new way that I have moved to after discovering the weaknesses in my first approach.

<!--more-->

I am going to assume that you have a fairly good idea about javascript, backbone and marionette. If you don't I recommend reading up on all of them beforehand!

The intial approach I took was to use module initializers inside every module to add their layouts/views etc to the applications region. This is demonstrated below.

{% highlight javascript linenos %}
var MyApp = new Backbone.Marionette.Application();

MyApp.addRegions({
  someRegion: "#some-div"
});

MyApp.addInitializer(function(options){
  if(options.somethingOrOther) {
    MyApp.Foo.start();
  }
});

MyApp.module("Foo", {
  startWithParent: false,

  define: function(Foo, App){
    var layout = Backbone.Marionette.Layout.extend({
    });

    Foo.addInitializer(function(){
      App.someRegion.show(new layout());
    });
  }
});
{% endhighlight %}

This shows basically what I was doing, the main modules that I had under the application would only be started based on some configuration option passed in when it was started. They would then handle themselves creating any views they needed and showing them in the applications regions. This worked reasonably well and so I continued following this pattern. When I needed to add more functionality to a particular module I would create a new sub module and follow the same pattern.

The weakness of this approach is when you want/need to use one of your modules somewhere else in the application you are in for a world of hurt. This is because your only interaction with the module is by starting it up. You are not able to place the functionality contained within the Foo module anywhere else. You could stop the module and start it again, but this would only leave you with the modules functionality in the same region it was in. By using the initializer approach it ensured that the modules could really only be used in the one place. Flying in the face of the 'composite' application architecture that Marionette is based on. As we can no longer "compose" our application with reusable pieces of functionality.

The approach I have begun to take now is demonstrated below.

{% highlight javascript linenos %}
var MyApp = new Backbone.Marionette.Application();

MyApp.addRegions({
  someRegion: "#some-div"
});

MyApp.addInitializer(function(options){
  if(options.somethingOrOther) {
    MyApp.Foo.display(MyApp.someRegion);
  }
});

MyApp.module("Foo", function(Foo, App){
  var layout = Backbone.Marionette.Layout.extend({
  });

  Foo.display = function (region) {
    region.show(new layout());
  };
});
{% endhighlight %}

This approach is only subtly different. The Foo module is now started by default and the initializer has been replace by a 'display' function that takes the region to display Foo's functionality in. The advantage of this approach is that now Foo's functionality can be more easily used within other areas of the application. By calling the display function and passing in a different region it can be displayed somewhere else in the application.

Hopefully this will stop you going down the same path that I did and allow to create more reusable Marionette applications. Let me know if you have any better ideas on structuring Marionette modules
