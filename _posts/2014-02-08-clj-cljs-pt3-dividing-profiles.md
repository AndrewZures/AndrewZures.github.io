---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 3: Dividing Profiles"
date:   2014-02-08 09:57:28
categories: jekyll update
---

[speclj]:    https://github.com/slagyr/speclj 
[sample_project]: https://github.com/AndrewZures/combining_clj_cljs_libraries/tree/base_project
[part_4]: https://github.com/slagyr/speclj

#Dividing Your ClassPaths with Profiles:

Here is a link to our [sample project][sample_project] that shows the end result of this Part of Tutorial

The Clojure and ClojureScript versions of your project, although residing within the same jar, will have different classpaths.  It is important that you isolate these classpaths in a way that will allow you to test and run the two portions of your library independently.  This can be done by adding separate profiles to your project.clj file.  Let’s make clj and cljs profiles.

Both profiles will use the `org.clojure/clojure` and `speclj` dependency so they should remain outside the `:profile` map.  That means that all we need for the `:clj` profile is:

{% highlight clojure linenos %}
         :clj {
             :source-paths ["src/clj"]
             :test-paths ["spec/clj"]
         }
{% endhighlight %}

For the `:cljs` profile, we'll need the standard `:cljsbuild` information.  Additionally, the sample base project uses `speclj` for testing so we'll see some of the syntax necessary for speclj as well.

{% highlight clojure linenos %}
         :cljs {:dependencies [[org.clojure/clojurescript "0.0-2014"] ;necessary for current version of speclj
                               [org.clojure/tools.reader "0.7.10"] ;necessary for current version of speclj
                               [lein-cljsbuild "1.0.2"]]
               :plugins [[lein-cljsbuild "1.0.2"]]

               :cljsbuild ~(let [run-specs ["bin/speclj" "target/tests.js"]]
                   {:builds
                       {:dev {:source-paths ["src/cljs"  "spec/cljs"]
                              :compiler {:output-to "target/tests.js"
                                         :pretty-print true}
                              :notify-command run-specs}}
               :test-commands {"test" run-specs}})
              }
          }
{% endhighlight %}

Now let make our lives easier by adding testing aliases to our project.  Here is the alias to run your Clojure `speclj` tests using the `clj` profile:

{% highlight clojure linenos %}
    :aliases {  "clj-test" ["with-profile","clj","spec"]
             }
{% endhighlight %}

The ClojureScript testing alias looks very similar:

{% highlight clojure linenos %}
    :aliases {  "clj-test" ["with-profile","clj","spec"]
                "cljs-test" ["with-profile","cljs", "cljsbuild", "test"]
             }
{% endhighlight %}

Now, if you have `Speclj` configured correctly, you can run `lein clj-test` and `lein cljs-test` from the command line to run your tests.

With our profiles in place, we can move ot [Part 4][part_4], where we addded the `cljx` dependency.
And again, here is a link to our [sample project][sample_project] that shows a working sample project reflecting part of the tutorial.

Here's a look at the entire `project.clj` file after our Part 3 changes.

{% highlight clojure linenos %}
(defproject myproject "0.1.0-SNAPSHOT"
  :description "Combining Clojure and ClojureScript Libraries"

    :dependencies [[org.clojure/clojure "1.5.1"]
                     [speclj "2.9.4"]]

    :plugins [[speclj "2.9.4"]]

    :aliases {  "clj-test" ["with-profile","clj","spec"]
                "cljs-test" ["with-profile","cljs", "cljsbuild", "test"]
             }


    :profiles {

         :clj {
             :source-paths ["src/clj"]
             :test-paths ["spec/clj"]
         }

         :cljs {:dependencies [[org.clojure/clojurescript "0.0-2014"] ;necessary for current version of speclj
                               [org.clojure/tools.reader "0.7.10"] ;necessary for current version of speclj
                               [lein-cljsbuild "1.0.2"]]
               :plugins [[lein-cljsbuild "1.0.2"]]

               :cljsbuild ~(let [run-specs ["bin/speclj" "target/tests.js"]]
                   {:builds
                       {:dev {:source-paths ["src/cljs"  "spec/cljs"]
                              :compiler {:output-to "target/tests.js"
                                         :pretty-print true}
                              :notify-command run-specs}}
               :test-commands {"test" run-specs}})

              }
        }
)
{% endhighlight %}