---
layout: post
title: "Combining Clojure and ClojureScript Libraries - Part 2: Set Up Your Base Project"
date:   2014-02-08 09:57:29
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 

[project_part_1_thru_3]: https://github.com/AndrewZures/combining_clj_cljs_libraries/tree/part_1_thru_3_base_proj 

[part_1]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt1-context.html 
[part_2]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt2-setup.html
[part_3]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt3-dividing-profiles.html
[part_4]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt4-cljx.html
[part_5]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt5-platform.html
[part_6]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt6-platform-and-macros.html
[part_7]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt7-if-macros.html
[part_8]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt8-combining-profiles.html
[part_9]: http://andrewzures.github.io/jekyll/update/2014/02/08/clj-cljs-pt9-final-thoughts.html

#Setting Up Your Base Project:

A [Sample Base Project][project_part_1_thru_3] is available if you would like to use it.  It already has [Speclj][speclj] installed and running for both Clojure and ClojureScript.  It also has branches with working code for each part of this tutorial.  We will be referencing this project throughout this tutorial.  

Regardless of whether you use the sample project, your project structure should look similar its structure.  This is especially true for your `src/` and `spec/` paths.  You'll want a structure that resembles `src/file-type/project-name/code.file-type`.  For example, in the sample project we have `src/clj/myproject/core.clj` for Clojure source and `src/cljs/myproject/core.cljs` for the ClojureScript source. 

A similar structure should be used for your tests.

{% highlight clojure %}
myproject
   |
   |--- test
   |     |--- clj
   |     |    |--- myproject
   |     |            |--- test-code.clj
   |     |--- cljs
   |           |--- myproject
   |                   |--- test-code.cljs
   |--- src
         |--- clj
         |     |--- myproject
         |             |--- source-code.clj
         |--- cljs
              |--- myproject
                       |--- source-code.cljs
{% endhighlight %}

#Where We Are
We now have the proper project structure, but that's about it.  In [Part 3][part_3] of this tutorial we'll use profiles to separate our Clojure and ClojureScript classpaths.

[Part 1: A Bit Of Context][part_1]

[Part 2: Setting Up Your Base Project][part_2]

[Part 3: Dividing Profiles][part_3]

[Part 4: Using CLJX][part_4]

[Part 5: Platform Files][part_5]

[Part 6: Platform Files and Macros][part_6]

[Part 7: Context Aware Macros][part_7]

[Part 8: One Jar to Rule Them All][part_8]

[Part 9: Final Thoughts][part_9]
