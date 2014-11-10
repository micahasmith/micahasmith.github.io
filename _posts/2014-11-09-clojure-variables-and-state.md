---
title: Clojure A Day-- Variables
layout: post
excerpt: A look into variables/state in Clojure, which is surprsingly not straightforward.
---

# Clojure A Day-- Variables

`variable = value`

In most languages, that's all there is. Sure, some languages allow you to decide whether or not the variable should be immutable or mutable, or if its a reference or a value type-- but for the most of us, variables are just variables. There's nothing fancy about them.

Granted I've been slammed at work, but I've been trying to figure out variables in clojure for the last two months. Its just not like other languages. I've read [aphyr's "Clojure from the ground up: state"](http://aphyr.com/posts/306-clojure-from-the-ground-up-state) post, chapters from ["The Joy of Clojure"](http://www.manning.com/fogus2/), and chapters from ["Clojure for the Brave and True"](http://www.braveclojure.com/), yet as I write this I know that I have yet to "get it." Hopefully through researching this post I start to piece it together.

To begin with, there are multiple types of variables/state in clojure.

### Type 1: The "let" Binding

The `let` binding is the simplest. `let` is a function that binds a value immutably within its function body. So it's purely immutable, scoped, unchangeable state.

{% highlight clojure %}
;; let binding example
;; a function to greet someone
(defn greet 
   [name]
   ;; create a variable 
   ;; to hold part of the greeting
   (let [greeting "hi "]
     (str greeting name)
     )
   )

(greet "micah") ;; returns "hi micah"
{% endhighlight %}

Everything is great! So in this example, `let` is a function that binds the variable `greeting` to its value and makes it accessible to the scope of the rest of the function.

Note that if i tried to do this:

{% highlight clojure %}
;; let binding example
;; a function to greet someone
(defn greet 
   [name]
   ;; create a variable 
   ;; to hold part of the greeting
   (let [greeting "hi "]
     (str greeting name)
     )
   )

(greet "micah") ;; returns "hi micah"

;; here we try to use the greeting variable again
;; this throws a big ol' error
(str greeting "micah")
{% endhighlight %}

I would get the error "`clojure.lang.Compiler$CompilerException: java.lang.RuntimeException: Unable to resolve symbol: greeting in this context`" because the value doesn't exist there.

### Type 2: "vars"

Vars appear the most like classic mutable programming language variables, with a few subtle differences.

You define a var using the `def` function. Note that to reassign to the same value you use `def` again.

**It is important to note that `def` binds a value to the entire namespace its in.** In other words, doing a `def` within a function is equivalent to doing a `def` in that function's enclosing namespace-- the value is now there. Just because you `def` in a function it **does not** mean that the function is scoped only in that function. If you're looking for that perhaps `let` is a better alternative?

{% highlight clojure %}
;; define a variable with def
(def myname "micah")
(str "hi " myname) ;; "hi micah"

;; since vars are mutable, we can reassign to them
(def myname "jenna")
(str "hi " myname) ;; "hi jenna"
{% endhighlight %}

When you change a var value, it changes everywhere.

### Type 3: "Dynamic vars"

Ready for it to get weird?

Dynamic vars are also mutable. However, dynamic vars can be changed in such a way that it only happens within a particular scope. So where regular vars changes' propagate everywhere, dynamic vars *can* change just in one area.

Its a generally accepted convention to name dynamic vars with asterisks around it like `*myname*`.

You make a dynamic var with `def ^:dynamic [variablename] [value]`, and you "rebind" values to it via the `binding` function.

{% highlight clojure %}
;; define our dynamic var
;; using the recommended *[name]* style
(def ^:dynamic *name* "micah")
(str "hi " *name*) ;; "hi micah"

;; a function that specifically prints
;; our dynamic variable
(defn say-hi
  []
  (str "hi " *name*)
  )

;; we set the value of *name* to "billybob"
;; JUST for the function say-hi
(binding [*name* "billybob"] (say-hi)) ;; returns "hi billybob"

;; `eval` evaluates a variable
(eval *name*) ;; returns "micah"
{% endhighlight %}

Kind of interesting how the value of `*name*` was changed within the `binding` form just for, `say-hi`, right?

### Type 4: "Atoms"

You start to see some of the beauty of clojure with atoms; its a really well thought out, needed language feature.

An `atom` is a mutable variable that provides coordinated get/set functionality. This is **so perfect** for providing thread safety to variable state. 

To set an atom's value to a new, definite value, you use `reset!`.

To apply a function to an atom's value that uses the value and returns a new value for it, use `swap!`. `swap!` ensures that within the function you pass to it no other thread gets to update the atom's value.

Note that both `reset!` and `swap!` return the values as well.

Example:

{% highlight clojure %}
;; create an atom
(def a-name (atom "micah"))

;; get the value from the a-name atom
;; via the `deref` function
(deref a-name) ;; returns "micah"

;; set the value to jenna
(reset! a-name "jenna") ;; returns "jenna"

;; set the value to name + firstname
;; via a function that does it in a coordinated manner
(swap! a-name #(str % " " "smith")) ;; returns "jenna smith"
{% endhighlight %}

So `swap!` applies the value to a function which then returns a new value, in a coordinated, synchronous manner. `reset!` is just for directly setting the atom's value to a new value.

### Type 5: "refs"

Clojure slam dunks mutable state with refs. `ref` provides a means to have transactional state.

So if you want to update multiple values with the safety of a transaction, do so with `ref`.

{% highlight clojure %}
;; create two refs
(def micah-name (ref "micah"))
(def jenna-name (ref "jenna"))

;; update both names in a transaction
(dosync
  (ref-set micah-name "micah smith")
  (ref-set jenna-name "jenna smith"))

;; use `deref` to get a ref value
(deref micah-name) ;; returns "micah smith" 
{% endhighlight %}

There are *many* ways to handle setting the values inside of `dosync`:

- `ref-set` as we saw linearly updates the values, e.g. `(ref-set micah-name "micah smith")`
- `alter` applies a function (kinda like `swap!` with atoms), e.g. `(alter micah-name #(str $ " smith"))`
- `commute` is the same as `ref-set`, except not necessarily in order. This grants a speed boost. Example, `(commute micah-name "micah smith")`

If you're going to use one ref's value to set another's in a `do-sync`, you need to use `ensure`.

{% highlight clojure %}
;; using ensure
(def mname (ref "micah"))
(def jname (ref "jenna"))
(def family (ref "smith"))

;; function to uppercase things
(def up-it clojure.string/capitalize)
(up-it "micah") ;; returns "Micah"

(dosync
 (alter mname up-it)
 (alter jname up-it)
 (alter family #(str (ensure mname) " and " (ensure jname) " " (up-it %))))

(deref family) ;; returns "Micah and Jenna Smith"
{% endhighlight %}

### Type 6: "agents"

Agents apply functions asyncronously against a mutable or immutable object. One uses the `agent` function to create an agent.

To push a function onto the stack of functions to be applied to an agent, one uses the `send` function.

If you want to block the thread until all functions are applied to an agent, you can do so via the `await` keyword. Note that this is entirely optional.

You can get the values of agents via the `deref` function.

{% highlight clojure %}
;; create an agent
(def my-agent (agent 0))

;; "queue" functions into agent, asynchronously
;; here we queue adding 1 to it
(send my-agent #(+ % 1))

;; block the thread until all fns are evaluated
;; on the agent
(await my-agent) ;; just blocks, returns nil

;; get the value of the agent
(deref my-agent) ;; returns 1
{% endhighlight %}

### Conclusion

That's it! You have to admit that the options clojure gives to a programmer for variables are pretty amazing, right?