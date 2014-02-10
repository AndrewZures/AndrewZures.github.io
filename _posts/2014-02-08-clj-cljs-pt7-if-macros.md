---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 7: Fragile 'If's"
date:   2014-02-08 09:57:24
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 

#Using a Fragile "If" Statement to Get a Bit More Functionality

In the previous portion of this tutorial, we were able to get a great deal of cross-platform functionality using platform files. But now we'll look at a way to fully define a clojure macro using a very fragile if statemnt.  First, let's look at an example:

In your `shared_file_spec.cljx` files add this test:

{% highlight clojure linenos %}
(it "catch slurp failure"
   (should= true (macros/slurpable-file? "badfilename")))
{% endhighlight %}

In this test, we're attempting to `slurp` a bad file name.  In both Clojure and ClojureScript, this will raise an exception.  But exceptions are a little different in Java and JavaScript.  Java will require an `Exception` while JavaScript will use a `js/Ojbect`. You can see the [Clojure documentation][clojure_documentation] for an example of both situations.

Let's see go to our `macros.clj` file and see what we can do to pass this test for both platform.

Here's what the Clojure version might look like.  But of course it won't work in ClojureScript.  ClojureScript won't know what to do with `Exception`.

{% highlight clojure linenos %}
(defmacro slurpable-file? [file-name]
  `(try
     (slurp ~file-name)
     (catch Exception e# true)))

{% endhighlight %}

Here's what the ClojureScript version might look like. But of course it won't work in Clojure.  Clojure won't know what to do with `js/Ojbect`.
{% highlight clojure linenos %}
(defmacro slurpable-file? [file-name]
  `(try
     (slurp ~file-name)
     (catch js/Object e# true)))
{% endhighlight %}

#Context Aware Macros

We have two separate macros that simply won't work on both platforms.  But what if we could determine if the `macro.clj` file was expanding in a Clojure context or a ClojureScript context. Maybe then we could use this information to build the correct macro for the currently-executing library. 

This is where our "fragile if" statement comes into play.  It looks like this:

{% highlight clojure linenos %}
(defn cljs? []
    (boolean (find-ns 'cljs.analyzer)))
{% endhighlight %}

As you can see it determines if the file is running in a ClojureScript context if the `cljs.analyzer` namespace can be found. Otherwise, we will assume it is in a Clojure context.  We can use this function to create a macro that can combine our two previous macros.

{% highlight clojure linenos %}
(defmacro slurpable-file? [file-name]
 `(try
   (slurp ~file-name)
      ~(if (cljs?)
        '(catch js/Object e# true)
        '(catch Exception e# true)))
 )
{% endhighlight %}

If we try this macro, it will pass the tests for both platforms.  It does this by adding the correct platform-specific catch statement during macro expansion.  This may seem like it opened up an amazing set of functionality but the if statement is fragile.  It relies on the existence (or lack thereof) of a ClojureScript specific namespace.  If something changed in ClojureScript, the entire library could fail.  Thus this is a bit of hack.  But it gets us where we want to go and there are few other options.  

#Where We're At
So now we've seen how to use platform files to isolate platform-specific code and we've also seen a little hack that can help when macros must be defined a certain way during macro expansion.  In [Part 8][part_8] of this tutorial we'll put it all together, quite literally, and combine our Clojure and ClojureScript libraries into a single jar.


