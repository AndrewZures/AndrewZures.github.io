---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 5: Platform Files"
date:   2014-02-08 09:57:26
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 

#Using Platform Files to Isolate Library Differences

Now that we can write a single file that ultimate becomes separate `.clj` and `.cljs` files, we can look at how we're going to deal with the differences in the Clojure and ClojureScript platforms.

We will isolate these differences in two files (one `clj` and one `cljs`) with the same file name and same namespace name. We will then reference this common namespace in our code.  When our clojure code runs, the `.clj` namespace will execute, when we run our ClojureScript code, our `.cljs` namespace will execute.  Thus the rest of our files can be written without a need to focus on platform details.

An example will illustrate the project design:

Let's say we want to use our platform's `abs` function to find the absolute value of a number.  In Clojure this is done using the `org.clojure/math.numeric-tower` library, while ClojureScript would use JavaScript's `Math/abs` function.

Lets add an "absolute difference" functionality to `shared_file.cljx`.  This function will simply find the absolute difference between two numbers.

First, let's make a test.  Add the code below to the `describe` block in your `shared_file_spec.cljx`:

{% highlight clojure linenos %}

(it "finds the absolute difference between two numbers"
      (should= 1 (shared-file/abs-diff -101 100)))

{% endhighlight %}

This test will simply help us decide if everything is working correctly. Now let's focus on the source code.

Add the code below to your  `shared-file.cljx`:

{% highlight clojure linenos %}
(ns myproject.shared-file
  (:require [myproject.platform :as platform]))
{% endhighlight %}

And add your new function, which should look like this:

{% highlight clojure linenos %}

(defn absolute-difference [x y]
  (- (platform/absolute-val x) (platform/absolute-val y)))

{% endhighlight %}

As you can see, we're referencing a platform namespace.  This namespace will change based off of what platform we're executing.  Let's now make our platform namespace files.

Create `platform.clj` in `src/clj/myproject/` and `platform.cljs` in `src/cljs/myproject/`

In `platform.clj` add:

{% highlight clojure linenos %}
(ns myproject.platform
  (:require [clojure.math.numeric-tower :as math]))

  (defn abs [num]
      (math/abs num))

{% endhighlight %}

You'll also have to add the `numeric-tower` dependency to your `:clj` profile in `project.clj`:

{% highlight clojure linenos %}
   :dependencies [[org.clojure/math.numeric-tower "0.0.4"]]
{% endhighlight %}

Now let's move to the cljs side.  In `platform.cljs` add:

{% highlight clojure linenos %}
 (ns myproject.platform)

 (defn abs [num]
     (js/Math.abs num))
{% endhighlight %}

Now we have two like-named functions in two like-named namespaces.  If we run our tests, they'll pass.  When Clojure runs, the `platform.clj` file will be used and the `numeric-tower` library will be executed.  When ClojureScript runs, the `platform.cljs` file will be used and JavaScript's `abs` function will be executed.  Our `shared-file` doesn't need to know about those details.  It can simply call the platform namespace's function and receive the results.

#Where We're At
We now have a function, `abs-diff` that is written once yet can be used for both Clojure and ClojureScript.  This means that not only can we write a single code base that can run in Clojure and ClojureScript, it also means that the differences between the two platforms is isolated to the `platform` namespace.

In [Part 6][part_6] we'll utilized this same technique but will Clojure macros.
