---
layout: post
title: "GOTO 2018 - Let's Get Lazy: Exploring the Real Power of Streams"
category: blog
---

Today I will talk about the GOTO 2018 talk [Let's Get Lazy: Exploring the Real Power of Streams](https://www.youtube.com/watch?v=bSbCJUSaSkY) by Venkat Subramaniam.

## Applicative Order vs Normal Order

[Applicative Order](https://en.wikipedia.org/wiki/Evaluation_strategy#Applicative_order) is an evaluation strategy that always evaluates all the arguments.

[Normal Order](https://en.wikipedia.org/wiki/Evaluation_strategy#Normal_order) is an evaluation strategy that applies functions before evaluating its arguments.

For example, the following Haskell code will work since `undefined` will never be evaluated by the `normalOrderIsFun` function as it doesn't need to use its argument.

```haskell
normalOrderIsFun :: a -> Int
normalOrderIsFun _  = 42

main :: IO()
main = do
  print $ normalOrderIsFun undefined
```

In contrast, a language such as Java, which follows applicative order, will always evaluate the arguments of a method before calling it. Imagine you have a very huge object that takes some time to be serialized into a string. You only want to log it in debug mode. With the following code, even if you are not in debug mode, the `toString()` method for `hugeObject` will be called since the argument of the `debug()` method must be **fully evaluated** before the call.

```java
logger.debug("Value of object " + hugeObject);
```

The workaround is to use [parameterized messages](https://www.slf4j.org/faq.html#logging_performance). That is, the `hugeObject` is passed as an additional argument and a pattern is used in the message to indicate where we want the object to be printed. Then, the `toString()` method will only be called if we are in debug mode.

```java
logger.debug("Value of object {}", hugeObject);
```

In conclusion, we can say that applicative order is an **eager** strategy which ensures that everything will be evaluated. Meanwhile, normal order is a **lazy** strategy that does not ensure that all arguments will be fully evaluated. Indeed, arguments are only evaluated when there are needed.

## Lazy Evaluation

> Efficiency is not about going faster, it's about avoiding things that shouldn't be done in the first place.

This sentence might look obvious, but how many times we do jump into the code and start optimizing it without thinking about the big picture. Are all the things that we are doing necessary? There is no reason to optimise something that is no needed! After all, **one of the easiest way to be inefficient is to do unnecessary work**.

> All problems in Computer Science can be solved using one more level of indirection

In other words: rather than solving this particular problem here, solve a more general problem that will solve, among others, that particular problem.

In functional programming, **the level of indirection comes from lambda expressions**. For example, you can postpone calling a function by wrapping it in a lambda expression. That way, you can lazily execute the call to the original function.

## Laziness in Collections

Imperative style has **built-in accidental complexity**. You must be aware of the state of each variable, counters, loop conditions, off-by-one errors... On the other hand, functional style has reduced complexity. Sometimes, the code reads like the problem state. In addition, laziness leads to infinite streams which generate values on demand.

Lazy evaluation makes the functional style efficient. Otherwise, a collection would be entirely evaluated after each stream operation and it wouldn't be efficient at all. Let's not even think about what would happen when fully evaluating an infinite stream... To achieve laziness in stream operations, **immutability** plays an essential role.

If you mutate your values, you cannot **postpone evaluation** as you don't know if the value will be changed later. Furthermore, it may not be obvious whether you want to evaluate the value before or after mutation, thus it adds some complexity. Meanwhile, immutability allows us to evaluate the values whenever we want, as we know they won't change. We want to evaluate as late as we can, so we can even **skip an evaluation** if we realise that we don't need it!

 By limiting ourselves on what we can do, we also make our lives easier. After all, **the more choices you have, the harder it is to choose**. Usually, we blame a language whose syntax allows doing the same thing in different ways. A similar thing happens when we work with immutability: there is only one way of working with our values, not many.
