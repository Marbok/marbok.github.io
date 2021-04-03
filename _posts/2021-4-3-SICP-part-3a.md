---
layout: post
title: SICP. Part 3. A Simulator for Digital Circuits
categories: [SICP, Clojure]
---
Now I'm learning chapter 3. So far, I've read about imperative programming and mutable state. Not so interesting, if you use it several years.
I've realised [queue](https://github.com/Marbok/sicp/blob/main/src/sicp/chapter3/solution_3_21.clj) and [deque](https://github.com/Marbok/sicp/blob/main/src/sicp/chapter3/solution_3_23.clj), but author get more complicated example: "A Simulator for Digital Circuits". All code in Clojure is [here](https://github.com/Marbok/sicp/tree/main/src/sicp/chapter3/simulator_digit_circuits).

The simulator isn't only program, it's small [DSL](https://en.wikipedia.org/wiki/Domain-specific_language#:~:text=A%20domain-specific%20language%20(DSL)%20is,as%20HTML%20for%20web%20pages) based on an ***event-driven simulation***, in which actions (“events”) trigger further events that happen at a later time, which in turn trigger more events, and so on.

All digital electronics consist of three logic gates: [inverter](https://en.wikipedia.org/wiki/Inverter_(logic_gate)), [and-gate](https://en.wikipedia.org/wiki/AND_gate), [or-gate](https://en.wikipedia.org/wiki/OR_gate). And our dsl has three similar [function blocks](https://github.com/Marbok/sicp/blob/main/src/sicp/chapter3/simulator_digit_circuits/logical_primitives.clj) too. Also we need to connect the elements, for this there is an object [wire](https://github.com/Marbok/sicp/blob/main/src/sicp/chapter3/simulator_digit_circuits/wire.clj). And the [agenda](https://github.com/Marbok/sicp/blob/main/src/sicp/chapter3/simulator_digit_circuits/agenda.clj) object is used to model delays. As a bonus, we get the ability combine elements and create more complex blocks, such as [half-adder](https://github.com/Marbok/sicp/blob/main/src/sicp/chapter3/simulator_digit_circuits/adders.clj).

### How it's work?
First, we need create [circuit](https://github.com/Marbok/sicp/blob/main/src/sicp/chapter3/simulator_digit_circuits/circuit.clj), set initial put signals and start simulation:
``` clojure
;; set delay for primitive elements
(set-inverter-delay! 2)
(set-and-gate-delay! 3)
(set-or-gate-delay! 5)

;; create wires
(def input1 (make-wire))
(def input2 (make-wire))
(def sum (make-wire))
(def carry (make-wire))

;; set testers
(probe 'sum sum)
(probe 'carry carry)

;; add complex element
(half-adder input1 input2 sum carry)

;; set signal in input1
(set-signal! input1 1)

;; start the simulation
(propagate)

;; set signal in input2 
(set-signal! input2 1)

;; continue the simulation
(propagate)

;; output:
;;
;; when create testers:
;; sum 0 New-value = 0
;; carry 0 New-value = 0
;;
;; first propagate:
;; sum 8 New-value = 1
;;
;; second propagate:
;; carry 11 New-value = 1
;; sum 16 New-value = 0
```
If we look inside, we will see how simple the structure is based on. 
Wire stores current signal value and list of actions, when the signal changes all actions are applied. 
Actions are used to transmit signals or print values like in probe. 
Logical primitives add new action in input wires. This actions have common structure - apply logical operation after delay. 
To model the delay, we use the agenda. Inside, it's just a time counter and a map with actions and the time when they should happen.
When we call propagate, the simulator gets the actions from the agenda according to the delay time and executes them.