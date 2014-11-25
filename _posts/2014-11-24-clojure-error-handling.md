---
title: Clojure A Day-- Error Handling
layout: post
excerpt: Handling errors in clojure
---

# Clojure A Day-- Error Handling

You can't program in a language where you don't understand how errors are handled. 

### try/catch/finally

Nothing terribly new to learn here if you come from a Java/.NET background-- clojure has `try`/`catch`/`finally` error handling. Same sort of propagation. 

Example:

{% highlight clojure %}
;;
;; error handling
;;

(defn throw-err []
  ;; start our "try block"
  (try
    ;; what's 5 divided by zero again?
    (/ 5 0)
    
    ;; output the what the error is
    (catch Exception e (str "exception was " (.getMessage e)))
    
    ;; no matter what, let's end by printing "oh well"
    (finally prn "oh well")))


;; run the function above here
;; returns "exception was Divide by zero"
;; outputs "oh well" at the end as well
(throw-err)   
{% endhighlight %}