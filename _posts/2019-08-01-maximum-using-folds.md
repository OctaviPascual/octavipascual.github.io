---
layout: post
title: "Maximum using folds"
category: blog
---

I'm still working on [The Haskell Book](http://haskellbook.com/), and in the chapter devoted to folding lists I found an exercise worth to talk about. It consists in using folds to get the maximum and minimum of lists. If you are not familiar with fold, you should read [this](https://wiki.haskell.org/Fold).

## Maximum

The statement is simple: write a function `myMaximumBy` which takes a comparison function and a list and returns the greatest element of the list based on the last value that the comparison returned `GT` for. The signature of the function is: `myMaximumBy :: (a -> a -> Ordering) -> [a] -> a`. To better understand it let's see some examples:

```
λ> myMaximumBy (\_ _ -> GT) [1..10]
1
λ> myMaximumBy (\_ _ -> LT) [1..10]
10
λ> myMaximumBy compare [1..10]
10
```

In the first case, the first element is always greater than any other, so we return the first element. In the second, there is never an element that is greater than another, so we return the last element. Finally, if we use `compare`, the result is the maximum as expected.

Usually it is not a good idea to start thinking on how this problem would be solved in imperative programming, but in this case it might help. We start by selecting the first element as the maximum one, and then we traverse the rest of the list comparing the current element with the maximum so far. If the current element is greater than the maximum, we update the maximum to the current element:

```
def myMaximum(l):
    if len(l) == 0:
        raise Exception("cannot get maximum of empty list")
    maximum = l[0]
    for current in l[1:]:
        if current > maximum:
          maximum = current
    return maximum
```

Note that this algorithm is linear and that we only need to traverse the list once. Also, by traversing from left to right we ensure that we return the last value that the comparison returned `GT` for. In other words, in case the maximum value appears more than once, we return the one which is more to the right. Now, let's see if we can achieve the same performance using folds.

The first thing we must consider when using folds is whether to use `foldl` or `foldr`. This will determine in which way we traverse the list. Let's remember really fast how they work:

```
foldl :: (b -> a -> b) -> b -> [a] -> b
foldl f z [1, 2, 3] <=> ((z `f` 1) `f` 2) `f` 3

foldr :: (a -> b -> b) -> b -> [a] -> b
foldr f z [1, 2, 3] <=> 1 `f` (2 `f` (3 `f` z))
```

We see that if we want to mimic the left to right traversal, we need to use `foldl`. The initial value will be the head of the list and we will fold on the rest of the list. A possible implementation is the following:

```haskell
-- returns the greatest element of the list based on the last value
-- that the comparison returned GT for
myMaximumBy :: (a -> a -> Ordering) -> [a] -> a
myMaximumBy _ []     = error "cannot get maximum of empty list"
myMaximumBy f (x:xs) = foldl go x xs
  where go = \max a -> if f max a == GT then max else a
```

We can also add an additional example to see that indeed, when there is more than one maximum, we return the last one:

```
λ> myMaximumBy (\(a, b) (c, d) -> compare a c) [(4,1), (1,2), (4,5), (1,3)]
(4,5)
```

## Minimum

The statement is similar: write a function `myMinimumBy` which takes a comparison function and a list and returns the least element of the list based on the last value that the comparison returned `LT` for. The signature of the function is: `myMinimumBy :: (a -> a -> Ordering) -> [a] -> a`. To better understand it let's see some examples:

```
λ> myMinimumBy (\_ _ -> GT) [1..10]
10
λ> myMinimumBy (\_ _ -> LT) [1..10]
1
λ> myMinimumBy compare [1..10]
1
```

In the first case, there is never an element that is less than another, so we return the last element. In the second, the first element is always less than any other, so we return the first element. Finally, if we use `compare`, the result is the minimum as expected.

The solution is very similar to the `myMaximumBy`.

```haskell
-- returns the least element of the list based on the last value
-- that the comparison returned LT for
myMinimumBy :: (a -> a -> Ordering) -> [a] -> a
myMinimumBy _ []     = error "cannot get minimum of empty list"
myMinimumBy f (x:xs) = foldl go x xs
  where go = \min a -> if f min a == LT then min else a
```

Again, we can check that when there is more than one minimum, we return the last one:

```
λ> myMinimumBy (\(a, b) (c, d) -> compare a c) [(1,4), (2,2), (1,5), (3,3)]
(1,5)
```

We could end here, but we would be missing something. Indeed, both functions are very similar, so we can try to write one in function of the other. That is, use `myMaximumBy` in `myMinimumBy`. `flip :: (a -> b -> c) -> b -> a -> c` function does the trick. Using it, we can flip the comparison function. Here's how flip works:

```
λ> compare 2 1
GT
λ> flip compare 2 1
LT
λ> flip compare 2 2
EQ
```

Using that, we can call to `myMaximumBy` flipping the comparison function, so we will retrieve the minimum. The first idea that we might have is to do the following:

```
myMinimumBy :: (a -> a -> Ordering) -> [a] -> a
myMinimumBy f = myMaximumBy (flip f)
```

This works for some cases, but let's see what happens here:

```
λ> myMinimumBy (\_ _ -> GT) [1..10]
1
```

This returns the first element instead of the last one. The problem is that flipping the comparison function has no effect, so we are just executing `myMaximumBy` which has a different behavior than `myMinimumBy` for this specific case. One way of solving this issue is doing the following:

```haskell
myMinimumBy :: (a -> a -> Ordering) -> [a] -> a
myMinimumBy f = myMaximumBy (flip f) . reverse
```

Finally, don't try to be too smart and implement both `myMaximumBy` and `myMinimumBy` in terms of each other, because then you will infinitely recurse between both functions. You must implement properly at least one of them.
