---
title: The Big O Notation
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Algorithm
mermaid: false
layout: post
---

## Introduction

**O-notation** (pronounced _big-oh notation_) is a way of describing the performance of an algorithm and provides an upper bound or worst case on a function. Using O-notation you can describe the running time of an algorithm by inspecting its structure.

## Upper, Lower and Tight Bound

- **Lower bound**  ($$\Omega$$) functions are those that stay below $$T(n)$$ given a constant $$c$$, that is, $$T(n)\leq c n$$ for all $$n > 0$$. In other words, lower bounding is about finding a function that given a $$c$$ value always stays below $$T(n)$$.  e.g. given a quadratic function $$T(n)=n^2$$  a lower bound function would be $$f(n)=nlog(n)$$.

- **Upper bound** ($$O$$) functions are those that stay above $$T(n)$$ given a constant $$c$$, that is, $$T(n)\geq cn$$ for all $$n > 0$$. In other words, upper bounding is about finding a function that given a $$c$$ value always stays above $$T(n)$$. e.g. given a linear function $$T(n) = n$$  an upper bound function would be $$f(n)=n^2$$.

- **Tight bound or exact bound** ($$\Theta$$) is a combination of lower bound and upper bound. In other words, it's a function that bound $$T(n)$$ from the top and from the bottom. Tight bound functions are those that stay us much close as possible to $$T(n)$$ given a constant $$c$$, that is, $$T(n)\approx cn$$ for all $$n > 0$$. e.g. given a linear function $$T(n) = n$$ a tight bound function would be $$f(n) = n$$, since we were able to choose a value $$c$$ that satisfy the lower bound, $$f(n) = \frac12n$$ and a value $$c$$ that satisfy the upper bound, $$f(n) = 100n$$. As a result, we could find a value $$c$$ that can serve as an exact bound ($$\Theta(n)$$) for this function.

>IMPORTANT:  When we talk about time complexity we usually use $$O$$ as  $$\Theta$$

## Running Time Cases

Algorithms can be classified by three running time cases:

- **Worst case**: This is the worst runtime behavior. The execution time **upper bound** uses the notation $$O(f(n))$$.
- **Average case**: This is the expected behavior given random input data.
- **Best case**: This is how an algorithm performs in an ideal situation. This is also called as the **lower bound** and the notation used is this case is $$\Omega(f(n))$$

## Performance Categories

| O-Notation    | Name         | Example             |
| ------------- | ------------ | ------------------- |
| $$O(1)$$        | Constant     | Get in a hash map   |
| $$O(log(n))$$   | Logarithmic  | Binary Search       |
| $$O(n)$$        | Linear       | Linear Search       |
| $$O(n log(n))$$ | Linearithmic | Merge Sort          |
| $$O(n^2)$$      | Quadratic    | Bubble Sort         |
| $$O(2^n)$$      | Exponential  | Recursive Fibonacci |

