---
title: Quick Introduction to ThreadLocal
image: https://lh3.googleusercontent.com/pw/ACtC-3cGsAf1VIlfMYA5VGHv6Sw_ApQdRJc-WzB6TR-Lgh10XbuiHfrAjAcUzahZp_uNC4Uyg0TtM-Jkp0H84RKsvAklajBp-5LYj0ecospNMSlaNRjYMHA0W3Cil0-LArwdLScKPyxXMLgNHAZSF_li8TSe=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
mermaid: false
layout: post
---

[ThreadLocal](https://docs.oracle.com/javase/7/docs/api/java/lang/ThreadLocal.html){:target="_blank"} variables are usually private static fields in classes and hold a separate value for each thread that uses them. There is only ever one `ThreadLocal` object; what is per-thread is the *value* it holds, so each thread that calls `get()` sees its own independently initialized copy. Therefore, information saved in a `ThreadLocal` can only be accessed by the thread that set it.

## Lifecycle

`ThreadLocal` variables are created when a thread accesses an instance, and all the copies of `ThreadLocal` variables are ready to be garbage collected when the thread goes away, unless there are other references to the `ThreadLocal` variables.

If a class is setting a `ThreadLocal` value in a static block, only the thread that initializes the class will be able to retrieve the value, since static blocks are only called during class initialization.

```java
public class MyThreadLocalManager {
    private static ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
    static {
        threadLocal.set(10);
    }
    public Integer getInteger() {
        return threadLocal.get();
    }
    public void setInteger(Integer integer) {
        threadLocal.set(integer);
    }
}
```

Therefore, if a thread accesses the instance of the previous class, which was originally initialized by another thread, and calls `getInteger()`, it will return `null`.

```java
MyThreadLocalManager myThreadLocalManager = new MyThreadLocalManager();
System.out.println(myThreadLocalManager.getInteger()); // prints 10
new Thread("thread-2") {
    @Override
    public void run() {
        System.out.println(myThreadLocalManager.getInteger()); // prints null
    }
}.start();
```

In order to initialize our `ThreadLocal` with a value, we can use the new **Java 8** withInitial()

```java
static {
    threadLocal = ThreadLocal.withInitial(() -> 10);
}
```

Now the output will be **“10”** for every thread that uses the `MyThreadLocalManager` instance.

## Operations

- `get()`
- `set(value)`
- `remove()`
- `withInitial(supplier)` -> static factory method, **since Java 8**

## Use Cases

- To avoid **synchronization**: `ThreadLocal` is thread safe, since only one thread can access the data saved in it.
- To create **global variables** shared by a single thread.
- In web applications in order to **keep a global state** (context, session…) **during an HTTP request**.  You might need some expensive data which needs to be computed multiple times for each HTTP request, and this data changes in each request, which means that we cannot use a plain cache. So `ThreadLocal` can be a solution to store this data, so we only have to calculate it once for each request.

## Caveats

`ThreadLocal` can cause memory leaks if it is not used safely. In web applications thread workers are provided by web servers like **Tomcat** or **Jetty**, and they maintain a thread pool to serve HTTP requests. Therefore, if a request creates a `ThreadLocal` and it is not removed before the response is returned, the value stays reachable from that worker thread's internal `ThreadLocalMap`, and since the lifespan of a pooled worker thread is longer than the request — and often longer than the web-app itself — it will prevent that value from being garbage collected. So always remember to remove `ThreadLocal` variables when they are not useful anymore.
