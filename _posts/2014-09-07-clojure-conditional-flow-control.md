---
title: Clojure A Day-- Conditional Flow Control
layout: post
excerpt: Methods for conditional flow control in clojure
---

# Clojure A Day-- Conditional Flow Control


Let's take a look at some entry level conditional flow control techniques in clojure.

### The Classic If/Then/Else

The form for if then is pretty simple-- `(if [true/false expression] [expression to call if truthy] [expression to call if falsey])`. The final else (fn to call if falsey) is optional.

**In clojure, both `false` and `nil` evaluate to logical false.** Its kinda like JS in that regard.

{% highlight clojure %}
;; example if/then/else
(defn greeting  
  [name]
  ;; because this if is the last form in the function
  ;; its result is what gets returned
  (if (= name "jenna")
    ;; if true do this
    (str "hello " name)    
    ;; if false do this
    (str "hi " name)    
    )
  )
(greeting "micah") ;; outputs "hello jenna"
(greeting "jenna") ;; outputs "hi micah"
{% endhighlight %}

Be careful with the placement of your if statement. Coming from JS/.NET i would've expected the result from the `if` to be returned as the final return, but nope-- the rule is still that the last expression/value is what is returned:

{% highlight clojure %}
;; be careful putting anything after an if
(defn greeting2  
  [name]
  (if (= name "jenna")
    ;; if true do this
    (str "hello " name)    
    ;; if false do this
    (str "hi " name)    
    )
  ;; function directly following an if
  ;; sorry is always returned
  (str "sorry")
  )

(greeting2 "micah") ;; outputs "sorry"
(greeting2 "jenna") ;; outputs "sorry"
{% endhighlight %}

Good to watch for!

### The if-not Function

I'm kinda surprised that a language would have an if-not. But then again F#'s `not` operator was a bit strange at first as well. I guess its nice to not have to negate the test.

{% highlight clojure %}
;; example of if-not
(defn greeting3 
    [isEvening]
  (if-not isEvening
    "good morning!"
    "good night!"
    )
  )

(greeting3 true) ;; outputs "good night!"
(greeting3 false) ;; outputs "good morning!"
{% endhighlight %}

### The when and when-not Functions

`when` and `when-not` are effectively `else`-less `if` statements. Example:

{% highlight clojure %}
;; example of when
(defn greeting4
  [name]
  (when (= name "micah")
    (str "its micah!")
    )
  )

(greeting4 "micah") ;; returns "its micah!"
(greeting4 "jenna") ;; returns nil
{% endhighlight %}




### The cond Function-- Clojure's "else if"

If you're looking for `else if` style flow, look no farther than `cond`. Notice how you end a `cond` statement via the `:else` keyword. 

{% highlight clojure %}
(defn greeting5
  [name]
  (cond
   (= name "micah") "hello micah!"
   (= name "jenna") "hi jenna!"
   :else (str "nice to meet you, " name "!")
   )
  )

(greeting5 "micah") ;; returns "hello micah!"
(greeting5 "jenna") ;; returns "hi jenna!"
(greeting5 "john") ;; returns "nice to meet you, john!"
{% endhighlight %}




### The Cooler "else if" -- condp

`condp` is awesome. This is the first time I've seen a language that has something similar to a case/switch statement that isn't horrendously ugly. What's more, it seems like there are some ways to get `condp` close to pattern matching-- though I won't cover it here (because it's still above my level).

Notice how with condp instead of starting the "test expressions" with real functions, you can start them with real values for if you're looking to match exactly to what you're looking for.

Alternatively, you can use an expression and then the `::>` keyword. With this form, (from what I can tell) any expression that evaluates to true will work. Notice how in the following example we used `true ::>` as a "catch-all."

If you don't have a matching expression/clause expect an exception to be thrown.

{% highlight clojure %}
(defn greeting6
  [name]
  (condp = name
   "micah" "hello micah!"
   "jenna" "hi jenna!"
    true ::> (str "nice to meet you, " name "!")
   )
  )

(greeting6 "micah") ;; returns "hello micah!"
(greeting6 "jenna") ;; returns "hi jenna!"
(greeting6 "john") ;; returns "nice to meet you, john!"
{% endhighlight %}
