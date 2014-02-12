---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 5: Platform Files"
date:   2014-02-08 09:57:26
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 

[part_1]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt1-context.html 
[part_2]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt2-setup.html
[part_3]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt3-dividing-profiles.html
[part_4]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt4-cljx.html
[part_5]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt5-platform.html
[part_6]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt6-platform-and-macros.html
[part_7]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt7-if-macros.html
[part_8]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt8-combining-profiles.html
[part_9]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt9-final-thoughts.html

#Using Platform Files to Isolate Library Differences

Now that we can write a single file that ultimately becomes separate `.clj` and `.cljs` files, we can look at how we're going to deal with the differences in the Clojure and ClojureScript platforms.

We will isolate these differences in two files (one `.clj` and one `.cljs`) with the same file name and same namespace name. We will then reference this common namespace in our code.  When our clojure code runs, the `.clj` namespace will execute, and when we run our ClojureScript code, our `.cljs` namespace will execute.  Thus the rest of our files can be written without a need to focus on platform details.

An example will illustrate the project design:

#Platform Files: An Example

Let's say we want to use our platform's `abs` function to find the absolute value of a number.  In Clojure this is done using the `org.clojure/math.numeric-tower` library, while ClojureScript would use JavaScript's `Math/abs` function.

To set up this example, we'll make a `.cljx` function that uses `abs`

Lets add an "absolute difference" functionality to `shared_file.cljx`.  This functionality will simply find the absolute difference between two numbers.

As always, we'll start with a test.  Add the code below to the `describe` block in your `shared_file_spec.cljx`:

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

Now we have two like-named functions in two like-named namespaces.  If we run our tests, they'll pass.  This works because when Clojure runs, the `platform.clj` file will be used and the `numeric-tower` library will be executed.  When ClojureScript runs, the `platform.cljs` file will be used and JavaScript's `abs` function will be executed.  Our `shared-file` doesn't need to know about those details.  It can simply call the platform namespace's function and receive the results.

#Where We're At
We now have a function, `abs-diff` that is written just once yet can be used for both Clojure and ClojureScript.  This means that not only can we write a single code base that can run in Clojure and ClojureScript, it also means that the differences between the two platforms is isolated to the `platform` namespace.

In [Part 6][part_6] we'll utilized this same technique but with Clojure macros.

[Part 1: A Bit Of Context][part_1]

[Part 2: Setting Up Your Base Project][part_2]

[Part 3: Dividing Profiles][part_3]

[Part 4: Using CLJX][part_4]

[Part 5: Platform Files][part_5]

[Part 6: Platform Files and Macros][part_6]

[Part 7: Context Aware Macros][part_7]

[Part 8: One Jar to Rule Them All][part_8]

[Part 9: Final Thoughts][part_9]
