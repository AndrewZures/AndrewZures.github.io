---
layout: post
title: "Combining Clojure and ClojureScript Libraries - Part 2: Set Up Your Base Project"
date:   2014-02-08 09:57:29
categories: jekyll update
---

[8thLight]: https://8thlight.com
[speclj]:    https://github.com/slagyr/speclj 
[sample_project]: https://github.com/AndrewZures/combining_clj_cljs_libraries/tree/base_project
[part_3]: http://localhost:4000/jekyll/update/2014/02/08/clj-cljs-pt3-dividing-profiles.html

#Setting Up Your Base Project:

A [Sample Base Project][sample_project] is available if you would like to use it.  It already has [Speclj][speclj] installed and running for both Clojure and ClojureScript.  We will be referencing this project throughout this tutorial.  

Regardless of whether you use the sample project, your project structure should look similar.  This is especially true in how your `src/` and `spec/` paths should look.  You'll want a structure like `src/file-type/project-name/code.file-type`.  For example, in the sample project we have `src/clj/myproject/core.clj` for Clojure source and `src/cljs/myproject/core.cljs` for the ClojureScript source. 

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
We now have a base project template that can be used to get a sense of the structure of our project, but that's about it.  In [Part 3][part_3] of this tutorial we'll use profiles to separate our Clojure and ClojureScript classpaths.
