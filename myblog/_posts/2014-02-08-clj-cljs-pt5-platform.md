---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 5: Platform Files"
date:   2014-02-08 09:57:07
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 

####Introduction: ClojureScript and the Case of Two Painfully Similar Libraries

  ClojureScript is an incredible new project that any Clojurian can appreciate.  What could be better than replacing headache-inducing JavaScript with the clean, flowing syntax of Clojure?  But ClojureScript is very new, and in some cases it's lack of polish can be frustrating.  One of the of most frustrating aspects of ClojureScript currently, is that it fails to fails to be a complete abstraction of JavaScript in Clojure.  this comes from the fact that ClojureScript's syntax and abilities do not align perfectly with their Clojure counterparts.  For example, the binding function is not currently implemented in ClojureScript.  That may seem trivial but if you're hoping to transfer a piece of Clojure code to ClojureScript, and that code has a binding call, you're going to have to learn additional aspects of ClojureScript and make a new version of you existing Clojure code to make that piece of code work.  

  ClojureScript is an incredible new project that any Clojurian can appreciate.  What could be better than replacing the often clunky syntax of JavaScript with the clean, flowing syntax of Clojure?  But ClojureScript is very new, and in some cases it's lack of polish can prove to be frustrating.  One of the of most frustrating aspects of ClojureScript currently, is that it fails to be a complete abstraction of JavaScript in Clojure.  This comes from the fact that ClojureScript's capabilities do not align perfectly with their Clojure counterparts.  For example, Clojure’s binding function is not currently implemented in ClojureScript.  This may seem trivial but if you're hoping to transfer a piece of Clojure code to ClojureScript with a binding function call, you now must learn additional aspects of ClojureScript to get that piece of code to work.  You now must also maintain the new ClojureScript version of your code along with your pre-existing Clojure version.  These small differences can add up and before you know it you’re could be maintaining two separate Libraries.  The kicker is of course that these libraries have about 95% code that is common but the 5% of difference simply cannot be resolved.

  We encountered this problem here at [8th Light][8thLight] while attempting to extend our Clojure testing framework [Speclj] to ClojureScript.  There were numerous ClojureScript incompatibilities sprinkled throughout the existing Clojure library.  In our original solution, we built a pre-compiler that could be run against the Clojure code which would switch out all of the incompatibilities with their ClojureScript equivalents.  This solution got us pretty far but it came with a few major drawbacks.  First, the pre-compiler was a one-off design, built specifically for the Speclj project.  There was little chance that it would be maintained, which introduced an added layer of fragility to our project.  More importantly, the ClojureScript version still was its own library.  This led to the oft confusing distinction betweent Speclj (Clojure) and Specljs “with an ‘s’ at the end” (ClojureScript).  Both libraries had to be added independently to a project.   

  Recently, we took another look at our two massively similar libraries and asked if, with the changes to the Clojure/ClojureScript landscape, we could do better.   Not to spoil the fun, but we were able to combine our two libraries into one.  Unfortunately it did take a few absolutely-necessary hacks but that is the cost of working with such a exciting, constantly changing technology like ClojureScript.  We definitely look forward to the day when ClojureScript becomes an effortless abstraction of JavaScript but until then a little creativity will have to suffice.

#Combining Clojure and ClojureScript Libraries: A Creative How-To

#Dividing Your ClassPaths with Profiles:

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


#Generating Code using CLJX:

Next, we'll use the CLJX library.  CLJX translates a .cljx files into separate cljs and clj files.  You can use small #+cljs an #+clj tags to differentiate which forms you would like to be in which version.  In the 8th Light Speclj project, CLJX replaced our hand-rolled pre-compiler.  this gave us the benefit of relying on an open-source, updated dependency instead of our own little program.  

lets configure it:

-project.clj changes
-note location of information

However, CLJX can be a little tricky.  The hardest part may be just keeping everything straight.  All shared clj and cljs files must be renamed as .cljx files.  Normally, these files will be in your src file just like the other source files.  However, they will generate .clj and cljs files in a separate location, normal something like target/generated/.  So now some your “true” source code is in your src file while a good amount could be in your target/generated file.   This also means that your project.clj src and test maps will have to be updated with your target/generated equivalents.

#Using Platform Files to Generate Polymorphic Action:

For most Clojure/ClojureScript discrepancies, a platform dependency will do the trick.  Make two files with the same name - one a .clj and the other .cljs.  Add them to their respective src folders.  For any Clojure/ClojureScript discrepancy add matching functions to each file and implement the platform’s version of that functionality.  

For most cases you can simply :require the file, although it’s best if you use an alias since ClojureScript effectively requires aliases for its imports.  Clojure will obviously find the .clj version of your platform file and run based off of that.  ClojureScript will do the same for the .cljs file.  Here are a few examples:

- throw error
- add

#When All Else Fails, Add an IF:

But some issues can’t be brushed away through runtime polymorphism.  For example, a Throwable is required to be defined at Compile time.  Our previous platform strategy won’t work.  Instead we’ll be limited to macros and we’ll nefariously switch compile time object types during macro expansion.  We’ll do this by determining if a macro file is expanding in a clojure or clojurescript context.  This is the necessary-hack portion of the how-to. 

We’ll determine the macro expansion context by simply checking to see if a file necessary to cljs compilation is present.  This is fragile since that file could change or something, really anything could change the feedback for the if statement leaving us with a broken program.  But such life in the land of ClojureScript.  Here is an example of what might work:

- cljs hack

So here, we’re seeing if we can find the cljs.analyzer namespace and choose which compile-time pieces to add to our macros based off of that result.

Of course clojure macros are a monster in themselves but for this how-to you really don’t need to know much other than that the form after the tilde will be evaluated during expansion time, meaning that at compile time the if statement will be gone and the only thing that will remain will be the result of that if statement.  So by compile time all that should be left is exactly what we want for each platform.  Here is an example using the Throwable object in Java.

- throwable

#NS-Resolve Your Last Concerns Away

So we’re so close to being done but you might run into one additional problem.  What if the cljs part of your (cljs?) hack references a cljs file?  clojure will attempt to find the namespace before the if statement is evaluated, thus you’ll get a compile time error.  Here is an example of this error:

-cljs-compiler issue

We can get around this by using “ns-resolve”  with a namespace and function argument.  Now the cljs.compiler namespace will not need to be resolved until runtime and only when it runs.  Since that code shouldn’t be called in the first place.  


#Combining it All into One:

So now things finally work.  Let’s combine them into one jar and see if it runs in both clojure and clojurescript.  For this we’ll create a production profile that will combine our clj and cljs files.  

