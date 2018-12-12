---
layout: post
title: "Concurrency is not Parallelism"
category: blog
---

I watched the talk [Concurrency is not Parallelism](https://www.youtube.com/watch?v=cN_DpYBzKso) by Rob Pike.

Many programming languages consider that the best way to represent the real world is through objects, hence they are Object Oriented Programming Languages. However, the world is also concurrent, that is, things happen simultaneously. For example, while I was writing this article you were doing another thing and while you are reading this article I will be doing something else.

## Go

Go is a language that supports concurrency. It provides:

* concurrent execution: **goroutines** are like cheap threads that under the hood are multiplexed onto OS threads
* synchronization and messasging: **channels** synchronize and exchange information between goroutines
* multi-way concurrent control: **select** is like a switch but the decision is based on the ability to communicate with a channel rather than on equal values

## Concurrency vs Parallelism

Concurrency is the composition of independently executing processes. It is related to how we structure lots of things at once.

Parallelism is the simultaneous execution of computations. It is related to how we can execute lot of things at once.

Once we have the concurrent design, parallelization can fall out and correctness is easy to achieve. Note that concurrency does not imply parallelism, we can have a concurrent program being executed in a single-core CPU.

## Conclusion

Concurrency and parallelism are two terms that sometimes are used interchangeably and this talk helps to highlight the differences between them. Concurrency can lead to parallelism but the latter can be achieved by other means. For example, bit-level parallelism is achieved through hardware and not through concurrency.
