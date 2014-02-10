---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 6: Platform Files and Macros"
date:   2014-02-08 09:57:25
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 

#Using Platform Files with Macros

In [Part 5][part_5] we used separate `.clj` and `.cljs` files with the same namespace name to isolate the platform differences between Clojure and ClojureScript.  But what if we want to use these files in macros?  You can still use these files in macros but there are a few tweaks needed to our previous platform strategy.

The issue with macros is that there will be no equivalent `.cljs` macro file.  Most macros will stay on the Clojure side of your project. Let's take a look at an example.

Let's add a new test to our `shared_file.cljx` file which will test a simple macro.

Create a `macro.clj` file in your `src/clj/myproject/` folder. You can leave it empty for now.

Next, add the dependency to your `shared_file_spec.cljx` under `:require-macros`. Your namespace should now look like this:

{% highlight clojure linenos %}
(ns myproject.shared-file-spec
    (#+clj :require #+cljs :require-macros
          [speclj.core :refer [describe it should=]]
          [myproject.macros :as macros])
    (:require [speclj.core]
      [myproject.shared-file :as shared-file])
)
{% endhighlight %}

Now add a new test that will evaluate an `abs` function located in your macro file

{% highlight clojure linenos %}
  (it "finds absolute value using macro"
        (should= 2 (macros/abs -2)))
{% endhighlight %}

So now we have a test but it won't pass since we have nothing in our `macro.clj` file

Let's now add a namespace and simple `abs` function to our macro file.

{% highlight clojure linenos %}
(ns myproject.macros
  ;(:require [myproject.platform]) ;uncommenting will break cljs tests
    )

    (defmacro abs [x]
      `(myproject.platform/abs ~x))
{% endhighlight %}

Now here is where things get interesting.  As you can see the `:require` statement is commented out and our macro function uses a fully qualified namespace.  It seems like our macro might not be able to find the platform namespace.  But let's run our tests.

They should pass!

It seems like we should `:require` our platform namespace since the file uses it, but this will actually break the ClojureScript-side tests.  This is because the macro file will always attempt to evaluate the `.clj` version of your platform file if it is defined in the `:require` namespace.  By using a fully-qualified namespace in your macro instead of referencing it through a `:require` statement, the correct platform file will be evaluated at macro expansion.  So, in this case, you should not require the platform file but instead use its fully-qualified namespace where it is needed.

#Where We're At

Now we've seen how to use platform files to isolate platform difference in both normal functions and clojure macros.  These platform files can get you far, but they don't get you all the way there.  In [Part 7[part_7] we'll see how to use an ugly but effective "if" statement to get essentially complete cross-platform functionality.
