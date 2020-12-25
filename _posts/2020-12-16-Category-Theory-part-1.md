---
layout: post
title: Category theory. Part 1. Basic definitions
categories: [Category theory]
---

***Category theory*** is a high level language or abstraction (the highest at this moment), that describes relationships between different objects, but does not pay attention to their structure.

Each category consists of *objects* and *morthisms*. ***Object*** is a basic element, like physical point mass. And ***morthisms*** is a mapping or transformation function between objects. Each morthism have source and target objects.  
<img src="/images/category_theory_part1/simple_category.png" />
Then _**category**_ is a graph, which has next properties:
1. *Binary operation.* If there are morthisms f:a&#x27f6;b and g:b&#x27f6;c, then there is morthism g&#9900;f or "g after f".
2. *Associativity.* If f:a&#x27f6;b, g:b&#x27f6;c and h:c&#x27f6;d, then f&#9900;(g&#9900;h) = (f&#9900;g)&#9900;h.
3. *Identity.* For every object x, there exists a morphism 1<sub>x</sub>:x&#x27f6;x called the identity morphism for x, such that for every morphism f:a&#x27f6;b, we have 1<sub>b</sub>&#9900;f = f = f&#9900;1<sub>a</sub>.

Category with one object and one or same arrows is called ***monoid***. Traditionally, a monoid is defined as a set with a binary operation.
All that’s required from this operation is that it’s associative, and that there is one special element that behaves like a unit with respect to it.
For instance, natural numbers with zero form a monoid under addition. Associativity means that: (a + b) + c = a + (b + c).
The neutral element is zero, because: 0 + a = a and a + 0 = a.

In Haskell we have a class Monoid:
``` sql
class Monoid m where
    mempty :: m
    mappend :: m -> m -> m
```
where *mempty* - neutral element, and *mappend* - binary operation.

For instance, String like monoid:
``` sql
instance Monoid String where
    mempty = ""
    mappend = (++)
```
