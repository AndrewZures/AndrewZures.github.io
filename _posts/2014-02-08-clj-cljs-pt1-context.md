---
layout: post
title:  "Combining Clojure and ClojureScript Libraries - Part 1: A Bit Of Context"
date:   2014-02-08 09:57:30
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


####Introduction: ClojureScript and the Case of Two Painfully Similar Libraries

  ClojureScript is an incredible new project that any Clojurian can appreciate.  What could be better than replacing vexing JavaScript with the clean, flowing syntax of Clojure?  But ClojureScript is very new, and in some cases its lack of polish can be frustrating.  One of these frustrating aspects is ClojureScript's failure to complete abstract JavaScript using Clojure syntax.  This comes from the fact that ClojureScript's does not currently have all the functionality of its counterpart.  For example, Clojure's `bound?` function is not currently implemented in ClojureScript.  That may seem trivial but if you're hoping to transfer a piece of Clojure code to ClojureScript, and that code has a `bound?` call, you must learn additional aspects of ClojureScript to get your code to work.  You will also have to maintain the new ClojureScript version of your code along with your pre-existing Clojure version.  These small differences can add up and before you know it you could be maintaining two separate Libraries.  The kicker is of course that these libraries have about 95% code in common but the 5% of difference simply cannot be resolved.

We encountered this problem here at [8th Light][8thLight] while extending our Clojure testing framework [Speclj] to ClojureScript.  There were numerous ClojureScript incompatibilities sprinkled throughout the existing Clojure library.  In our original solution, we built a pre-compiler that would run against the Clojure code. This program could switch out all of the incompatibilities with their ClojureScript equivalents.  However, the solution came with a few major drawbacks.  First, the pre-compiler was a one-off design, built specifically for the Speclj project.  There was little chance that it would be maintained, which introduced an added layer of fragility to our project.  More importantly, the ClojureScript version still was its own library.  This led to the confusing distinction between Speclj (Clojure) and Specljs (ClojureScript).  Both libraries had to be added independently to a project.   

Recently, we took another look at our two massively similar libraries and asked if, with the changes to the Clojure/ClojureScript landscape, we could do better.   Not to spoil the fun, but we were able to combine our two libraries into one.  Unfortunately it did take a few absolutely-necessary hacks but that is the cost of working with such an exciting, constantly changing technology like ClojureScript.  We definitely look forward to the day when ClojureScript becomes an effortless abstraction of JavaScript but until then a little creativity will have to suffice.

[Part 1: A Bit Of Context][part_1]

[Part 2: Setting Up Your Base Project][part_2]

[Part 3: Dividing Profiles][part_3]

[Part 4: Using CLJX][part_4]

[Part 5: Platform Files][part_5]

[Part 6: Platform Files and Macros][part_6]

[Part 7: Context Aware Macros][part_7]

[Part 8: One Jar to Rule Them All][part_8]

[Part 9: Final Thoughts][part_9]
