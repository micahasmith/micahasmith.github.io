---
title: Clojure A Day-- Loops
layout: post
excerpt: Methods for looping in clojure
---

# Clojure A Day-- Looping

Continuing in learning clojure, today we're going to look at looping constructs. 

### loop/recur

The standard looping construct in clojure is `loop` and `recur`. It's super easy and does all your fancy tail call optimization stuff.

The format of `loop` is `(loop [bindings*] exprs*)`; all that means is that it takes a list of variable/value bindings and it has a function body. 

`recur` works hand in hand with `loop`--it passes back into the loop new values for the `loop` binding arguments.

Example:

{% highlight clojure %}
;;
;; loop/recur
;;

(defn zero-to-nine
  []
  ;; start the loop
  (loop
    ;; set the initial state of the loop
    ;; to one variable--
    ;; x having a value of 0
    [x 0]

    ;; continue while x < 10
    (when (< x 10)
      ;; print out x
      (prn x)
      ;; loop, assigning x to a new value of x+1
      (recur (+ x 1)))))

(zero-to-nine)
{% endhighlight %}

### dotimes

For "range-y" style looping, `dotimes` is a great solution. You simply pass in a single binding of how many times you want it to loop:

{% highlight clojure %}
;;
;; dotimes
;;

(defn zero-to-nine-2
  []
  ;; "do x times"
  (dotimes [x 10]
    (prn x)))

(zero-to-nine-2)
{% endhighlight %}

### while

While is a strange looping construct in clojure (IMO) b/c it expects its test expression to change at some point. 

{% highlight clojure %}
;;
;; while
;;

(def iter (atom 0))
(defn zero-to-nine-3
  []
  (while (< @iter 10)
    (prn @iter)
    (swap! iter inc)))

(zero-to-nine-3)
{% endhighlight %}

That's it!