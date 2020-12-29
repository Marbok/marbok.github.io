---
layout: post
title: Clojure. Part 2. Advanced language constructs
categories: [Clojure]
excerpt: "Advanced language constructs: multimethods, protocols, type hint, EDN and etc."
---
## Multimethods

A multimethod is a good way to implement a function in which you need to select an action depending on the type. Analog - "case" operator or polymorphism in OOP.

``` clojure
(def figs [{:type :rect     :w 10 :h 20}
           {:type :rect     :w 5  :h 6}
           {:type :circle   :r 15}
           {:type :circle   :r 2}])

(defn perimeter [fig]  ;; analog
  (case (:type fig)
    :circle (* 2 Math/PI (:r fig))
    :rect (* 2 (+ (:w fig) (:h fig)))))

(defn dispatch [fig] (:type fig))

(defmulti p2 dispatch)

(defmethod p2 :circle [fig]
  (* 2 Math/PI (:r fig))

(defmethod p2 :rect [fig]
  (* 2 (+ (:2 fig) (:h fig))))
```

## Protocols
This is a special case of multimethods, for dispatching by the type of the first argument.

``` clojure
(require '[clojure.string :as str])
(import '[java.util Date])

(defprotocol ICal ;; define protocol
  (day [_])
  (month [_ base-month])
  (year [_]))

(defn format-cal [cal]
  (str (day cal) "." (month cal 1) "." (year cal)))

(extend-type Date ;; implement protocol
  ICal
  (day [this] (.getDate this))
  (month [this base-month] (+ base-month (.getMonth this)))
  (year [this] (+ 1900 (.getYear this))))

(defn create-ical [d m y] ;; create protocol in runtime
  (reify ICal
    (day [_] d)
    (month [_ b] (+ b m))
    (year [_] y)))
```

## Records

Records are a more optimized way of structuring data.

``` clojure
(def p {:x 1 :y 2}) ;; like map
(:x p)

(deftype PointT [^long x, ^long y]) ;; like Java class
(def p (PointT. 10 10))             ;; but this way isn't reommended
(.-x p)

(defrecord PointR [^long x ^long y]) ;; use it!!!
(def p (PointR. 10 10))
(:x p)               ;; record has map's interface
(assoc p :x 20)
(:z (assoc p :z 10))
(= (PointR. 10 10) (PointR. 10 10)) ;; work only with records, not with deftype


(defprotocol IFormat 
  (format [this]))

(defrecord Cal [d m y]  ;; combine protol and record
  IFormat
  (format [_] (str d "." m "." y))
  java.lang.Comparable  ;; we can implement Java interface
  (compareTo [_ o2]
    (compare
      [y m d]
      [(:y o2) (:m o2) (:y o2)])))

(.compareTo (Cal. 12 2 2014) (Cal. 14 2 2014)) ;; return -1

(format (Cal. 12 2 2020)) ;; return "12.2.2020"
```

## Work with Java
``` clojure
(import '[java.io File]) ;; like import in java

(File. "." "calling_java.clj") ;; create object

(.toUpperCase "abc") ;; invoke method

(Math/sqrt 25) ;; static method

Math/PI ;; constant

(deftype Struct [a b c])
(.-a (Struct. 1 2 3)) ;; get field
(set! (.-a x) 5)      ;; set field

(doto (java.util.HashMap.) ;; like ->, but
  (.put "a" 1)             ;; for mutable object
  (.put "b" 2))

(defn strlen [^String s]  ;; type hint
  (.length s))


(import '[java.util ArrayList Collection COmparator
                    Timer TImerTask])

;; use reify for implement interface
(let [arr (ArrayList. [3 2 4 1])
      dir -1
      comp (reify Comparator
             (compare [this a b]
               (* dir (- b a))))]
  (Collection/sort arr comp)
  arr) ;; return [1 2 3 4]

;; use proxy for inherit
;; (proxy [SuperClass Ifaces ...] [constructor-args]
;;    (method [args] ...))
(let [task (proxy [TimerTask] []  ;; prints a random number 
             (run []              ;; in a second
               (println (rand))))
  (.schedule (Timer.) task 1000)])
```

## Extention data notion (EDN)

This is a special file's format on base Clojure syntax, like Json.

``` clojure
(pr-str {:a 1}) ;; serialization
(read-string "{:a 1}")  ;; deserialization

(require 'clojure.edn)
(clojure.edn/read-string (slurp "f.edn")) ;; more safer serialization
```

## Metadata

We can add a map with arbitrary data to any object. This's hints for compiler:
``` clojure
(def m {:s 2})
(with-meta m {:doc "123"}) ;; write meta
(meta m)    ;; read meta

;; offen used meta:
(def ^:private x 0)
(def ^:dynamic x 0)
(def ^:const x 0)

;; function doc and type hints are metadata too 
```