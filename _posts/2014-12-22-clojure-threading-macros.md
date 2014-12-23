---
title: Clojure A Day-- Threading Macros
layout: post
excerpt: A look into threading macros like ->, ->>, and as-> in clojure
---

# Clojure A Day-- Threading Macros

In clojure code I read, the threading macros are some of the most frequently used fns/macros I see. I would add the threading macros to the list of things that make clojure friendly to imperative programmers-- they allow values to flow forward through expressions, entirely avoiding the "deeply nested" style clojure/lisp normally imposes.

### An Example

We're all aware of how nested lisp/clojure can get.

{% highlight clojure %}
;; this is your classic lisp
(str "a" (str "b" "c" ( str " " "123"))) ;; returns "abc 123"
{% endhighlight %}

The thread macros break this nesting.

### The -> (Thread First) Macro

The thread first macro threads values into **the first argument of the next expression**. 

{% highlight clojure %}
(->
 (eval "a")
 (str "b" "c")
 (str " " "123")
 ) ;; returns "abc 123"
{% endhighlight %}

I always felt like threading into the first argument of the next expression was kinda weird. What I tended to want when programming clojure was something that would thread into the last argument of the next expression...

### The ->> (Thread Last) Macro

The thread last macro threads values into **the last argument of the next expression**. 

{% highlight clojure %}
(->>
 (eval "c")
 (str "a" "b")
 (str "123" " ")
 ) ;; returns "123 abc"
{% endhighlight %}

So its kinda like it pops the value from the previous expression onto the next expression. Super useful!

### The as-> Macro

The `as->` macro is one of my favorite things in clojure. You start with a value, and a variable, and you thread the results of expressions through successive expressions by specifying where you want the value to go.

So where `->` and `->>` expect values to be either the first or last arguments, `as->` allows you to put values **wherever you want**. I like that. 

{% highlight clojure %}
;; "a" is our value, x is our variable/specifier
(as->
 "a" x 
 (str "mic" x "h")
 (str x "smith")
 ) ;; returns "micahsmith"
{% endhighlight %}

So in our example above, the value went wherever the `x` variable was. I think I especially like this having spent so many years writing LINQ expressions.