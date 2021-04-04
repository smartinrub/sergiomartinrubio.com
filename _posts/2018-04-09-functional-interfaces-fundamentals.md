---
title: Functional Interfaces Fundamentals
image: https://lh3.googleusercontent.com/pw/ACtC-3eIXeU8xN1k2UWvQELWznFx7U9Jq4rllrxT73xi1vXjOWKGNGbJBrrekMzpCFoghMCNDR3JI54wFn9zzDbmx0nZD7xHl8df3i8VyahbY1dLKALrpmbCTKsQjQ0Ay8lNaZfxmOik_qCuGbDswTcmG7Sy=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
mermaid: false
layout: post
---

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

Image by <a href="https://pixabay.com/users/donterase-1070369/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=789628">donterase</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=789628">Pixabay</a>
