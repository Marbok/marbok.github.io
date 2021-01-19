---
layout: post
title: Clojure. reference types
categories: [Clojure]
---
## Identity/Value Separation

In Clojure, ***values*** are immutable. They never change. For example, a number is a value. A map *{:language "Clojure"}* is a value. A vector with 3 elements is a value.

When you attempt to modify a value (a data structure), a new value is produced instead. These are known as persistent data structures (the word "persistent" has nothing to do with storing data on disk).

An ***identity*** is a named entity (e.g., a list of active chat group members or a counter) that changes over time and at any given moment references a value. For example, the current value of a counter may be 42. After incrementing it, the value is 43 but it is still the same counter - the same identity. This is different from, say, Java or Ruby, where variables serve as identities that (typically) point to a mutable value and which are modified in place.
<img src="/images/clojure_ref_types/identity_value.png" />
Identities in Clojure can be of several types, known as ***reference types***.

## Clojure Reference Types
### Overview
In Clojure's world view, concurrent operations can be roughly classified as coordinated or uncoordinated or isolated, and synchronous or asynchronous. Different reference types in Clojure have their own concurrency semantics and cover different kind of operations:

| Type of modification | Synchronous | Asynchronous |
|:---------------------|:-----------:|:------------:|
| Coordinated          |    refs     |       -      |
| Uncoordinated        |    atoms    |    agents    |
| Isolated             |    vars     |       -      |

### Atoms
Atoms are references that change atomically (changes become immediately visible to all threads, changes are guaranteed to be synchronized by the JVM). If you come from a Java background, atoms are basically atomic references from *java.util.concurrent* with a functional twist to them. Atoms are identities that implement synchronous, uncoordinated, atomic updates.

To create an atom, use the *clojure.core/atom* function. Its argument will serve as the atom's initial value:
``` clojure
(def currently-connected (atom []))
```
The line above makes the atom currently-connected an empty vector. To access an atom's value, use *clojure.core/deref* or the *@atom* reader form:
``` clojure
(def currently-connected (atom []))

@currently-connected
;; -> []
(deref currently-connected)
;; -> []
currently-connected
;; -> #<Atom@614b6b5d: []>
```
As the returned values demonstrate, the atom itself is a reference. To access its current value, you dereference it.

To mutate an atom, we can use *clojure.core/swap!*.

*swap!* takes an atom, a function and optionally some other args, swaps the current value of the atom to be the return value of calling the function with the current value of the atom and the args:
``` clojure
(swap! currently-connected conj "chatty-joe")
;; -> ["chatty-joe"]
currently-connected
;; -> #<Atom@614b6b5d: ["chatty-joe"]>
@currently-connected
;; -> ["chatty-joe"]
```
<img src="/images/clojure_ref_types/atom_state.png" />

Occasionally you will need to mutate the value of an atom the same way you would with atomic references in Java: by setting them to a specific value. This is what *clojure.core/reset!* does. It takes an atom and the new value:
``` clojure
@currently-connected
;; -> ["chatty-joe"]
(reset! currently-connected [])
;; -> []
@currently-connected
;; -> []
```
*reset!* may be useful in test suites to reset an atom's state between test executions, but it should be used sparingly in your implementation code. Consider using *swap!* first.

***Atoms*** is the most commonly used concurrent feature in Clojure. It covers many cases and lets developers avoid explicit locking. Atoms cover a lot of use cases and are very fast. It's fair to say that when you need uncoordinated reference types (e.g., not Software Transactional Memory), the rule of thumb is, "start with an atom, then see".

It is not uncommon to initialize an atom in a local and then return it from the function and share a piece of state with other functions and/or threads.