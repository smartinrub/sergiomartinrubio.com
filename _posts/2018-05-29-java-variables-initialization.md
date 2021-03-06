---
title: Java Variables Initialization
image: https://lh3.googleusercontent.com/I4PsbWqvL4ibwrOgetxKfJ5JVkKQjVIeMpzrlQ08Ng88LL9EACrLC4MhsnbHZhALRvmyzySCFJyNaKU3yjTL64ZduAlcQFOHk7-kKJ5ZF_3nf3BOBMVy7CVaOXuJj9D7XlATPp9H0Gx5gaAdxV8sZ-8fxV3EgG2uXnmAORgPNmqd7yPxtiKRFYNC6x1-CgSxCUl0HICt0DcoqWheaT5dYYOqLAWMbc6tXs-ivvK1EG3VmIMOIxIeaB0jOMyKI7buaNu9Hi_vzAgYDajbkVhTbxGkAoPk5WbBMIpWJDOY6zYBr_HtpzAfkrL9w5QHhtgoaScIAVxRapcfoACXp4xunp_teqyvZ4w_Riiwo_n-RS6vkOPMaotPfFrwIjh9zYgCv8JdqlKIQ_mZ_YiFEdBzrKO6bI0d26Tlj6zBizNG6yRjZGhYPqrctNiEyB1mk-E4U8Y45e-6gdI2X4MhLiatUtxflvCx5b5QtClKHWIdzJbtkKoRFk0C67kETKfk2J923pV_Mf7fcqg__SwomTXB8RhYgopaPBenjalnx5lRXv1Bt7rk0kwrbangOdkhAM3BbONO8k-jNJG9si0QKEwdjzeDQpyGgWwSQ38Q_XWnNld3RytyA8qsAs0Ed8RFvcpGwCxJRSW3GFKFe3Irx24PHrv85-pNbGTZ19ZgAKrh9PhlzNgyO0UbQed833wt=w1920-h1280-no?authuser=1
categories:
    - Java
mermaid: false
layout: post
---

## Introduction

There are two ways of initialize a field, eagerly and lazily. Lazy initialization is based on initializing the field when the value is needed, whereas the eager initialization is based on initializing the variable when it is declared.

Both choices are compatible with _static_ and _instance fields_, however the implementations may differ when trying to achieve performance improvement.

## Eager Initialization

Eager initialization is commonly used when:

- **Performance is not a priority**.
- **It is not expensive to initialize**.
- **High number of accessors inside a class**.
- **Many threads**.

```java
private final Object eagerObject = new Object();
```

## Lazy Initialization

Lazy initialization is recommended when:

- **You want to improve performance**
- **You want to reduce access from instances of a class**
- **It is costly to initialize**
- There are not many threads** (when there are two o more threads, _synchronization_ might be required)

    e.g. 

    ```java
    private Object lazyObject;

    private synchronized Object getLazyObject() {

        if (lazyObject == null)

            lazyObject = new Object();

        return lazyObject;
    }
    ```

## Use Cases

As you can see it is really important to know the context of the field and how it is going to be accessed, to decide what kind of initialization is recommended. However, bear in mind that most of the time eager initialization is recommended.

When **lazy initialization** and **static** fields are combined, the lazy initialization holder class idiom is recommended.

```java
private static class OjbectHolder {

    static final Object object = new Object();
}

private static Object getField() { return OjbectHolder.object; }
```

By doing this, the static object is initialized only the first time `getField()` is called. In addition, the `getField()` method is not synchronized and it is only a field access, therefore there is no cost of access.

In case you want to use lazy initialization for an instance field use the [Double Check Locking Pattern](https://sergiomartinrubio.com/articles/creational-patterns#double-checked-locking-pattern).

```java
private volatile Object lazyIntanceObject;

private Object getLazyIntanceObject() {

    Object result = lazyIntanceObject;

    if (result == null) {  // 1st check

        synchronized(this) {

            if (lazyIntanceObject == null)  // 2nd check (locking thread)
                lazyIntanceObject = result = new Object();
        }
    }
    return result;
}
```

- A `volatile` field is created to hold the object. The variable is stored in the main memory, so reading or writing to the _volatile_ variable will be on the main memory and not only on the _CPU_ cache. This ensures that the field will never be half initialized.
- The new variable is created to ensure that the field is read only.
- The first check is to improve performance by verifying if the field is already initialized before locking and allowing that multiple threads can access to the field.
- The synchronized block ensures that only one thread can go inside the block, it checks if the field is already initialized (it might happen if there is a [race condition](http://tutorials.jenkov.com/java-concurrency/race-conditions-and-critical-sections.html){:target="_blank"}), otherwise, it initializes the object.

## Conclusion

You should choose eager initialization over lazy initialization in most cases. Only use lazy initialization when the performance is a priority or in order to break _initialization circularity_.
