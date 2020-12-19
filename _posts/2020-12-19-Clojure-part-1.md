---
layout: post
title: Clojure. Part 1. The Basics of the language
categories: [Clojure]
---

Clojure is a modern, dynamic, and functional dialect of the Lisp programming language on the Java platform. Like other Lisp dialects, Clojure treats code as data and has a Lisp macro system. The current development process is community-driven, overseen by Rich Hickey as its benevolent dictator for life (BDFL).

Clojure advocates immutability and immutable data structures and encourages programmers to be explicit about managing identity and its states. This focus on programming with immutable values and explicit progression-of-time constructs is intended to facilitate developing more robust, especially concurrent, programs that are simple and fast. While its type system is entirely dynamic, recent efforts have also sought the implementation of gradual typing.


Clojure has a infix notation. For instance:
``` clojure
(+ 1 2) ;; 1 + 2
(f 1 2) ;; f(1, 2)
(f 1 (g 1 2)) ;; f(1, g(1, 2))
```

## Data types in Clojure. 
This language use type from Java, because we have:

| data in Clojure | type in Java / description |
|-----------------|--------------|
| nil  | null |
| 123  | java.lang.Long |
| 1.1  | java.lang.Double |
| 12N  | clojure.lang.BigInt like java.math.BigInteger |
| 1.2N | java.math.BigDecimal |
| true false | java.lang.Boolean |
| "string" | java.lang.String |
| \a | java.lang.Character |
| #"[Rr]egexp" | java.util.regex.Pattern |
| (/ 1 7) | clojure.lang.Ratio - rational number, used for maximum precision |
| :keyword | like enum in Java |
| #inst "2020-01-01" | java.util.Date | 

## Conditions
Basic expression is:
``` clojure
(if <condition> <then> <else>)

;; example
(if (> x 5)  ;; condition
    (- x 5)  ;; if condition true
    (- 5 x)) ;; if condition false
```
*False values* are false and nil. *True values* are true, "string", "", 77, [], [7, 8], 0 and other values.

More shorter version:
``` clojure
(when <condition> <then>)

;; example
(when (= x 3) ;; condition
      65)     ;; return if condition is true, else return nil
```

Also Clojure has insruction case:
``` clojure
(case <value>
      <matching-value1> <expression1>
      <matching-value2> <expression2>
      ...
      <default>)

;; example
(case x             ;; value to be compare
      "v1" (+ x 1)  ;; execute if x = "v1"
      "v2" (- x 1)  ;; execute if x = "v2"
      0)            ;; if x != "v1" and x != "v2", 
      ;; if don't write this value, then throw IlligalArgumentException
```
If we want implement chain of conditions, then we can use "cond":
``` clojure
(cond 
    <condition1> <expression1>
    <condition2> <expression2>
    ...
    :else <default-expression>)
;; example
(cond
    (> x 0) "positive"  ;; return "positeve" if x > 0
    (< x 0) "negative"  ;; return "negative" if x < 0
    :else "neutral")    ;; return "neutral" if x = 0, 
    ;; if don't write :else, then return nil
```

Clojure has standard logical operations: *and*, *or*. Also all condition expressions are lazy. It means Clojure does't evaluate an expression, until it is necessary to do so.

## Functions
We can define a function with the following expressions:
``` clojure
(defn <function-name> [args] ;; for private function used defn-
      "Documentation"
      <function-body>)

;; example
(defn f [x]        ;; a function with a name - "f" and one argument - "x"
      "Print x and return x + 1" ;; Documentation
      (println x)  ;; print value x
      (+ x 1))     ;; and return x + 1
```
Function with several arguments:
``` clojure
(defn sum [x y] ;; a function with two arguments
    (+ x y))

(defn f [x & xs] ;; a function with an arbitary number of args
    xs)

(defn f        ;; define function with 0, 1 and 2 args
    ([] 0)
    ([x] 1)
    ([x y] 2))
```
Clojure has build-in support for checking pre- and postconditions:
``` clojure
(defn f ^{ :pre [(pos? x)]   ;; if pre or post is false,
           :post [(< x 50)]} ;; then throw exception
        [x]
        (* x 5))
```
For tail recursive used special word "recur":
``` clojure
(defn sum [x y]
      (if (x = 0)
          y
          (recur (dec x) (inc y))))
```

Anonymous functions:
``` clojure
((fn [x] (inc x)) 1)

#(inc %) ;; (fn [x] (inc x))
#(+ %1 %2) ;; (fn [x y] (+ x y))
#(str %1 %&) ;; (fn [x & xs] (str x xs))
```

## Namespaces

Namespaces are mappings from simple (unqualified) symbols to Vars and/or Classes. Vars can be interned in a namespace, using def or any of its variants, in which case they have a simple symbol for a name and a reference to their containing namespace, and the namespace maps that symbol to the same var. A namespace can also contain mappings from symbols to vars interned in other namespaces by using refer or use, or from symbols to Class objects by using import. Note that namespaces are first-class, they can be enumerated etc. Namespaces are also dynamic, they can be created, removed and modified at runtime, at the Repl etc.

``` clojure
(ns org.marbok.clojure
    (:require
        [clojure.string :as str]) ;; set an alias
    (:use
        [clojure.core.async]))    ;; trim the full name to async 
```

## Collections

``` clojure
;; Lists
(def l (list 1 2 3 4))
(conj l 4 5 6) ;; (6 5 4 1 2 3 4)
(nth l 2 :default) ;; 3

;; Vector
(def v [1 2 3 4])
(conj v 5 6)  ;; [1 2 3 4 5 6]
(get v 10 :default)  ;; default

;; Map
(def m {:x 1 [1 2 3] 2})
(sorted-map "y" 1 "x" 2) ;; {"x" 2, "y" 1}
(get m :x 6) ;; 1
(assoc m :z 3) ;; {:z 3, :x 1, [1 2 3] 2}
(dissoc m [1 2 3]) ;; {:x 1}
(contains? m :x) ;; true

;; Set
(def s #{1 2 3})
(conj s 2) ;; #{1 2 3}
(contains? s 2) ;; true
(get s 1) ;; 1
(clojure.set/difference #{1 2 3} #{3 4 5}) ;; #{1 2}

;; Equality - used only values
(= [1 2 3] '(1 2 3)) ;; true
(= #{[1 2]} #{'(1 2)}) ;; true
```

Collection implemented an interface "function", then we can:
``` clojure
(def m { 1 "one"
         2 "two"
         3 "three"})

(map m (1 2 3 1)) ;; use m like function -> ("one" "two" "three" "one")

```

## List comprehension
``` clojure
(for [v [[1 2] [3 4]]
      x v
      :let [y (+ 1 x)]
      :when (even? y)
      :while (< y 4)]
    (+ 1 x)) ;; (2)
```

## Exceptions
Clojure uses exceptions from java:
``` clojure
(try
    <actions>
    (catch Exception e
        (handle e))
    (finally
        <finally-action>))
```

## Chaining
``` clojure
(as-> [1 2 3 4 5 6] xs
    (map inc xs)
    (filter even? xs)
    (remove #(> % 5) xs)
    (vec xs)
    (str xs)) ;; "[2 4]"

(->> [1 2 3 4 5 6]
    (map inc)
    (filter even?)
    (remove #(> % 5))
    (vec)
    (str)) ;; "[2 4]"

(-> 1
    (+ 1 2 3 4)
    (- 2)
    (str 3)) ;; "93"

;; some->> and some-> - nil-safe version
```

## Good example:

I found a good exercise on basic language constructs: creating a very simple DBMS. All details in [Github](https://github.com/Marbok/Clojure.csvdb)