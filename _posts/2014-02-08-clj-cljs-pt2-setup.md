---
layout: post
title: "Combining Clojure and ClojureScript Libraries - Part 2: Set Up Your Base Project"
date:   2014-02-08 09:57:29
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 

#Combining Clojure and ClojureScript Libraries: A Creative How-To

#Seting Up You Base Project:

  The Clojure and ClojureScript versions of your project, although residing within the same jar, will have different classpaths.  It is important that you isolate these classpaths in a way that will allow you to test and run the two portions of your library independently.  This can be done by adding separate profiles to your project.clj file.  Let’s make a clj and cljs profile.  

{% highlight clojure linenos %}
(defproject myproject "0.1.0-SNAPSHOT"
  :description "Combining Clojure and ClojureScript Libraries"

    :dependencies [[org.clojure/clojure "1.5.1"]
                     [speclj "2.9.4"]]

    :plugins [[speclj "6.9.4"]]

    :aliases {  "clj-test" ["with-profile","clj","spec"]
                "clj-test-auto"  ["with-profile","clj","spec", "-a"]
                "cljs-test" ["with-profile","cljs", "cljsbuild", "test"]
                "cljs-test-auto" ["with-profile","cljs", "cljsbuild", "auto"]
            }


    :profiles {
         :dev { }

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

Now we can manipulate the two classpaths independently of each other.  However, you should still be wary of the fact that the two paths will have to get along somewhat when we put everything together.  This is especially the case for .clj files in your :cljs profile.  With separate profiles you can two different .clj files with the exact same name - one in the clj profile path and the other in the cljs profile path.  Only a single file-name.clj can exist when we combine the paths.
