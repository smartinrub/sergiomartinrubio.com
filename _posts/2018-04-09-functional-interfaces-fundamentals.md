---
title: Functional Interfaces Fundamentals
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java
mermaid: false
layout: post
---

## Introduction

**Functional interfaces** only have one abstract method different from the public methods of `Object.class` (`equals`, `hashCode`…), so that the contract contains a single method.

## Annotation

```java
@FunctionalInterface
interface Foo {
	void method();
}
```

`@FunctionalInterface` annotation is not mandatory but highly recommended, because the interface is checked at compiling time if in fact your interface is functional. Moreover, this annotation makes your architecture easier to understand.

## Use Cases

Since **Java 8** has been released functional interfaces are very useful when combining with lambda expressions.

```java
@FunctionalInterface
interface Foo {
	String method(String string);
}
```

And you can use it like this:

```java
Foo foo = string -> string + "world!";
System.out.println(foo.method("Hello "));
```

It is important to mention that **Java 8** already provides some functional interfaces out-of-the-box in `Function<T,R>` from the `java.util.function`. Therefore, in same cases, as the one explained previously, we can make use of them.

```java
Function<String, String> function = string -> string + "world!";
System.out.println(function.apply("Hello "));
```

## Default methods

**Default methods** are allowed in functional interfaces since they are not abstract methods.

```java
@FunctionalInterface
interface Foo {
	String method(String string);
		
	default String defaultMethod() {
		return "Hello World!";
	}
}
```

Although you are allowed to define default methods in functional interfaces, it is not a good practice to overuse this technique, because default methods might clash with each other when your interface extends several interfaces with default methods.
