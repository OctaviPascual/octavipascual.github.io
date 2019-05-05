---
layout: post
title: "Parametric Polymorphism"
category: blog
---


I was reading [The Haskell Book](http://haskellbook.com/), and the chapter 5 talks about types. More in detail, section 5.5 talks about polymorphism and I found useful to explain what it is using my own words.

First of all, let's review concepts related with type signatures:

- Concrete type: `Bool`, `Int` (capitalized in type signatures)
- Type variable: `a`, `b` (lowercase names in type signatures)
- Type class: `Num`, `Eq` (on the left of the `=>` sign in type signatures, constrains a type variable as an instance of a given type class)

Type signatures can have three kinds of types: concrete, constrained polymorphic (constrained type variable) or parametrically polymorphic (unconstrained type variable).

In Haskell, there are two categories of polymorphism: constrained polymorphism and parametric polymorphism. To better understand the differences between them, let's introduce some ideas:

- Fully polymorphic type variable: Type variable that is not constrained to any type class (so its final concrete type can be anything)
- Polymorphic function: Function whose signature has at least one type variable (`succ :: Enum a => a -> a`, `even :: Integral a => a -> Bool`)
- Parametrically polymorphic function: Function whose signature has at least one fully polymorphic type variable (`id :: a -> a`, `fst :: (a, b) -> a`)

Constrained polymorphism puts type class constraints on the type variables whereas in parametric polymorphism type variables are unconstrained. What does that mean?

In constrained polymorphism, constraints limit the set of potential types that a type variable can take. For that reason, we can make assumptions about said types. For example, if a type variable is constrained by the `Num` type class, we can compute its absolute value. That's what `abs :: Num a => a -> a` does.

Meanwhile, in parametric polymorphism we cannot make any assumption about the final type of a type variable. While those functions are really powerful since they will work for any type, they are also quite limited in what they can do. However, this is not necessarily bad. For example, if we look at this signature `id :: a -> a`, we already know what that function does as it can only do one thing: return the value it got!

All in all, both kinds of polymorphism are good and complement each other. In addition, a concrete type might be instance of multiple type classes so we can gain as many methods as we want.

Finally, now I hope that this sentence, which ends the section about polymorphism, makes sense:

> Parametricity is the property we get from having parametric polymorphism. Parametricity means that the behavior of a function with respect to the types of its (parametrically polymorphic) arguments is uniform. The behavior can not change just because it was applied to an argument of a different type.

Note that the last sentence is a really strong property. In other languages, how many times have you seen the following pattern: casting a variable and then having a switch statement for each possible class that said variable might belong to? Well, in Haskell you won't see it because you simply can't do it :)
