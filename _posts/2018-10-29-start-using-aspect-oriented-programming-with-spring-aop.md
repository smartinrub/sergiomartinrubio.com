---
title: Start using Aspect Oriented Programming with Spring AOP
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Design Pattern
    - Spring
    - Java Framework
mermaid: false
layout: post
---

## Introduction

**Aspect Oriented Programming (AOP)** helps us to solve specific situations in a very elegant way and is used to insert code depending on how our code looks.

Why _aspects_? When we have a concern which is cross domain, _aspects_ are a good solution. e.g. logs, _exceptions_, security…

[AspectJ](https://www.eclipse.org/aspectj/){:target="_blank"} is the _AOP Java framework_ commonly used when we want to use aspects on our application.

How does _AspectJ_ work? The AOP framework does not look at you actual code, it looks at the byte code, so we give patterns to search for, and _AspectJ_ inserts the code where the match is found.

There are other _AOP_ implementations such as [Spring AOP](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html){:target="_blank"} or [JBoss AOP](http://jbossaop.jboss.org/){:target="_blank"} that are built on top of _AspectJ_ and simplify its use.

## Spring AOP

### Concepts

- **Aspect**: in **Spring AOP**, _aspects_ are declared using classes annotated with `@Aspect`.
- **Join Points**: are the possible points in the program execution where our code can be inserted. e.g. method call, _exception_ thrown…
- **Advice**: is the action taken by an aspect on a particular join point.
- **Pointcut**: is where you cut the flow of the program to add our advice (some code). We use patterns to specify where to put the code. e.g. the name of a method.
- **Weaving**: process of finding a match and inserting the code that you want to execute.

### How to create Aspects

To create an aspect you need to have a class annotated with `@Aspect` and register the aspect as a regular _bean_ in your **Spring XML** configuration or add the `@Component` annotation. Once the class is created with the annotations required, you can declare as many _Advice_ as you want inside.

```java
@Aspect
@Component
public class MyBeforeAspect {
    
    @Before("org.smartinrub.aopspringdemo.aspect.JoinPoints.serviceLayer()")
    public void beforeSomething(JoinPoint joinPoint) {
        log.info("ASPECT - Before: {}", joinPoint.getSignature());
    }
}
```

As a good practice, we will also create another class where all _join points_ will be declared, so we can refer the _join points_ from an _advice_ annotation.

```java
public class JoinPoints { 
    
    @Pointcut("execution(* org.smartinrub.aopspringdemo.service.*.*(..))")
    public void serviceLayer(){}

    @Pointcut(
            value = "execution(* org.smartinrub.aopspringdemo.repository.*.*(..)) && args(id)",
            argNames = "id")
    public void repositoryLayer(int id){}

    @Pointcut("execution(* org.smartinrub.aopspringdemo.controller.*.*(..))")
    public void controllerLayer(){}
}
```

### Advice Types

- `@Before`: it runs before a _join point_.
- `@After`: it runs after a method call
- `@AfterReturning`: it runs when a _join point_ completes successfully and after `@After`.
- `@AfterThrowing`: it runs when a method exits because of an _exception_.
- `@Around`: it runs before and after the method execution. It allows us to proceed to the _join point_, return its own object or throw an _exception_.

#### @Before

```java
@Before("org.smartinrub.aopspringdemo.aspect.JoinPoints.serviceLayer()")
public void beforeSomething(JoinPoint joinPoint) {
    log.info("ASPECT - Before: {}", joinPoint.getSignature());
}
```

In this example, apart from the `@Before` annotation which will make run the _advice_ body before the method execution, we can notice that `JoinPoint` variable was added to the method signature. `JoinPoint` allows us to get some information from the target method.

#### @After

```java
@After(value = "org.smartinrub.aopspringdemo.aspect.JoinPoints.repositoryLayer(id)")
public void afterSomething(JoinPoint joinPoint, int id) {
    log.info("ASPECT - After: {}", joinPoint.getSignature());
}
```

In this example `@After` will make run the _advice_ after the method execution. A parameter name was also added to this example to show how we can make arguments available to the _advice_ body. Therefore, when the method that satisfies the _pointcut_ pattern runs, the _advice_ will be able to use the value.

#### @AfterReturning

```java
@AfterReturning(
        value = "org.smartinrub.aopspringdemo.aspect.JoinPoints.repositoryLayer(id)",
        returning = "result")
public void afterReturning(JoinPoint joinPoint, Object result, int id) {
    log.info("ASPECT - After Returning: repository {} with id {} returned {}", joinPoint, id, result);
}
```

`@AfterReturning` allows us to run some code after the method returns a value. If you need access to the returned value, we can use the _returning_ attribute on the `@AfterReturning` annotation and add the same value name on the _advice_ signature to make it available in our _advice_ body.

#### @AfterThrowing

```java
@AfterThrowing(
        value = "org.smartinrub.aopspringdemo.aspect.JoinPoints.controllerLayer()",
        throwing = "ex")
public void afterThrowing(JoinPoint joinPoint, IllegalArgumentException ex) {
    log.info("ASPECT - After Throwing: exception \"{}\" on controller \"{}\"", ex, joinPoint.getSignature());
}
```

This example shows how to run some code when an _exception_ is thrown, and expose the exception in your _advice_ body. We have to do the same as we did for `@AfterReturing`, but in this case the annotation attribute is _throwing_.

#### @Around

```java
@Around("@annotation(org.smartinrub.aopspringdemo.annotation.TrackTime)")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    long startTime = System.currentTimeMillis();
    log.info("ASPECT - Around: Time before proceed: {}", startTime);
    
    Object proceed = joinPoint.proceed();
    
    long totalTime = System.currentTimeMillis() - startTime;
    log.info("ASPECT - Around: Time after proceed {}: {}", joinPoint.getSignature(), totalTime);
    return proceed;
}
```

This is the most complex _advice_. `@Around` includes _before_, _after_, _throwing_ and _returning_ functionalities.

The first parameter of this _advice_ must be `ProceedingJoinPoint`, and needs to be called with `proceed()` to execute the method. The value returned by the _advice_ will be the same as the value of the target method.

In this examples we also make use of an annotation as a _join point_ pattern, so this advice will be only executed when the annotation `TrackTime` is found.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
    public @interface TrackTime {   
}
```

#### @TrackTime

```java
@Override
public String getSomething(int id) throws InterruptedException {
    Thread.sleep(2000);
    return repositoryOne.getSomething(id);
}
```

### Advantages over AspectJ

- It is simpler to use than **AspectJ**, since we do not need the [AspectJ compiler](https://www.eclipse.org/aspectj/doc/next/devguide/ajc-ref.html){:target="_blank"} or **LTW** ([load-time weaving](https://www.eclipse.org/aspectj/doc/released/devguide/ltw.html){:target="_blank"}).
- It uses **proxy pattern** and **decorator pattern**.

### Disadvantages over AspectJ

- **Spring AOP** is proxy based, so you can only use method-execution _join points_.
- There is some runtime overhead.

## Conclusion

**Aspect-oriented programming** is intended to address common problems that **object-oriented programming** doesn’t address well and can avoid code duplication in some situations. However, _AOP_ is an extreme solution that can hide parts of your code and could make debugging a difficult task, so it can cause more harm than good.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/aop-spring-demo" text="Source Code" %}
</p>