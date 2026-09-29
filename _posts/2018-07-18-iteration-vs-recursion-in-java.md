---
title: Iteration vs. Recursion in Java
image: https://lh3.googleusercontent.com/fFwWC-rNv1YL5MIaFHFsvtCZiZr6e_EVJfUVIqvKzfT3g2xvJYimQNR0PqA_npJkTt17j2GLOIRKakG-aL9GLsXowIWNgkgfpp3WzSKM4hghbNeb0ONRm3TpKUoLYpDsdApVjyf6MDycAoUcEyBNFHuBqjGpNXWZtgqCjeBXIbZGnNYsWdGaMWPiZQfRS4Ec4VeczB9Wv4j655jxEYYHGyKMDfu-EWuoj-MXWmBwF9BjM1EnisqwVJ2vuIgksmC1bYS8QXivpBCir6uFmbEWVSkVhhQ4RocmSo4p0KdqqfEtTwfo2s2mxmhCOVVlGCjuWDLoou9MRN0n9KvFYoils1TnvgPl2ueGLnBqW5e4WtnNt1PGTGUOrqKJqZFjYGU6yS0pX-EeUhBi5dCdeD6YxsTewMtUDfYTRUahoTb2_diVPSwclovn1NAd8ds9eLCDghKppVeoibL3K1O2I-6tMDvJsXsj8BpH_4iUGeepDdTqB08bek4xnO0qchKrEseh5vHBdXi86J55bLl1ZKsKwAIIy3OAvSEduH74sf9VlDlK0a8mJxLrlXbFo1XV8YUhsGSLXtsgmzr2O6tO5C01Vto9lXaaM7HCQ4R0HRlJK6fxBsKAkQ6AHfPpJDb34xbQtmsUc83SFLsAGM6LCsbE7VzrP5uI1nZmxIx3eRb3HB8kyAgrSlnOp2PUOOa-=w1400-h400-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Algorithm
    - Java
mermaid: false
layout: post
---

Iteration and recursion are exchangeable in most cases. In theory, every program can be rewritten to avoid iteration using recursion. However, it is important to know that when using one or the other, this decision might have severe impacts on performance or potentially raise unexpected errors.

## Getting Started
### Mechanisms

- `for` loop: is the most traditional way to iterate an array or collection
- **Recursion**: a cleaner, simplified way to achieve the same as iterations
- **Tail recursion**: an optimized version of recursion
- `stream` library: the functional perspective to iterate collections

Calculating the factorial of a number is a useful problem here because it shows the performance differences between iteration and recursion. 

**JMH** harness will be used to conduct the test in a single thread with the following setup:

- Factorial of number: 20
- CPU: 1.4GHz quad-core Intel Core i5, Turbo Boost up to 3.9GHz
- OS: macOS Catalina

Both warm up and test iterations ran for 2 seconds, with two forks, three warmups, and three iterations. The tested code is:

* **`for` loop**:

```java
public long factorialForLoop(long number){
    long result = 1;
    for(; number > 0; number--){
        result *= number;
    }
    return result;
}
```

* **Recursion**:

```java
public long factorialRecursive(long number){
    return number == 1 ? 1 : number * factorialRecursive(number - 1);
}
```

* **Tail recursion**:

```java
public long factorialTailRecursive(long num){
    return factorial(1, num);
}

public long factorial(long accumulator, long val){
    return val == 1 ? accumulator : factorial(accumulator * val, val - 1);
}
```

* **"Stream" Library**:

```java
public long factorialStream(long number){
    return LongStream.rangeClosed(1, number)
            .reduce(1, (n1, n2) -> n1 * n2);
}
```

### JMH Benchmark

Running **JMH** is quite simple. You only need to add some annotations and a couple of dependencies to your `pom` file. Then, you can run the `.jar` file with _Maven_.

```shell
mvn clean install
java -jar target/benchmarks.jar
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-iterative-benchmark" text="Source Code" %}
</p>

Benchmark  | Score
------------- | -------------
for Loop  |  8.437 ns/op
Tail Recursion  | 15.500 ns/op
Recursion  | 16.887 ns/op
stream  | 76.064  ns/op


As you can see, the `for` loop is the winner as expected, because of the simplicity of the operations done during the iteration. However, this does not mean that it is always the best choice. Loops might be problematic when dealing with data structures shared by the caller of a method. It mutates the state of the object, so this is not a side-effect free option. Another negative factor of _loops_ is their readability. Therefore, in case we want to use immutable data objects and write a cleaner code, there are other options.

In terms of readability, the winner is the `Stream`. The functional style allows to write the iteration in a really simple manner and it is side-effect free. However, the performance is quite bad since it is about nine times slower than the `for` loop. The next candidate is _recursion_, which is also side-effect free. Before _Java 8_ was released, recursion had been used frequently over loops to improve readability, and problems such as _Fibonacci_, _factorial_ or _Ackermann_ are classic examples of the technique. The main problem of recursion is the risk of getting a `StackOverflowError`. The method pushes a new frame onto a thread's stack every time it calls itself, holding that call's intermediate operations, parameters and variables, and the frame cannot be discarded until the call returns. If the method keeps pushing frames for too long, the stack will exceed the limit and `StackOverflowError` will be thrown. _Tail recursion_ is the usual answer to this: by passing the intermediate result into the next call, the recursive call becomes the last thing the method does, so in principle the current frame could be reused instead of a new one being stacked. The catch is that the compiler has to actually perform that optimization, and the _JVM_ does not — so tail-recursive Java still stacks a frame per call and can still overflow. What we measured is only a slightly better constant factor, not a change in stack behaviour.

## Conclusion

If performance is the priority, traditional _loops_ are the way to go. In regards to readability and immutability, if these are top priority, _streams_  are the best option. Lastly, if you are looking for something in between, recursion offers a good performance and is side-effect free.
