---
title: Clojure + MS SQL Azure
layout: post
excerpt: How to get clojure/jdbc set up to interact with a sql azure database
---

# Clojure + MS SQL Azure

Surprisingly easy to get set up. Here's the steps:

** Init Your Lein Project **

If you don't already have an existing lein project setup, run `lein new [proj name]` in order to get a lein project setup. This will give you a project.clj file which is necessary for following steps. 

** Download the MS SQL JDBC Jars**

At the time of writing, these are [here](http://www.microsoft.com/en-us/download/details.aspx?id=11774).

** Add the Jars to Your Project **

To do this:

- Add the jar files to your `/resources` directory
- Add the jars to your classpath via adding the following to your project.clj file:


{% highlight clojure %}
:resource-paths ["resources/sqljdbc4.jar" "resources/sqljdbc.jar"]
{% endhighlight %}

** Add Clojure JDBC Jars **

Also ensure that you have the following dependencies registered in your project.clj. 

{% highlight clojure %}
  :dependencies [
                 [org.clojure/clojure "1.6.0"]
                 [org.clojure/java.jdbc "0.3.6"]
                 ])
{% endhighlight %}

After you have done this, run `lein deps` to install the dependencies. 

** Add The Code! **

Last step! Here is some example code that connects to sql azure and runs a query:

{% highlight clojure %}
(ns cljs-sqltest.core
  ;; register the jdbc namespace here
  (:use [clojure.java.jdbc])
  )

(def db-spec {:classname "com.microsoft.jdbc.sqlserver.SQLServerDriver"
              :subprotocol "sqlserver"
              :subname "//your.db.address:1433;Initial Catalog=dbname;"
              :database "dbname"
              :user "dbusername"
              :password "dbpassword"})

(with-db-connection [connection db-spec]
  (let [rows (query connection
                         ["select * from People"])]
    ;; this would output the "name" column in the People db
    (doseq [row rows] (println (:name row)))))
{% endhighlight %}

### Thats It!

Not bad!