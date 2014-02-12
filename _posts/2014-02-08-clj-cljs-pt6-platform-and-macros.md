---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 6: Platform Files and Macros"
date:   2014-02-08 09:57:25
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:  https://github.com/slagyr/speclj 
[sample_project]: https://github.com/AndrewZures/combining_clj_cljs_libraries/tree/macro_dependency

[part_1]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt1-context.html 
[part_2]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt2-setup.html
[part_3]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt3-dividing-profiles.html
[part_4]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt4-cljx.html
[part_5]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt5-platform.html
[part_6]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt6-platform-and-macros.html
[part_7]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt7-if-macros.html
[part_8]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt8-combining-profiles.html
[part_9]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt9-final-thoughts.html


Here is [Sample Project][sample_project] with working code and tests through this part (Part 6) of the tutorial.

#Using Platform Files with Macros

In [Part 5][part_5] we used separate `.clj` and `.cljs` files with the same namespace name to isolate the platform differences between Clojure and ClojureScript.  But what if we want to use these files in macros?  You can still use these files in macros but there are a few tweaks needed to our previous platform strategy.

The issue with macros is that there will be no equivalent `.cljs` macro file.  Most macros will stay on the Clojure side of your project. Let's take a look at an example.

#Platform Files and Macros: An Example

Let's add a new test to our `shared_file.cljx` file which will test a simple macro.

Create a `macro.clj` file in your `src/clj/myproject/` folder. You can leave it empty for now.

Next, add the dependency to your `shared_file_spec.cljx` under `:require-macros`. 
Your namespace should now look like this:

{% highlight clojure linenos %}
(ns myproject.shared-file-spec
    (#+clj :require #+cljs :require-macros
          [speclj.core :refer [describe it should=]]
          [myproject.macros :as macros])
    (:require [speclj.core]
      [myproject.shared-file :as shared-file])
)
{% endhighlight %}

For this example, we'll test add our existing `platform/abs` function to a macro.  Add a new test that will evaluate an `abs` function located in your macro file

{% highlight clojure linenos %}
  (it "finds absolute value using macro"
        (should= 2 (macros/abs -2)))
{% endhighlight %}

So now we have a test but it won't pass since we have nothing in our `macro.clj` file

Let's now add a namespace and simple `abs` function to `macro.clj`.

{% highlight clojure linenos %}
(ns myproject.macros
  ;(:require [myproject.platform]) ;uncommenting will break cljs tests
    )

    (defmacro abs [x]
      `(myproject.platform/abs ~x))
{% endhighlight %}

Now here is where things get interesting.  As you can see we've commented out the  `:require` statement and our macro function uses a fully qualified namespace.  It seems like our macro might not be able to find the platform namespace.  But let's run our tests.

They should pass!

Now uncomment the `:require` statement and rerun your tests.  Your ClojureScript tests will fail to run!

It seems like we should `:require` our platform namespace since the file uses it, but this will actually break the ClojureScript-side tests.  This is because the macro file will always attempt to evaluate the `.clj` version of our platform file if it is defined in the `:require` namespace.  By using a fully-qualified namespace in our macro instead of referencing it through a `:require` statement, the correct platform file will be evaluated at macro expansion.  So, in this case, you should not require the platform file but instead use its fully-qualified namespace where it is needed.

#Where We're At

Now we've seen how to use platform files to isolate platform difference in both normal functions and clojure macros.  These platform files can get you far, but they don't get you all the way there.  In [Part 7][part_7] we'll see how to use an ugly but effective "if" statement to get essentially complete cross-platform functionality.

[Part 1: A Bit Of Context][part_1]

[Part 2: Setting Up Your Base Project][part_2]

[Part 3: Dividing Profiles][part_3]

[Part 4: Using CLJX][part_4]

[Part 5: Platform Files][part_5]

[Part 6: Platform Files and Macros][part_6]

[Part 7: Context Aware Macros][part_7]

[Part 8: One Jar to Rule Them All][part_8]
