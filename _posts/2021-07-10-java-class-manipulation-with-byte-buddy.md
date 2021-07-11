---
title: Java Class Manipulation with Byte Buddy
image: https://lh3.googleusercontent.com/pw/AM-JKLUPY3ogCqAGtp-fxvhjXAiP5sZZ_snDV_SAsVLBe0sNQkvPDFUqdyZZ2OyGVKBYNb8ysg3SPI0vumW3vUl3A-L7DMvFY81LanCWKt46kmGRxFYIt2Oi-KCWwcMc-wvy6AirHtcgUsrjWH2dbfMpOfBk=w2760-h1640-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
mermaid: false
layout: post
---

[Byte Buddy](https://bytebuddy.net/#/){:target="_blank"} is a library to help you create and modify Java classes and provides a feature for generating [Java Agents](https://sergiomartinrubio.com/articles/getting-started-with-java-agent/).

This library is written in Java 5 but is compatible with any Java version. It's also very lightweight and only depends on [ASM](https://asm.ow2.io){:target="_blank"}.

Libraries like [Mockito](https://sergiomartinrubio.com/articles/get-started-with-mockito/) or Hibernate use Byte Buddy under the hood.



## Getting Started

### Installation

You can start using Byte Buddy in your own Java projects by including a single dependency:

```xml
<dependency>
  <groupId>net.bytebuddy</groupId>
  <artifactId>byte-buddy</artifactId>
  <version>1.11.6</version>
</dependency>
```

### How to Use

#### Method Overriding

You can use Byte Buddy by creating a instance of `ByteBuddy`. i.e.:

```java
@Test
void fixedValue() throws InstantiationException, IllegalAccessException, NoSuchMethodException, InvocationTargetException {
    Class<?> dynamicType = new ByteBuddy()
            .subclass(Object.class)
      			.name("com.sergiomartinrubio.bytebuddyexample.NewClass")
            .method(ElementMatchers.named("toString"))
            .intercept(FixedValue.value("Hello Fixed Value!"))
            .make()
            .load(getClass().getClassLoader())
            .getLoaded();

    assertEquals("Hello Fixed Value!", dynamicType.getDeclaredConstructor().newInstance().toString());
}
```

The previous code will do the following:

1. Define a class that inherits from `Object`.
2. Name the class with a fully qualified name `"com.sergiomartinrubio.bytebuddyexample.NewClass"`.
3. Override `toString` method from `Object`.
4. Replace the returned value of `toString` with `Hello Fixed Value!`. Did you noticed? This is similar to what *Mockito* does when you are mocking the return value of a particular method.
5. `make` triggers the generation of the class.
6. The class is loaded into the JVM with `load(getClass().getClassLoader())`.
7. `getLoaded` is used to return an instance of the new loaded class.

Instead of hardcoding the returned value in the class creation, we can use something called `MethodDelegation` which allows you to pass an interceptor class which replace the method implementation with the "best method definition match".

```java
@Test
void methodDelegation() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    Class<? extends Foo> dynamicType = new ByteBuddy()
            .subclass(Foo.class)
            .method(ElementMatchers.named("myFooMethod")
                    .and(ElementMatchers.isDeclaredBy(Foo.class))
                    .and(ElementMatchers.returns(String.class))
            )
            .intercept(MethodDelegation.to(FooInterceptor.class))
            .make()
            .load(getClass().getClassLoader())
            .getLoaded();

    assertEquals("Hello From Foo Interceptor!", dynamicType.getDeclaredConstructor().newInstance().myFooMethod());
}
```

given the parent class:

```java
public class Foo {
    public String myFooMethod() {
        return "Hello from my Foo method!";
    }
}
```

and interceptor class:

```java
public class FooInterceptor {
    public static String intercept() {
        return "Hello From Foo Interceptor!";
    }
}
```

In case you have multiple methods defined in the interceptor class with the same method arguments and return type we will get an exception like `java.lang.IllegalArgumentException: Cannot resolve ambiguous delegation...`. We need to set priorities in this case:

```java
public class FooInterceptor {
    @BindingPriority(1)
    public static String intercept() {
        return "Hello From Foo Interceptor!";
    }

    @BindingPriority(2)
    public static String secondInterceptor() {
        return "Hello from second Interceptor";
    }
}
```

The higher the number, the higher is the priority, therefore, the second method will be used.

#### Method Creation

We can also create methods from scratch as follows:

```java
@Test
void newMethod() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    Class<?> dynamicType = new ByteBuddy()
            .subclass(Object.class)
            .name("com.sergiomartinrubio.bytebuddyexample.NewClassWithNewMethod")
            .defineMethod("invokeMyMethod", String.class, Modifier.PUBLIC)
            .intercept(FixedValue.value("Hello from new method!"))
            .make()
            .load(getClass().getClassLoader())
            .getLoaded();

    Method method = dynamicType.getMethod("invokeMyMethod");

    assertEquals(
            "Hello From another interceptor!",
            method.invoke(dynamicType.getDeclaredConstructor().newInstance())
    );
}
```

1. Define a class that inherits from `Object`.
2. Give a name to the class.
3. Define public method  `invokeMyMethod` with a `String` return type.
4. Override return value for new method.

#### Field Overriding

You can also override the value of a constant (`static` and `final` fields) of an existing class:

```java
@Test
void overrideFieldValue() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException, NoSuchFieldException {
    Class<? extends Foo> dynamicType = new ByteBuddy()
            .redefine(Foo.class)
            .name("com.sergiomartinrubio.bytebuddyexample.NewFooClass")
            .field(ElementMatchers.named("myField"))
            .value("World")
            .make()
            .load(getClass().getClassLoader())
            .getLoaded();


    Field field = dynamicType.getDeclaredField("myField");
    assertEquals(String.class, field.getGenericType());
    assertEquals("World", field.get(dynamicType.getDeclaredConstructor().newInstance()));
}
```

1. Make a copy of an existing class with `redefine`.
2. Use a new name of the copy.
3. Select the field to override.
4. Change value.

#### Field Creation

Similarly we can define fields with values as follows:

```java
@Test
void newField() throws NoSuchFieldException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    Class<?> dynamicType = new ByteBuddy()
            .subclass(Object.class)
            .name("com.sergiomartinrubio.bytebuddyexample.NewClassWithNewField")
            .defineField("newField", String.class, Modifier.PUBLIC | Modifier.FINAL | Modifier.STATIC)
            .value("Hello From New Field!")
            .make()
            .load(getClass().getClassLoader())
            .getLoaded();

    Field field = dynamicType.getDeclaredField("newField");
    assertEquals(String.class, field.getGenericType());
    assertEquals("Hello From New Field!", field.get(dynamicType.getDeclaredConstructor().newInstance()));
}
```

We have to define the field as `STATIC` and `FINAL` to be able to set the value.

If you want to set the value of an instance field you can do it by defining a constructor:

```java
@Test
void newNonStaticField() throws NoSuchFieldException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    Class<?> dynamicType = new ByteBuddy()
            .subclass(Object.class)
            .name("com.sergiomartinrubio.bytebuddyexample.NewClassWithNewMethod")
            .defineConstructor(Modifier.PUBLIC)
            .withParameters(String.class)
            .intercept(MethodCall
                    .invoke(Object.class.getConstructor())
                    .andThen(FieldAccessor
                            .ofField("newField")
                            .setsArgumentAt(0)
                    )
            )
            .defineField("newField", String.class, Modifier.PUBLIC)
            .make()
            .load(getClass().getClassLoader())
            .getLoaded();

    Object newInstance = dynamicType.getConstructor(String.class)
            .newInstance("Hello From New non Static Field!");
    Field field = dynamicType.getDeclaredField("newField");
    field.setAccessible(true);
    assertEquals("Hello From New non Static Field!", field.get(newInstance));
}
```

## Java Agents

Byte Buddy also provides an API for creating Java agents with `new AgentBuilder()`. Therefore we can perform byte code manipulation at runtime.

This API has similar features to what *AOP* (Aspec Oriented Programming) libraries like [AspectJ](https://sergiomartinrubio.com/articles/start-using-aspect-oriented-programming-with-spring-aop/) provides. Some of the features provided by the Byte Buddy Java Agent API are:

- Intercept the method execution and perform additional logic.  You can use annotations like `@Advice.OnMethodEnter`, `@Advice.OnMethodExit` , `@Advice.Origin` or `@Advice.Enter`.
- Get method fields: `@Advice.AllArguments`
- Add fields and methods to classes: `@Advice.FieldValue`

In the following example we are going to show how to use the Agent builder with an agent that will intercept the method execution before and after the method is called.

The `Main` class contains the target method `invokeCustomMethod`.

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello from main method!");
        invokeCustomMethod();
    }

    public static void invokeCustomMethod() {
        System.out.println("Hello from custom method!");
    }
}
```

> Remember to create the `MANIFEST.MF` file under `resources/META-INF/` with something like:
>
> ```makefile
> Manifest-Version: 1.0
> Main-Class: com.sergiomartinrubio.bytebuddyclient.Main
> ```

In a separate project we can create our Java Agent with ByteBuddy. First we can define the "advice" class that will intercept the calls before and after a method is executed:

```java
public class HelloAdvice {
    @Advice.OnMethodEnter
    static long invokeBeforeEnterMethod(
            @Advice.Origin String method) {
        System.out.println("Method invoked before enter method by: " + method);
        return System.currentTimeMillis();
    }

    @Advice.OnMethodExit
    static void invokeAfterExitMethod(
            @Advice.Origin String method,
            @Advice.Enter long startTime
    ) {
        System.out.println("Method " + method + " took " + (System.currentTimeMillis() - startTime) + "ms");
    }
}
```

The method annotated with `@Advice.OnMethodEnter` will be executed before the intercepted method. `@Advice.Origin` gives you information about the intercepted method. On the other hand, `@Advice.OnMethodExit` will be executed after the intercepted method. `@Advice.Enter` contains the value returned by the method annotated with `@Advice.OnMethodEnter` so we can do things like the method execution time.

Finally we can define our "agent":

```java
class Agent {
    public static void premain(String arguments, Instrumentation instrumentation) {
        new AgentBuilder.Default()
                .type(ElementMatchers.any())
                .transform((builder, typeDescription, classLoader, module) -> builder
                        .method(ElementMatchers.nameContainsIgnoreCase("custom"))
                        .intercept(Advice.to(HelloAdvice.class)))
                .installOn(instrumentation);
    }
}
```

1. All classes are targeted: `.type(ElementMatchers.any())`.
2. Methods which name contain `custom` are targeted: `.method(ElementMatchers.nameContainsIgnoreCase("custom"))`.
3. Select advice class: `.intercept(Advice.to(HelloAdvice.class)))`.
4. Creates and install the agent builder into a given `Instrumentation`.

> As part of the Java Agent `.jar`  you need to specify the location of the `premain` method. You can use the maven plugin `maven-shade-plugin`.
>
> ```xml
> <build>
>   <plugins>
>     <plugin>
>       <groupId>org.apache.maven.plugins</groupId>
>       <artifactId>maven-shade-plugin</artifactId>
>       <version>3.1.0</version>
>       <executions>
>         <execution>
>           <phase>package</phase>
>           <goals>
>             <goal>shade</goal>
>           </goals>
>           <configuration>
>             <transformers>
>               <transformer
>                            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
>                 <manifestEntries>
>                   <Premain-Class>com.sergiomartinrubio.adviceagent.Agent</Premain-Class>
>                 </manifestEntries>
>               </transformer>
>             </transformers>
>           </configuration>
>         </execution>
>       </executions>
>     </plugin>
>   </plugins>
> </build>
> ```

Byte Buddy offers many other features that are not covered on this article, like intercepting methods marked by a particular annotation, more granular element matchers for classes and methods, constructor interceptors...

## Conclusion

You might never need to use this kind of libraries but in case you are planning to do some Java Class manipulation at runtime or create Java Agents, Byte Buddy is an excellent choice since it provides a very easy to use API.



Photo by [Ambitious Creative Co. - Rick Barrett](https://unsplash.com/@weareambitious?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/transform?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
