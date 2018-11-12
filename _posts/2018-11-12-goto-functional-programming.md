---
layout: post
title: "GOTO 2018 - Functional Programming in 40 minutes"
category: blog
---

Today I watched the GOTO 2018 talk [Functional Programming in 40 minutes](https://www.youtube.com/watch?v=0if71HOyVjY) by Russ Olsen.

*Forget everything you know about programming*. This is a common phrase we see when we start learning functional programming. But in reality, we don't start from scratch. The ideas of programs, namespaces, arrays, strings, numerics, booleans or indentation are still valid. We can think of functional programming as a **refactor**.

## Functions

We are used to functions, methods, subroutines and methods. A mathematical function is a relationship between two sets: there is an input set and an output set and there is a mapping from one to the other. Furthermore, they are always the same and they don't change over the time, whereas in programming functions are just part of a code that will be executed by a computer.

We can try to set a set of rules in our functions to make them similar to mathematical ones: we call them **pure functions**. They can only look at the input and produce an output, that is, side effects are not allowed. That way, we hope our programs will be **easier to understand**.

## Immutable data structures

Immutable data structures make easier reasoning about your program. For example, let's consider the following program:

```
x = [1, 2, 3]
y = f(x) + g(f(x)) + 4*h(x) + 10*f(x)
x = ?
```

At this point, we know that *x* will still be [1, 2, 3] and we don't have to look at what the functions *f*, *g* or *h* do. Meanwhile, if data structures were mutable, we would have to check all those functions to make sure that none of them is somehow modifying *x*.

[Persistent data structures](https://en.wikipedia.org/wiki/Persistent_data_structure) are the way we deal with those immutable data structures in order to make them efficient.

## Side Effects

*Side effects are what programmers are paid to do*. Indeed, a customer does not care about the code, he cares about the effects. Customers do not care about the quality of the code, they care that the system is working properly.

We need a **bridge** from the functional world into the real world of side effects. In Clojure, an atom is a container for some mutable state. To update an atom, we must use a function that takes as input that state and returns another state which will override the state of the atom.

## Final thoughts

What I like about this talk is that the speaker insists in the fact that functional programming is not better or worse than imperative programming. For example, he explains that you can still make off-by-one errors, write redundant code or bad code in general. His point is to show the reasons why functional programming helps developers when programming.
