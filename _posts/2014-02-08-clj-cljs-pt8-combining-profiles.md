---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 8: A Single Jar"
date:   2014-02-08 09:57:23
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 
[sample_project]: https://github.com/AndrewZures/combining_clj_cljs_libraries/tree/one_jar

[part_1]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt1-context.html 
[part_2]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt2-setup.html
[part_3]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt3-dividing-profiles.html
[part_4]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt4-cljx.html
[part_5]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt5-platform.html
[part_6]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt6-platform-and-macros.html
[part_7]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt7-if-macros.html
[part_8]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt8-combining-profiles.html
[part_9]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt9-final-thoughts.html

------

Here is [Sample Project][sample_project] with working code and tests through this part (Part 7) of the tutorial.

#Adding Clojure and ClojureScript Code to a Single Jar

In the previous parts of this tutorial we've built a code base that can deliver the same functionality in both Clojure and ClojureScript.  Now we'll see how we can deploy this functionality in one Jar.  This gives others the ability to import one library and gain your clj and cljs functionality.

Let's go to our `project.clj` file.  We'll add an entirely new profile called "combined"

{% highlight clojure linenos %}
:profiles {
   :combined {
      :dependencies [[org.clojure/math.numeric-tower "0.0.4"]
                     [org.clojure/clojurescript "0.0-2014"] ;necessary or current version of speclj
                     [org.clojure/tools.reader "0.7.10"] ;necessary or current version of speclj
                     [lein-cljsbuild "1.0.2"]]

      :source-paths ["src/clj", "target/generated/src/clj"]
      :resource-paths ["src/cljs", "target/generated/src/cljs"]
      :test-paths ["spec/clj", "target/generated/spec/clj"]

      :cljsbuild ~(let [run-specs ["bin/speclj" "target/tests.js"]]
            {:builds
              {:dev {:source-paths ["src/cljs" 
                                    "spec/cljs"
                                    "target/generated/src/cljs"
                                    "target/generated/spec/cljs"] 
                     :compiler {:output-to "target/tests.js"
                     :pretty-print true}
              }}
              :test-commands {"test" run-specs}})
  }
}
{% endhighlight %}

This profile effectively combines our `:clj` and `:cljs` profiles into one. You'll note that all of the dependencies or both profiles are added to the `:combined`  profile. 

We've also added `:resource-paths` with `src/cljs` and `target/generated/src/cljs`.  This will add our ClojureScript files to our jar, while the normal `:source-paths` will add our Clojure code.

So let's test this new, combined profile. Here, an alias will be helpful:


{% highlight clojure linenos %}
  "combined-tests" ["do" "clean," "with-profile" "combined" "spec," "with-profile" "combined" "cljsbuild" "test"]
{% endhighlight %}

Now if we run `lein combined-tests` everything should pass!

If you `lein with-profile combined jar` and go to your `target/` file, you can `jar tf` the generated `.jar` file and see how both the `.clj` and `.cljs` file are combined into the same library.

All we have to do now is install our project using the `combined` profile.  Again we'll use an alias to make things easy:

{% highlight clojure linenos %}
   "install" ["do" "clean," "with-profile" "combined" "install"]
{% endhighlight %}

Now we can `lein install` and we will have a single library that will work for both Clojure and ClojureScript.

[Part 1: A Bit Of Context][part_1]

[Part 2: Setting Up Your Base Project][part_2]

[Part 3: Dividing Profiles][part_3]

[Part 4: Using CLJX][part_4]

[Part 5: Platform Files][part_5]

[Part 6: Platform Files and Macros][part_6]

[Part 7: Context Aware Macros][part_7]

[Part 8: One Jar to Rule Them All][part_8]

[Part 9: Final Thoughts][part_9]
