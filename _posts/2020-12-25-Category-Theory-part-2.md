---
layout: post
title: Category theory. Part 2. Kleisli category
categories: [Category theory]
---
*In category theory, a Kleisli category is a category naturally associated to any monad T. It is equivalent to the category of free T-algebras. The Kleisli category is one of two extremal solutions to the question Does every monad arise from an adjunction? The other extremal solution is the Eilenberg–Moore category. Kleisli categories are named for the mathematician Heinrich Kleisli.* (Wikipedia). This is an example of classical definitions for category theory. But it's not so scary.

Kleisli category is a category based on monad (I'll write a separate post about monads). We can describe it like functor (a -> mb), where a, b - two types, and m - a container (monad). Standart example is logging in pure function languages.
For instance, we have two functions: upCase (translate word in upper-case) and toWords (separate string on words), and we need write log for them. In an imperative language, would likely be implemented by mutating some global state, as in (pseudocode):
``` java
String logger;

String upCase (String s) {
  logger += "upCase! ";
  return s.upCase();
}
```
But this way is bad - all developers avoid to use a global state for known reasons. We can do next:
1. Function need to return pair: function's result and string for logger.
2. Outside code compose log.

Then in Haskell:
``` sql
type Writer a = (a, String) -- it equivalent pair

-- function for composing Writers
(>=>) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)
m1 >=> m2 = \x ->
  let (y, s1) = m1 x
      (z, s2) = m2 y
  in (z, s1 ++ s2)

return :: a -> Writer a -- add identity function and we get a category!
return x = (x, "")
```

For our function toUpper and toWords we get:
``` sql
upCase :: String -> Writer String
upCase s = (map toUpper s, "upCase ")

toWords :: String -> Writer [String]
toWords s = (words s, "toWords ")

process :: String -> Writer [String] -- compose function
process = upCase >=> toWords
```
If we call function process with argument "one two", then we'll get the pair, when first part is function result: ["ONE", "TWO"] and second part is log: "upCase toWords ".

Also very good example for Kleisli category is Maybe(Haskell) / Optional(Java).