---
title: Java Optional Values
image: https://lh3.googleusercontent.com/brD_jXEcDUo-kClrcODUbg5ZvCYP4Vxrp0Sq9FvV2zEy9iqLdC_FIIPSKe3tWJVvDOJBxjg9uqgmL_cCvzKZ08Ku7Zup-p8z6XCTz6Qm1fXDVabke9_xIg00u51YKPAvg197kDgCcSGD5Wb5mV14KsUwSS-po7XA2kEahCywON0g9lHjrTo1FtN0QZQ9KtA4flxd6saziPWQXlJv6yHNJumesWvdl7aLe59ddU026rauBaEPqei8pOeiY9qcfP7tIhPE5y6JCGwJNg8oe8mjukhrfIJjBDVDGlWX_Tww4ANmaXKfTtzxlk62VtawYxMAjFbdqdZYentr0VTYa0fyT_AmQo1Z67mom3GB6Q_2Sk55W8bPPONhkKimy9cSazgIp1XzeO9LH5llshPyjisxQwFn9-3o-1GrlDqF3KRNpuPgsBdFuR61yAnHfMTY3i6WOIQbXhkhTs72AyNLuXOuFDPeD7pvpxhlKKinWU_S9NcQO1zUKoygzyPCDQn3Q2SUs_qjhKbZcgqAY3USbQaPejABHFR27CpXjuZ1k9lSlAemUEBRnEoq7LclrsVu--wJwaEZezd6E6WN8msHGoTeyBua7llOxRVxXanVTlmGpj_qbJ5nDcZpZ3akvaJbJXE_A1MC5k7eRbkg2V-YfQEGpvVBhC80tt7usdf1CaVyLCSTjYKnyQUIYfcan55i=w640-h341-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Java
mermaid: false
layout: post
---

## Introduction

Before _Java 8_ there were only two possible ways to gracefully exit a method when a value is not present, either you return `null` or throw an exception. This has changed since _Java 8_ with the `Optional` interface.

- `null`: efficient, but risky (we have to make sure that the client is going to handle a possible `null` return value).
- `Exception`: elegant, but expensive (the stack trace is captured).

> Java 8 includes the class `java.util.Optional` out-of-the-box.

## Optional Class

Since **Java 8** we have a third alternative, **Optionals**, which can contain an element or nothing. `Optional` allows us to return an empty result.

The optional library provides two ways to return an empty `Optional`: `Optional.Empty()` and `Optional.ofNullable(nullValue)`. But do not use `Optional.of(value)` if a null value is expected to be returned, because in that case it will throw a `NullPointerException`.

```java
public class MyClass {

	public static Optional<Integer> findMyLuckyNumber(List<Integer> list, Integer luckyNumber) {
		
		if (list.isEmpty()) {
			return Optional.empty();
		}
		
		Integer result = null;
		
		for (Integer value : list) {
			if (value == luckyNumber ) {
				result = value;
			}
		}
		
		return Optional.ofNullable(result);
	}
}
```

```java
private static final Logger LOGGER = Logger.getLogger(Main.class.getName());

public static void main(String[] args) {
	
	// returns Optional with value
	LOGGER.info(MyClass.findMyLuckyNumber(List.of(2, 5, 9), 9).toString());
	// returns empty Optional
	LOGGER.info(MyClass.findMyLuckyNumber(List.of(2, 5, 9), 1).toString());
	// returns empty Optional
	LOGGER.info(MyClass.findMyLuckyNumber(new ArrayList<>(), 1).toString());
}
```

**Should I always used Optional instead of returning null or throwing an exception?** In most of the cases it is recommended to return an `Optional`, because returning `null` or throwing an unchecked exception might be ignored by the client, and returning a checked exception forces the client side to write more code. Therefore, if an `Optional` is returned, the client can decide what to do.

A default behavior can also be specified when `null` is returned by appending `.orElse(“Empty…”)` or `orElseThrow(Exception::new)`. Another alternative is to use `.orElseGet(Supplier)`, which is more efficient than `.orElse()`, because the value is only retrieved when necessary. Remember that all these options return unwrapped values. Also you can use `ifPresent()` to run a block of code if the value is present.

**Java 9** added three new methods into the `Optional` class: `.or(Supplier<T>)`, `ifPresentOrElse(Consumer, Runnable)` and the stream method, which allows you to use _Optionals_ as streams. The main advantage of the first two methods is that now we can return an `Optional` as default value.

```java
Optional<Integer> defaultValue = Optional.of(99);
Optional<Integer> emptyValue = Optional.ofNullable(null);
// prints 99
System.out.println(emptyValue.or(() -> defaultValue).get());
```

You can also check if an `Optional` is empty by calling `isPresent()`, but since we have the methods mentioned above, you would rather use those for simplicity.

## Drawbacks

Returning an `Optional` is not always the best solution:

- For those cases when a list needs to be returned, it is better to just return an empty list. 
- Optionals require to be allocated and initialized. 
- For primitives values the `Optional` class cannot be use, instead you have to use `OptionalInt`, `OptionalLong` or `OptionalDouble`. 
- You should not use _Optionals_ as keys, values or elements in collections or arrays.

## Conclusion

- **Return _null_ or _Exception_ when performance is priority**.
- **Only use _Optional_ as a return value**.
- **_Optional_ fields are sometimes considered as _code smell_**, because maybe there should be a subclass containing the optional values.
