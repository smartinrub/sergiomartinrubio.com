---
title: Quick Introduction to Reactive Programming
image: /assets/images/shubham-beeharry-223969.jpg
author: Sergio Martin Rubio
categories:
    - Design Pattern
    - Spring
    - Java Framework
mermaid: false
layout: post
---

## Introduction

**Reactive programming** is based on async and non blocking threads, so that, it allows us to use efficiently all the needed threads.

This paradigm makes use of the **Publisher-Subscriber** pattern to achieve asynchronous and event-based sequences of data.

## Operators

Operators are like the operations offered by the **Stream API** introduced in **Java 8**, this means that they do not get executed until a terminal operation is invoked, so that nothing happens until you subscribe. There are two operators:

- `Flux<T>`: for 0 to N elements
- `Mono<T>`: for 0 or 1 element.

These two operators allows us to use similar operations as the ones provided by the Stream API (`map`, `flatMap`, `filter`, `range…`), and eventually they will consumed when `subscribe()` is called.

## Other features

The [**Reactor API**](https://projectreactor.io/){:target="_blank"} also allows other kind of operations like schedules, publish on a particular thread the subsequence operations (`publishOn()`) or subscribe all the previous operations on a specific thread (`subscribeOn()`)

Moreover, **Reactor 3** is fully integrated with [**Spring 5**](https://spring.io/blog/2017/09/28/spring-framework-5-0-goes-ga){:target="_blank"}, and can by used through the [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html){:target="_blank"} framework, so you can create reactive routings. In order to fully make use of the reactive paradigm we will have to use reactive Spring Data and [reactor-netty](https://github.com/reactor/reactor-netty){:target="_blank"} which is built on Netty to provide reactive I/O.

## Testing

This library also provides some testing features. [StepVerifier](https://projectreactor.io/docs/test/release/api/reactor/test/StepVerifier.html){:target="_blank"} allows us to simulate a reactive chain of operations, and finally call `verify()` as the terminal operation, otherwise, `StepVerifier` will not subscribe to our sequence and nothing will be asserted.

We can also simulate delays by calling `withVirtualTime()`. This method takes a supplier as input, and lazily creates an instance of the tested `Flux`.

Another testing tool is [TestPublisher](https://projectreactor.io/docs/test/release/api/reactor/test/publisher/TestPublisher.html){:target="_blank"}, which allows us to trigger `onNext`, `onComplete` or `onError` events.

## Challenges

- Debugging: if an error is raised it will only show where the subscription happened, and we will not know where to start debugging. We might need to use debug sessions on our IDE, logs, execute operators on different threads, checkpoints…
- The complexity of our code might increase.
- More memory intensive.