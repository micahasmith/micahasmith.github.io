---
title: Clojure A Day--Intro, and Functions
layout: post
excerpt: Beginning learning clojure by analyzing functions
---

# Clojure A Day-- Intro, and Functions

We'll see how long this lasts, but I think I'm going to start posting tiny posts that cover a little bit of clojure here and there in order to help me learn it.

I've already read a few clojure books, and done some tutorials online (such as the awesome [clojure koans](http://clojurekoans.com/) and [modern cljs](https://github.com/magomimmo/modern-cljs)) but I still don't feel like I really know clojure. I'm hoping that writing out examples and playing with tiny bits of the language every day will help fix that.

### Intro

Since clojure is a [lisp](http://www.cs.kau.se/cs/education/courses/dvgc01/lectures/Lisp2.pdf), when you're calling a function it always takes the form of `(function-name arg1 arg2 ...)`. Everything is contained within the parenthesis. Anything 

To make a comment in code, you begin it with two semicolons `;;`.

### Functions

One creates functions via `defn`:

{% highlight clj %}
(defn say-hello
  ;; the following means that the function takes a single parameter called "name"
  ;; the brackets [] designate that parameters form a vector--
  ;;    [1 2 3] is an example vector containing those elements
  [name]
    ;; str is a function that concatenates strings
    (str "hello " name)
  )

(say-hello "micah") ;; returns "hello micah"
{% endhighlight %}

Notice that even the function to define a function follows the lisp rules.

### Built in Function Documentation

Interestingly there's a built in format for documentation-- I could've written the above function like so:

{% highlight clj %}
(defn say-hello
  ;; the following is documentation for the function
  "say-hello takes a name and returns a friendly greeting"
  ;; the following means that the function takes a single parameter called "name"
  ;; the brackets [] designate that parameters form a vector--
  ;;    [1 2 3] is an example vector containing those elements
  [name]
    ;; str is a function that concatenates strings
    (str "hello " name)
  )

(say-hello "micah") ;; returns "hello micah"
{% endhighlight %}

I don't know of another language that has that feature off hand-- I like it. 

### Multiple Arity Functions

This is one of the strangest part of clojure functions for me-- the same function definition can have multiple aritys. (What's the plural of arity?..) This means that you can define a different number of parameters with their own function bodies in the same function definition. 

To demonstrate, we'll make a new function `say-hello2` that accepts either a whole name or a first name and last name.

{% highlight clj  %}
;; making a new function named say-hello2 here...
(defn say-hello2
  ;; the following is documentation for the function
  "say-hello2 takes a name and returns a friendly greeting"
  
	  ;; the first way to call say-hello is with one param
	  ([name]
	    (str "hello " name)
	   )
  
	  ;; the second way to call say-hello is with two params
	  ([firstName lastName]
	    (str "hello " firstName " " lastName)
	  )
  )

(say-hello2 "micah") ;; returns "hello micah"
(say-hello2 "micah" "smith") ;; returns "hello micah smith"
{% endhighlight %}

### End

I think in the next installment we'll look at anonymous functions... we'll see.