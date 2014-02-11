---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 7: Fragile 'If's"
date:   2014-02-08 09:57:24
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 
[sample_project]: https://github.com/AndrewZures/combining_clj_cljs_libraries/tree/throwable
[clojure_documentation]: http://clojuredocs.org/clojure_core/1.2.0/clojure.core/try
[part_8]: https://github.com/AndrewZures/combining_clj_cljs_libraries/tree/macro_dependenc://github.com/AndrewZures/combining_clj_cljs_libraries/tree/one_jar  

Here is [Sample Project][sample_project] with working code and tests through this part (Part 7) of the tutorial.

#Using a Fragile "If" Statement to Get a Bit More Functionality

In the previous portion of this tutorial, we were able to get a great deal of cross-platform functionality using platform files. But now we'll look at a way to define a clojure macro with platform-specific code, using a very fragile if statement.  

First, let's look at an example:

#Fragile If: An Example

In your `shared_file_spec.cljx` files add this test:

{% highlight clojure linenos %}
(it "catch slurp failure"
   (should= true (macros/slurpable-file? "badfilename")))
{% endhighlight %}

In this test, we're attempting to `slurp` a bad file name.  In both Clojure and ClojureScript, this will raise an exception.  But exceptions are a little different in Java and JavaScript.  Java will require an `Exception` while JavaScript will use a `js/Ojbect`. You can see the [Clojure documentation][clojure_documentation] for an example of both situations.

Let's go to our `macros.clj` file and see what we can do to pass this test for both platforms.

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

If we try this macro, it will pass the tests for both platforms.  It does this by adding the correct platform-specific catch statement during macro expansion.  This may seem like it has opened up an amazing set of functionality but the if statement is fragile.  It relies on the existence (or lack thereof) of a ClojureScript specific namespace.  If something changed in ClojureScript, the entire library could fail.  Thus this is a bit of hack.  But it gets us where we want to go and there are few other options.  

#ns-resolve Can Help Too

As a brief side note.  The hack noted above will still fail if the clojure compiler cannot recognize the name of a currently absent ClojureScript namespace. An example is `cljs.compiler/munge`. If this included in a `.clj` macro, your Clojure-side tests will fails because Clojure will not find the namespace.  

However we can get around this using `ns-resolve`.  Instead of referencing `cljs.compiler/munge` we can replace it with:

{% highlight clojure linenos %}
(ns-reolve 'cljs.compiler "munge")
{% endhighlight %}

This will remove the compile check of the `cljs.compiler` namespace.

#Where We're At
So now we've seen how to use platform files to isolate platform-specific code and we've also seen a little hack that can help when macros must be defined a platform-specific manner.  In [Part 8][part_8] of this tutorial we'll put it all together, quite literally, and combine our Clojure and ClojureScript libraries into a single jar.


