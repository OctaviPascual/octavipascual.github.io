---
layout: post
title: "Covariance and contravariance"
category: blog
---

I'm following a course named [Programming with Categories](http://brendanfong.com/programmingcats.html), and in a section about profunctors the concepts of covariance and contravariance were mentioned. This led me to [this video](https://www.youtube.com/watch?v=OJtGECfksds) which allowed me to better understand what those two words mean.

## Covariance

For functions, return types are covariant. That means that if we have a function `f :: a -> b` and a function `g :: b -> c`, we can "extend `f` to the right" and return a function `h :: a -> c`.

```haskell
covariance :: (b -> c) -> (a -> b) -> (a -> c)
covariance g f = g . f
```

## Contravariance

For functions, argument types are contravariant. That means that if we have a function `f :: b -> c` and a function `g :: a -> b`, we can "extend `f` to the left" and return a function `h :: a -> c`.

```haskell
contravariance :: (a -> b) -> (b -> c) -> (a -> c)
contravariance g f = f . g
```

## Example of covariance

We have a parser data type which allows us to parse strings into any type:

```haskell
data Parser a = Parser (String -> a)

parse :: Parser a -> String -> a
parse (Parser f) s = f s

charParser :: Parser Char
charParser = Parser (\s -> head s)

boolParser :: Parser Bool
boolParser = Parser (\s -> s == "True")
```

It can be used the following way:

```
λ> parse charParser "a"
'a'
λ> parse charParser "hello"
'h'
λ> parse boolParser "hello"
False
λ> parse boolParser "True"
True
```

As you see, the type constructor of `Parser` requires one type argument `a`, and this type argument is used as a function return type. Hence, we can make `Parser` an instance of `Covariant` typeclass:

```haskell
class Covariant f where
    myMap :: (a -> b) -> f a -> f b

instance Covariant Parser where
    myMap f (Parser g) = Parser (f . g)
```

Implementing `myMap` is pretty straightforward: we just apply `f :: a -> b` after applying `g :: String -> a`. In other words, we are extending `g` to the right, adding an extra transformation step on its return value. Hence, before being able to apply the provided function `f`, we need to apply `g`.

Now we can easily transform a parser from one type to another. For instance, we can use `fromEnum :: Char -> Int` to transform `Parser Char` into `Parser Int`. In fact, we have `myMap :: (Char -> Int) -> Parser Char -> Parser Int`.

```
λ> parse (myMap fromEnum charParser) "a"
97
λ> parse (myMap fromEnum charParser) "hello"
104
λ> parse (myMap fromEnum boolParser) "hello"
0
λ> parse (myMap fromEnum boolParser) "True"
1
```

## Example of contravariance

We have a formatter data type which allows us to format any type to a string:

```haskell
data Formatter a = Formatter (a -> String)

format :: Formatter a -> a -> String
format (Formatter f) a = f a

charFormatter :: Formatter Char
charFormatter = Formatter (\c -> [c])

boolFormatter :: Formatter Bool
boolFormatter = Formatter (\b -> if b then "T" else "F")
```

It can be used the following way:

```
λ> format charFormatter 'a'
"a"
λ> format boolFormatter False
"F"
λ> format boolFormatter True
"T"
```

As you see, the type constructor of `Formatter` requires one type argument `a`, and this type argument is used as a function argument type. Hence, we can make `Formatter` an instance of `Contravariant` typeclass:

```haskell
class Contravariant f where
    myMap :: (a -> b) -> f b -> f a

instance Contravariant Formatter where
    myMap f (Formatter g) = Formatter (g . f)
```

There is a subtle difference in `myMap` implementation with respect to the covariance example: we apply `g :: b -> String` after applying `f :: a -> b`. Thus, we compose the functions the other way around. We are forced to do it in this order, otherwise types wouldn't match. In other words, we are extending `g` to the left, adding an extra transformation step on its argument value. Hence, before being able to apply `g`, we need to apply the provided function `f`.

Now we can transform a formatter from one type to another. However, note how the signature of myMap looks like, it might be surprising: `myMap :: (Int -> Char) -> Formatter Char -> Formatter Int`. We provide a function from `Int -> Char` and we transform `Formatter Char` into `Formatter Int`.

How can this work? Well, this can be understood as the following: we have a `Formatter Char` which knows how to format characters. If we provide a `Int -> Char` function, then we can generate a formatter that knows how to format integers. To format an integer, the `Formatter Int` will transform the integer into a character using `f`, and then use `Formatter Char` to format the character. After all, it's not that surprising: imagine that we were given a function `Char -> Int` instead. What could we do? We would be unable to use the function, because `Formatter Char` holds a `Char -> String` function, which can't be composed with `Char -> Int`.

```
λ> format (myMap Data.Char.intToDigit charFormatter) 1
"1"
λ> format (myMap Data.Char.intToDigit charFormatter) 9
"9"
λ> format (myMap (\n -> n /= 0) boolFormatter) 0
"F"
λ> format (myMap (\n -> n /= 0) boolFormatter) 1
"T"
```
