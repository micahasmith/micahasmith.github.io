---
title: Clojure A Day-- Vectors
layout: post
excerpt: An overview of Clojure's vector datatype, including functions that operate on vectors
---

# Clojure A Day-- Vectors

In our last post we dug in to lists (a.k.a `first` and `rest`). Lists seemed rather limiting/constrainging in functionality, wouldn't you agree? The vector fixes all these issues.

### The Vector Data Structure

Data within brackets constitutes a vector. So `[1 2 3]` is a vector.

There are other ways to make vectors as well:

{% highlight clojure %}
;; make a vector
(vector 1 2 3)

;; make a vector with brackets
[1 2 3]

;; make a vector from a list
(vec '(1 2 3))
{% endhighlight %}

So easy, right?

### Vector Access

While `rest` and `first` are available here as well, vectors support getting a variable at an index N. Complexity for getting is log32N.

{% highlight clojure %}
;; rest and first work here as well
(first [1 2 3]) ;; returns 1
(rest [1 2 3]) ;; returns (2 3) -- a list!

;; get the item at index x
;; get complexity is log32n
(get [1 2 3] 0) ;; returns 1

;; this is a neat way of getting a value by index
([1 2 3] 0) ;; returns 1
{% endhighlight %}

The last way to get a value by index is interesting-- you can use a vector as a function. 

### Vector Modification

You can add to the end of a vector, but not the front (without paying an O(n) cost). This would be the exact opposite of lists if not for the fact that you can replace elements at set indexes via `assoc`. 

{% highlight clojure %}
(cons 0 [1 2 3]) ;; returns (0 1 2 3) -- a list!
(conj [1 2 3] 4) ;; returns [1 2 3 4]

;; adding to the front of a vector is O(n) operation
;; so maybe consider a list for that?
(into [0] [1 2 3]) ;; returns [0 1 2 3]

;; replace an element at index n
(assoc [1 2 3] 1 9) ;; returns [1 9 3]

;; get new vector from index (inclusive) to index (exclusive)
(subvec [1 2 3] 1 3) ;; returns [2 3]

;; get new vector via plucking elements at indexes [indexes]
(replace [1 2 3] [0 1]) ;; returns [1 2]

;; reverse in con
(rseq [1 2 3]) ;; returns (3 2 1) -- a list!
{% endhighlight %}


### Special Functions

I like the `distinct?` function.

{% highlight clojure %}
;; are all values distinct?
(distinct? [1 2 3]) ;; returns true
{% endhighlight %}

One thing to note is the `contains?` function. You would think that this would tell you if a vector contains a specific value. Nope-- it tells you if the vector has a value at index X. So just don't use contains? for things other than maps/hash-based data structures. 