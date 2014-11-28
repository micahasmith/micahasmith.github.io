---
title: Clojure A Day-- Lists
layout: post
excerpt: An overview of Clojure's list datatype, including functions that operate on lists
---

# Clojure A Day-- Lists

Lists are central to clojure as they are to other lisps. If you're not familiar with lisps, a strangely interesting part of clojure/lisps is that all data/functions/everything are in a sense, a list. If you're interested in diving more into this I would recommend reading [John Hughes's paper "Why Functional Programming Matters"](http://worrydream.com/refs/Hughes-WhyFunctionalProgrammingMatters.pdf). For now we're going to focus on lists in the classical/data structure sense.




### The List Data Structure (`first` and `rest`)

{% highlight clojure %}
;; make a list
(list 1 2 3)

;; make a list via quoting
'(1 2 3)

;; make a list named "a-list"
(def a-list '(1 2 3))

;; first and rest-- basis of list functions
(first a-list) ;; returns 1
(rest a-list) ;; returns (2 3)
{% endhighlight %}

We're talking about linked lists here. Lists in clojure have similar properties to other FP lists; namely that you only have access to the list via two functions:

- the first element of the list via the `first` function
- the elements that are not the first element via the `rest` function

If you want an element that's deeper into the list, you gotta do the work to get there; you can't immediately jump to the element at index N in a list. You have to move from the beginning of the list *to* there. 

{% highlight clojure %}
;; function to get an element at index X in a list
(defn list-get [li index]
  (loop [iter index 
         the-list li]
    (cond
     
     ;; if the list is at this point empty, return nil
     (empty? the-list) 
       nil
     
     ;; if the list is at this point zero, return the first element
     (zero? iter) 
       (first the-list)
     
     ;; else recurse deeper into the rest of the list
     :else 
        (recur (dec iter) (rest the-list) ))))

(list-get '(1 2 3) 1) ;; returns 2
{% endhighlight %}

All based on `first` and `rest`! That's just how `list` works. 

So when you think about lists in clojure from here on out, you will only think "`first` and `rest`". Lists in clojure = only `first` and `rest`. Say it a few times. Great.

It's important to know how functions work on empty lists.

{% highlight clojure %}
;; rest and first on empty lists stay classy
(def empty-list '())
(rest empty-list) ;; returns ()
(first empty-list) ;; returns nil
{% endhighlight %}




### Peek and Pop

`peek` and `pop` supposedly help implement queue functionality? I think they're redundant and not as cool as `first` and `rest`.

{% highlight clojure %}
;; peek and pop on empty lists throw exceptions, FYI
(def a-list '(1 2 3))
(peek a-list) ;; returns 1
(pop a-list) ;; returns (2 3)
{% endhighlight %}





### Modifying Lists

Lists in clojure are immutable, persistent data structures. This means that you **cannot change** a list. All functions that modify lists actually return new lists-- the "source list" remains unchanged. 

Let's investigate this:

{% highlight clojure %}
;; lists are peristent, immutable
(def list1 '(1 2 3))

;; cons adds an element to the front of a list
;; and returns a new list
(def list2 (cons 0 list1))

list1 ;; it's (1 2 3)
list2 ;; it's (0 1 2 3)
{% endhighlight %}

See-- in the example above we couldn't modify the list. 

A strange thing about lists-- you can really only change the list in respect to the first element. For instance, you can't add an element to the end of a list easily in clojure.




### Other Notable List Functions

**count**

`count` returns the number of items in the list in O(1) time, surprisingly. So `(count '(1 2 3))` would return `3`.

**conj**

`conj` is short for conjoin. It's like `cons` with the args reversed-- `(conj '() 0)` returns the list `(0)`.

**empty?**

`empty?` returns true/false for if the list is empty.

**distinct?**

`distinct?` returns true/false for if all elements are distinct. 


### All Code Is Lists

Check this out:

{% highlight clojure %}
;; throwing quotes in front of functions
;; cause we're acting like functions are lists....
(first '(defn hi [] (prn "hi")))
{% endhighlight %}

It returns `defn`! Functions **are** lists!

