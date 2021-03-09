---
title: Contexts and Dependency Injection in Java EE
image: https://lh3.googleusercontent.com/1-q8fr3LePunCNwVG6jOKluL9PWPnv7bW5Bj5d972xoOtCk-lBvRyizYa8CRaszmUhkBB_knqjb8nJNj_br3C_d8G1eR4TNJHI2DFfI3Wb8MLL9W1Y8DUx3oSDtGtMNSA7T64ocgt4M9rQtzufk_ltzo9wnnWExlQVkROdLUcy2elTFYJQWV28Jk7AT9KvEg1trfe6jNSVPQcQ3C_46z6RvnHxXPFmteKkuPybqFGWXvor4wyvaJmM9rKOn-ACHOlAxj-UMgbqavWf_51FNrjhdIZPxYJyJ5HUWBCn5CffvApIebMgOBy0G6JhQVc1Pm68kb7dXx0B_hqkZQdHfXeGPKhR007imcYreu_9Ig9KkQEvvhMEKCpllUpWqc1fF8ahLymsGqh7BEQMQfj-N0j2C2UWN4dNdZl6ClxjAx2pywpG6p1qoxYSwBzUoIuWRD6DR9Fzq0FlKtuIRRxxpG1SkCgFup1sjqPt_bdKOcL61VKw08WpuUnLuh93wzxz-OCCIE1n00dtUt1cZyaoyzbmM7iF4y8URBgAu8THf1KC4PBG-nmiHgZFI0epNfSVZkNRnntJGR7TIt1whSTdpviXtlmQ7MBciW-5xQJWR4fLthUeoqfI73E2U6MQB-REg5jr7633hdshAJsvOo6mVi_Bn1zX7jWXrMNzE1xB0A1SJQkroiNoXi0PriKojn=w640-h422-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Java Framework
mermaid: false
layout: post
---

## Introduction

**CDI** (*Contexts and Dependency Injection*) is one of the core features of *Java EE*. CDI allows you to glue the different componentes of your Java EE application in a loosely coupled way.

Commonly you have a web tier, enterprise tier and persistence tier and you can use CDI to join these three layers.

As the name says CDI provides a context feature and a dependency injection feature. The **context** is used to bind stateful componentes whereas the **dependency injection** allows you to inject a class instance without having to do the initialization by yourself.

## CDI Beans

You can create your CDI beans by adding one of the bean scope annotation to your Java class.

```java
@ApplicationScoped
public class MyBean {

    public String getMessage() {
        return "Hello Bean";
    }
}
```

The scope of a bean dictates when a bean instance is created and when is destroyed. Types:

- **Dependent** pseudo-scoped: This is the default scope and the lifecycle of the bean annotated with `@Dependent` is bound to the bean that injects the dependent bean.
- **Application**: A class annotated with `@ApplicationScoped` is created only once for the application.
- **Request**: A class annotated with `@RequestScoped` is created once in each request and shared throughout the request. 
- **Session**: Beans annotated with `@SessionScoped` are shared during the all the requests that belong to the same HTPP session.
- **Conversation**: A bean annotated with `@ConversationScoped` can track a conversation with a client. A conversation can include multiple linked HTTP requests where a *JSF* generates a URL containing a conversation ID.

## Dependency Injection

You can inject a bean with the `@Inject` annotation in three different ways:

- **Constructor**: You have to add the `@Inject` annotation to the class constructor

```java
@WebServlet(urlPatterns = "/bean")
public class MyController extends HttpServlet {

    private final MyBean myBean;

    @Inject
    public MyController(MyBean myBean) {
        this.myBean = myBean;
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        out.println("<h1>" + myBean.getMessage() + "</h1>");
    }
}
```

- **Setter method**: You can create a setter method annotated with `@Inject` so you can inject your bean.

```java
@WebServlet(urlPatterns = "/bean")
public class MyController extends HttpServlet {

    private MyBean myBean;

    @Inject
    public void setMyBean(MyBean myBean) {
        this.myBean = myBean;
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        out.println("<h1>" + myBean.getMessage() + "</h1>");
    }
}
```

- **Field declaration**: Add `@Inject` to a field.

```java
@WebServlet(urlPatterns = "/bean")
public class MyController extends HttpServlet {

    @Inject
    private MyBean myBean;

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        out.println("<h1>" + myBean.getMessage() + "</h1>");
    }
}
```

There are different advantages and disadvantages depending on the dependency injection strategy used.

| DI Strategy           | Advantages       | Disadvantages               |
| --------------------- | ---------------- | --------------------------- |
| **Constructor**       | Safer            | More boilerplate            |
| **Setter Method**     | More concise     | More boilerplate and unsafe |
| **Field Declaration** | Less boilerplate | Unsafe                      |

## Dependency Resolution

You can declare multiple implementations of the same bean however you need to somehow differentiate them. Java EE provides the `@Qualifier` annotation to **inject the desired bean implementation**.

```java
public interface Foo {
    String getName();
}
```

```java
@Qualifier
@Retention(RUNTIME)
@Target({METHOD, FIELD, PARAMETER, TYPE})
public @interface FirstFoo {
}
```

```java
@Qualifier
@Retention(RUNTIME)
@Target({METHOD, FIELD, PARAMETER, TYPE})
public @interface SecondFoo {
}
```

```java
@FirstFoo
@ApplicationScoped
public class FirstFooImpl implements Foo {

    @Override
    public String getName() {
        return "First Foo";
    }
}
```

```java
@SecondFoo
@ApplicationScoped
public class SecondFooImpl implements Foo {

    @Override
    public String getName() {
        return "Second Foo";
    }
}
```

```java
@WebServlet(urlPatterns = "/qualifier")
public class MyQualifierController extends HttpServlet {

    private final Foo secondFoo;

    @Inject
    public MyQualifierController(@SecondFoo Foo secondFoo) {
        this.firstFoo = firstFoo;
    }

	@Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        out.println("<h1>" + secondFoo.getName() + "</h1>");
    }
}
```

> If we replace `@FirstFoo` with `@SecondFoo` it will inject `FirstFooImpl` instead.

If not qualifier is used it will use either an implementation with `@Default` annotation or if there is a single implementation it will use that one.

You can also inject **multiple implementations** at once combining `@Inject` with `@Any` annotation.

```java
@WebServlet(urlPatterns = "/inject-list")
public class MyInjectListController extends HttpServlet {

    private final Instance<Foo> foos;

    @Inject
    public MyInjectListController(@Any Instance<Foo> foos) {
        this.foos = foos;
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("text/html");

        PrintWriter out = response.getWriter();
        foos.forEach(foo -> out.println("<h1>" + foo.getName() + "</h1>"));
    }

}
```

You can also use  `@Named`  to define a bean implementation and behaves in the same was as `@Qualifer` but additionally it allows you to give a name to the implementation.

## CDI vs EJB

Both CDI bean and [EJB](https://sergiomartinrubio.com/articles/ejb-what-it-is-why-it-exists-and-how-it-works) can be injected and the both are complementary. CDI bean is heavily focused in separation of concerns whereas EJB main purpose is to provide container services. CDI can be seen as a simplified version of EJB, since EJB provides all the dependency injection features and container features. As as rule of thumb you will usually create a CDI bean and include EJB features when is required. 

By default all session beans have `@Dependant`. You can also combine session beans with CDI scopes but there is some incompatibilities like a `@Stateless` EJB cannot have an `@ApplicationScoped` annotation or a `@Singleton` is incompatible with `@RequestScoped`.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/cdi-javaee-example.git" text="Examples" %}
</p>

Image by <a href="https://pixabay.com/users/alfcermed-3552488/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3539566">Alfonso Cerezo</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3539566">Pixabay</a>