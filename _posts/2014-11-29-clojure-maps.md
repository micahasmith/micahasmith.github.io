---
title: Clojure A Day-- Maps
layout: post
excerpt: A look into the map data structure in clojure
---

# Clojure A Day-- Maps

The map data structure in clojure is entirely similar to ones in other languages-- key/value based data storage/access. In most other languages that i use, map access/setting is O(1), which is part of their advantage. In clojure access is log32n but at least we're getting the benefits of immutable, persistent storage.

Pretty easy to make a map:

{% highlight clojure %}
;; make a map 
(hash-map :key "value" )

;; make a map with curly brackets
{:key "value"}

;; nested map
{:key {:sub-key "value"}}
{% endhighlight %}

Clojure has [a few types of maps](http://clojuredocs.org/quickref#maps). We'll only be investigating hash maps in this article, though. 

### Map Data Access

Its super easy to get data out of a map. 

{% highlight clojure %}
;; get a key's value
(get {:key "val"} :key) ;; returns "val"

;; get a non-existing key/value
(get {:key "val"} :missing-key) ;; returns nil

;; another means to get a key's value
({:key "val"} :key) ;; returns "val"

;; get the keys of the map
(keys {:key1 "val1" :key2 "val2"}) ;; returns (:key1 :key2)

;; get the values of the map
(vals {:key1 "val1" :key2 "val2"}) ;; returns ("val1" "val2")
{% endhighlight %}

Note that like vectors, you can use maps as the functions to access their values. 

### Modification

Again, maps are immutable and persistent-- so functions that modify them return new objects with the old object left unchanged.

`assoc` is (from what I observe) the most used method to modify a map. 

{% highlight clojure %}
;; use assoc to "upsert" values in a map
(assoc {:key "val1"} :key "val2") ;; returns {:key "val2"}

;; use dissoc to remove entries in a map
(dissoc {:key1 "val1" :key2 "val2"} :key2) ;; returns {:key1 "val1"}

;; use assoc-in to update a map/sub-section of a map
;; this returns {:key1 {:sub-key1 "val2"}}
(assoc-in {:key1 {:sub-key1 "sval1"}} [:key1 :sub-key1] "val2")

;; use update-in to update a map/sub-section of a map via a function application
;; this returns {:key1 {:sub-key1 "SVAL1"}}
(update-in {:key1 {:sub-key1 "sval1"}} [:key1 :sub-key1] clojure.string/upper-case)
{% endhighlight %}

Interesting how `assoc-in` takes a map and a vector and updates "within" a structure. Seems really useful for nested scenarios.