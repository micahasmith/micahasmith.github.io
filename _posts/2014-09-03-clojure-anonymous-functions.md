---
title: Clojure A Day-- Anonymous Functions
layout: post
excerpt: Anonymous functions in clojure
---

# Clojure A Day-- Anonymous Functions

Anonymous functions are one of my favorite language features. I don't think I could work with a language that doesn't have them (here's lookin' at you, java 7).

### The "fn" keyword

The first way to go about anonymous functions is via the fn keyword.

{% highlight clojure %}
;; different version of our say-hello function from our last lesson
(fn [name] 
  (str "hello " name)
  )
{% endhighlight %}

Since one can call a function with the form `(function arg1 arg2 arg3)` we could call this function like so:

{% highlight clojure %}
;; different version of our say-hello function from our last lesson
(
  ;; first we have the function
  (fn [name] 
    (str "hello " name)) 
    ;; here we pass in the first argument
    "micah"
  ) ;; returns "hello micah"

{% endhighlight %}

Kind of like doing the following in JS:

{% highlight js %}
(function(name){
	return "hello "+name;
})("micah");
{% endhighlight %}

### The % Keyword

I love this second method. In general, I love any anonymous function shorthand. C#'s/LINQ's `j => j` for instance is a personal favorite.

Here's the `%` based version, note that there are two different ways to use it. Both start with a `#` at the beginning:

{% highlight clojure %}
( 
   ;; here is the entire function!
   #(str "hello " %) 
   ;; here is the arg1 to the function
   "micah"
 ) ;; returns "hello micah"
{% endhighlight %}

So in the above version, `%` refers to the **one and only argument**.

If you have multiple arguments, then there is a different way of using `%`:

{% highlight clojure %}
( 
   ;; here is the entire function!
   #(str "hello " %1 " " %2) 
   ;; here is the arg1 and arg2 to the function
   "micah" "smith"
 ) ;; returns "hello micah smith"
{% endhighlight %}

So you can use `%[arg number]` to reference multiple arguments.
