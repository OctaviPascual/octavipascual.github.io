---
layout: post
title: "Dividing turned harder than expected"
category: blog
---

I was doing some exercises of [The Haskell Book](http://haskellbook.com/), and I stumbled into one which looked easy at first sight but it turned out to be harder than I thought!

We are already given this code which divides two positive numbers, and the exercise consists on fixing this function in order to handle the case where the denominator is 0 and also to work with negative numbers. The goal is to have a function which works exactly as `divMod`, which is already implemented in Prelude.

```haskell
dividedBy :: Integral a => a -> a -> (a, a)
dividedBy num denom = go num denom 0
  where go n d count
          | n < d     = (count, n)
          | otherwise = go (n - d) d (count + 1)
```

After some time I had to look for help, as it was hard to get right all the cases with negative numbers. I found [this article](https://en.wikipedia.org/wiki/Division_algorithm) on division algorithms which helped quite a bit, even if it was not exactly the algorithm that I needed.

But after working a bit I finally managed to get very cool solution:

```haskell
dividedByUnsigned :: Integral a => a -> a -> (a, a)
dividedByUnsigned num denom = go num denom 0
  where go n d count
          | n < d     = (count, n)
          | otherwise = go (n - d) d (count + 1)

dividedBy :: Integral a => a -> a -> Maybe (a, a)
dividedBy _ 0       = Nothing
dividedBy n d
  | d < 0 =
    let Just (q, r) = dividedBy n (-d)
    in if r == 0 then Just (-q, 0) else Just (-q-1, r+d)
  | n < 0 =
    let Just (q, r) = dividedBy (-n) d
    in if r == 0 then Just (-q, 0) else Just (-q-1, d-r)
  | otherwise = Just $ dividedByUnsigned n d
```

As you can see, I renamed the function we were given to `dividedByUnsigned` and `dividedBy` is the function that works for any integer. As you can see, we always end up calling `dividedByUnsigned` to perform the actual division, but in `dividedBy` we play a bit with arguments. We even call the function recursively!

To better understand why and how this `dividedBy` function works, let's look at the initial solution I implemented:

```haskell
dividedBy :: Integral a => a -> a -> Maybe (a, a)
dividedBy _ 0       = Nothing
dividedBy n d
  | n < 0 && d < 0 =
    let (q, r) = dividedByUnsigned (-n) (-d)
    in Just (q, -r)
  | d < 0 = 
    let (q, r) = dividedByUnsigned n (-d)
    in if r == 0 then Just (-q, 0) else Just (-q-1, r+d)
  | n < 0 =
    let (q, r) = dividedByUnsigned (-n) d
    in if r == 0 then Just (-q, 0) else Just (-q-1, d-r)
  | otherwise = Just $ dividedByUnsigned n d
```

Here, there is no recursion and each case is separately treated. Note that here we have one more guard, which handles the case of two negative operands. However, this is not needed in the final solution showed before. How is that possible?

In the final solution, when both operands are negative, we enter the first guard and then, in the next call the second. The operations performed in each call are chained together and in the end we have (q, -r), which is the result that we expect!

Now we will see why this works, but before I encourage you to try with some examples to see what happens. For example, try with 10 -3, then with -10 3 and finally -10 -3.

For convenience, I post again the cool `dividedBy` function:

```haskell
dividedBy :: Integral a => a -> a -> Maybe (a, a)
dividedBy _ 0       = Nothing
dividedBy n d
  | d < 0 =
    let Just (q, r) = dividedBy n (-d)
    in if r == 0 then Just (-q, 0) else Just (-q-1, r+d)
  | n < 0 =
    let Just (q, r) = dividedBy (-n) d
    in if r == 0 then Just (-q, 0) else Just (-q-1, d-r)
  | otherwise = Just $ dividedByUnsigned n d
```

To show why this code works when the two numbers are negative, let's see what happens.

First, let x and y be two positive integers. We will label the three functions calls that will be performed. 1) is the initial call, which will then call 2) and finally 3) will be called. Note that in 3) both operands are positive integers, so it will return (q, r).

```
1) dividedBy -x -y (both operands are negative)
2) dividedBy -x y  (numerator is negative)
3) dividedBy x y   (both operands are positive)
```

The call 2) receives the (q, r) result returned by 3) and returns (-q-1, d-r). Since d is equal to y, we can write (-q-1, y-r).

The call 1) receives (-q-1, y-r) so it returns (-(q-1)-1, (y-r)+d). Since d is equal to -y, so we can write (-(q-1)-1, (y-r)-y) which is equal to (q, -r) and is the expected result.

If r is 0, the same reasonment can be applied and we get to (q, 0), which is also the expected result.

So, by properly handling what to do when numerator is negative and what to do when denominator is negative, we also handle the case when both are negative for free!

All in all, dividing turned harder than expected but it was interesting to see how something so trivial such as division would lead to all of this.
