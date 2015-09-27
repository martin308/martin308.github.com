---
layout: post
title: "Elixir Phoenix + Browserify"
date: 2015-09-27
---

This weekend I have been playing with a new language/web framework combo. That is
[Elixir](http://elixir-lang.org) and [Phoenix](http://phoenixframework.org). So
far it has been really. Finding out how to do things in Phoenix has been really
fun as there is not a lot of information out there on it and many of the blog posts
written about no longer work with the latest versions.

The first thing that struck me as odd when I created a new Phoenix project was the
choice of client side assets build framework. By default Phoenix comes with
[Brunch](http://brunch.io), which I am not saying that I am against. Although it
is a lot to ask someone to not only use a new language and web framework but also
a new front end asset build tool.

My first task then was to see if I could replace Brunch with something that I am
more familiar with. My choice was [Browserify](http://browserify.org) as I am
pretty familiar with it and it's quite simple to get setup.

When you create a new Phoenix project using its provided generator, it creates a
default project structure for you, much like Rails does. After creating a project
the first step was to remove Brunch. Phoenix uses NPM so this was just a matter
of removing all of the dependencies from the package.json file, deleting the
node_modules directory and deleting the brunch-config.js file. Then we install
some dependencies that are going to do what Brunch was doing previously and create
some build steps to generate all of the front end assets. We end up with this.

{% highlight json linenos %}
{
  "repository": {
  },
+ "scripts": {
+   "build-assets": "cp -r web/static/assets/* priv/static",
+   "watch-assets": "watch-run -p 'web/static/assets/*' npm run build-assets",
+   "build-js": "browserify -t babelify web/static/js/app.js | uglifyjs -mc > priv/static/js/app.js",
+   "watch-js": "watchify -t babelify web/static/js/app.js -o priv/static/js/app.js -dv",
+   "build-css": "cat web/static/css/*.css > priv/static/css/app.css",
+   "watch-css": "catw web/static/css/*.css -o priv/static/css/app.css -v",
+   "build": "npm run build-assets && npm run build-js && npm run build-css",
+   "watch": "npm run watch-assets & npm run watch-js & npm run watch-css",
+   "test": "tap test/*.js"
+ },
  "dependencies": {
-   "brunch": "^1.8.5",
-   "babel-brunch": "^5.1.1",
-   "clean-css-brunch": ">= 1.0 < 1.8",
-   "css-brunch": ">= 1.0 < 1.8",
-   "javascript-brunch": ">= 1.0 < 1.8",
-   "uglify-js-brunch": ">= 1.0 < 1.8",
+   "babelify": "^6.3.0",
+   "browserify": "^11.2.0",
+   "catw": "^1.0.1",
+   "tap": "^1.4.1",
+   "uglify-js": "^2.4.24",
+   "watch-run": "^1.2.2",
+   "watchify": "^3.4.0"
  }
}
{% endhighlight %}

Now we need to modify the import path in the generated JavaScript so that we can
still import the Phoenix provided JavaScript. This is in "web/static/js/app.js".

{% highlight javascript linenos %}
-import "deps/phoenix_html/web/static/js/phoenix_html"
+import "../../../deps/phoenix_html/web/static/js/phoenix_html"
{% endhighlight %}

There is one last step that we need to do in order to complete the replacement of
Brunch with these other build tools. We need to tell Phoenix how to call the new
build steps rather than calling Brunch. We can find this code in "config/dev.exs"

{% highlight ruby linenos %}
code_reloader: true,
cache_static_lookup: false,
check_origin: false,
-  watchers: [node: ["node_modules/brunch/bin/brunch", "watch", "--stdin"]]
+  watchers: [npm: ["run", "watch"]]
{% endhighlight %}

Now it's all done. You can continue to play with Phoenix and Elixir using the
JavaScript build tools you know and love.
