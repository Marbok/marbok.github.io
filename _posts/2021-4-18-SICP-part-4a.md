---
layout: post
title: SICP. Part 4. Lisp interpreter. v1.
categories: [SICP, Clojure]
---
This time the author gave us a very interesting task: implement Lisp interpreter. In the book, author develops [meta-circular evaluator](https://en.wikipedia.org/wiki/Meta-circular_evaluator). My implementation is a little different. It is based Clojure and evaluates Scheme dialect.

Lisp program is a syntax tree. A node consists of a function and its arguments, and arguments can also be nodes or primitive expressions (numbers and strings). If we want to evaluate node, we need evaluate argument-nodes, like in tree. 