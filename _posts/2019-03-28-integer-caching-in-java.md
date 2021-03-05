---
title: Integer Caching in Java
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java
mermaid: false
layout: post
---

## Introduction

**Java** provides some optimizations for integers, so instances of [Integer](https://docs.oracle.com/javase/7/docs/api/java/lang/Integer.html){:target="_blank"} are cached by the _JVM_ to increase performance.

## How it works

Basically, when an _Integer_ is initialized a cache is created if the number satisfies the range requirements. The default rage is set from -128 to 127, however, it can be tweaked by the `-XX:AutoBoxCacheMax=` option, so that, during VM startup, `java.lang.Integer.IntegerCache.high` will contain the new value and saved in the `sun.misc.VM` class.

## Example

As an example, every time we initialize an _Integer_ like `Integer num = 127;` the compiler will use the auto-boxing and convert this line to `Integer num = Integer.valueOf(127);`. This static method will cache the value if it is between -128 and 127.

Since _Java 9_, `Integer(int)` has been deprecated in favor of `valueOf(int)` in order to improve performance. The main difference is that the static method `valueOf(int)` will use cache, whereas the traditional `Integer` constructor will always get a new instance.

>`valueOf(int)`: Returns an `Integer` instance representing the specified int value. If a new `Integer` instance is not required, this method should generally be used in preference to the constructor `Integer(int)`, as this method is likely to yield significantly better space and time performance by caching frequently requested values. This method will always cache values in the range -128 to 127, inclusive, and may cache other values outside of this range.

_`valueOf(int)` **javadoc** in Integer class from java.lang library_

Therefore, next time you initialize an `Integer` with the same value, it will not create a new instance, instead it will return the one that has been already created. This can be simulated by running the following main method:

```java
public static void main(String[] args) {
    Integer i1 = 128, i2 = 128;
    Integer i3 = 127, i4 = 127;
    System.out.println(i1 == i2);
    System.out.println(i3 == i4);
}
```

output:

```shell
false
true
```

## Conclusion

Small integer values occur much more often than big values and therefore it makes sense to avoid the overhead of having different objects for every instance (an `Integer` object consumes 12 bytes of memory), so always avoid the use of the Integer constructor (`new Integer(int)`), which has been deprecated since **Java 9**.

Finally, this feature it not exclusive of `Integer`, other classes like `Byte`, `Short`, `Long`, `Character` are also using cache to improve performance.
