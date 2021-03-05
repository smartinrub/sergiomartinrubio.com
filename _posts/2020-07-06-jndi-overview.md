---
title: JNDI Overview
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java Framework
mermaid: false
layout: post
---

## Introduction

The Java Naming and Directory Interface is a Java API that allows Java applications to look up resources via name. It is basically an interface to get instances of objects like data sources (e.g. `javax.sql.DataSource`), [JMS](https://sergiomartinrubio.com/articles/understanding-messaging-pattern-with-jms) resources (e.g. `javax.jms.ConnectionFactory`) or Java objects of any type.

**JNDI** is used to access remote resources without having to know the details of the configuration. For instance, you can use `org.apache.activemq.jndi.ActiveMQInitialContextFactory` resource to connect to the _ActiveMQ_ server without having to know the connection details, like url or port. Therefore, JNDI allows distributed applications to look up services in an abstract and resource-independent way, so you do not need to keep the configuration hardcoded in your application or in a configuration file.

## Architecture

Your application can use the JNDI API to perform operations on the naming service, then what happens inside it depends on what service provider (SPI) is plugged in. To declare one of the SPI you want to use in your application you have to specify a class that is part of the specific SPI.

JDK contains out-of-the-box four service providers: _LDAP_, _DNS_, _RMI_ and _CORBA_. However, you can create your own service provider for your custom services.

{% include elements/figure.html image="https://lh3.googleusercontent.com/_ImgQ-2Es2c7sEaK8x8-6pCFm9mWLmNMoIZvy4HKfypcPaOvqRLGlaVjpV1KrGyuuhYoVHmUoZHo3TTzHgPdPvDeAVlMPLsbV9sWGOTugCv5FpfFvM4sbQtS-yuh1AdQgLtqxGDd4A=w800" caption="JNDI Architecture" %}

## Components

### Context

The context is a set of bindings and they provide a lookup resolution similar to a _DNS_ server. You can create a context like this:

```java
Context jndi = new InitialContext();
```

You can also create and delete subcontexts (directories) with:

```java
Context jndi = new InitialContext();
jndi.createSubcontext("subcontext_folder");
jndi.destroySubcontext("subcontext_folder");
```

Also, you can retrieve the classes bound to a context with:

```java
Context jndi = new InitialContext();
NamingEnumeration<NameClassPair> contextData = jndi.list("contextName");
```

### Name

The `Name` interface represents a name which consist of a list of sorted strings name separated by a marker. Each component is separated by the forward slash character (`/`).

### Federation


{% include elements/figure.html image="https://lh3.googleusercontent.com/W9zPk0rpe_MHQz3MoW4t_COCzVWNOL3XoyWUTTHq5mSRCfyb7ZemxucsBpv6zUM0KdCAc_EKJVRO4qSBlQ4hxA5CJ3STkjofIaKYdbsbVXk5RPlvki31EBmubTXLHPPWOOIHWJZIIg=w600" caption="Federation" %}

A [Federation](https://docs.oracle.com/javase/jndi/tutorial/beyond/fed/index.html){:target="_blank"} aggregates naming systems, so you can compose names from different name systems. Each naming system can have its own naming convention (e.g. name system 1 uses `.` as the separator and is read right-to-left and name system 2 uses `/` as the separator and is read left-to-right).

### Composite Name

A [Composite Name](https://docs.oracle.com/javase/jndi/tutorial/beyond/names/composite.html){:target="_blank"} consist of multiple components that may belong to different namespaces. Each component is a string name from the namespace of a naming system. They are left-to-right and slash separated. A component can also be split into smaller parts by using `CompoundName` class (e.g. `fileSystemRoot.rightChildComponent.rootComponent` is right-to-left and the separator is ".").

```java
CompositeName name = new CompositeName();
CompositeName name = new CompositeName();
name.add("fileSystemRoot.rightChildComponent.rootComponent");
name.add("tutorial");
name.add("report.txt");
```

`name` will print out: `fileSystemRoot.rightChildComponent.rootComponent/tutorial/report.txt`

The function `lookup()` will look like:

```java
File file = (File) jndi.lookup("fileSystemRoot.rightChildComponent.rootComponent/tutorial/report.txt");
```

### Compound Name

A [Compound Name](https://docs.oracle.com/javase/7/docs/api/javax/naming/CompoundName.html){:target="_blank"} is a name within a single namespace that supports different naming conventions.

```java
Properties properties = new Properties();
properties.put("jndi.syntax.direction", "right_to_left");
properties.put("jndi.syntax.separator", ".");
CompoundName compoundName = new CompoundName("fileSystemRoot.rightChildComponent.rootComponent", properties);
Enumeration<String> components = compoundName.getAll();

while (components.hasMoreElements()) {
    System.out.println(components.nextElement());
}
```

this will print out: 

```
rootComponent
rightChildComponent
fileSystemRoot
```

>`CompoundName` properties can be found [here](https://docs.oracle.com/javase/7/docs/api/javax/naming/CompoundName.html){:target="_blank"}.

The service provider usually creates instances of the class. `CompoundName` should be created in a class that implements `NameParser` interface.

### Name Parser

A [NameParser](https://docs.oracle.com/javase/jndi/tutorial/provider/basics/parser.html){:target="_blank"} is used to parse names from a hierarchical namespace. You can create your own `NameParser` or use one from an existing `Context`.

- Create your own `NameParser`:

    1. Create a class that implements `NameParser` and override the `parse()` method that returns a `CompoundName` with properties.

        ```java
        public class MyNameParser implements NameParser {

            private static final Properties properties = new Properties();

            static {
                properties.put("jndi.syntax.direction", "left_to_right");
                properties.put("jndi.syntax.separator", ".");
                properties.put("jndi.syntax.ignorecase", "false");
                properties.put("jndi.syntax.escape", "\\");
            }

            @Override
            public Name parse(String name) throws NamingException {
                return new CompoundName(name, properties);
            }
        }        
        ```

    2. Create a class that implements `Context`, initialize the name parser and override the methods `getNameParser`:

        ```java
        public class MyContext implements Context {

            public static final NameParser myParser = new MyNameParser();

            ...

            @Override
            public NameParser getNameParser(Name name) throws NamingException {
                return myParser;
            }

            @Override
            public NameParser getNameParser(String name) throws NamingException {
                return myParser;
            }

            ...
                
        }
        ```

    3. Use your parser:

        ```java
        Context jndi = new MyContext();
        NameParser nameParser = jndi.getNameParser("");

        Name name = nameParser.parse("hello.world:.txt");

        Enumeration<String> components = name.getAll();
        while (components.hasMoreElements()) {
            System.out.println(components.nextElement());
        }         
        ```

        this will print out:

        ```
        world.txt
        hello              
        ```

- Use an existing `NameParser`:

    ```java
    Properties properties = new Properties();
    properties.setProperty(INITIAL_CONTEXT_FACTORY, "com.sun.jndi.fscontext.RefFSContextFactory");
    properties.setProperty(PROVIDER_URL, "file:/Users/sergio/workspace/jndi-example");
    Context jndi = new InitialContext(properties);

    NameParser nameParser = jndi.getNameParser("");
    Name name = nameParser.parse("tutorial/report.txt");

    Enumeration<String> components = name.getAll();
    while (components.hasMoreElements()) {
        System.out.println(components.nextElement());
    }
    ```

    >You can retrieve the File System Name parser by passing an empty string

    this will print out: 

    ```
    tutorial
    report.txt
    ```

### Reference

A **Reference** contains address information about the object being referenced. A `Reference` object consists of a list of addresses. Each address could be for example a network address, a location in memory or any other kind of information kept in the class. 

## Operations

To start using JNDI operations you need the Java EE library. The following Maven dependency will allow you to use JNDI operations:

```xml
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>8.0.1</version>
    <scope>provided</scope>
</dependency>
```

### Binding

The association of the name with the object is called _binding_. You can get a list of binding by calling `listBindings()`. Alternatively, you can use `list()` which is more lightweight. We can create a binding as follows:

1. Create JNDI context with environment properties. These environment properties can be passed to the `Context` constructor or create a `jndi.properties` file where the properties are set.
2. Define a class that implements `Referenceable` interface. This class has to implement this interface in order to be able to use the `getReference()` method to get its `Reference` that is used for binding. You can store the addresses when overriding the `getReference()`.
3. Call `bind()` method to bind a name to the object implementing `Referenceable`. You can also use `rebind()` if the name already exists and you want to override the object.

For this example bindings are saved in the file system, so the following dependency is required in order to be able to use `com.sun.jndi.fscontext.RefFSContextFactory`.

```xml
<dependency>
    <groupId>com.sun.messaging.mq</groupId>
    <artifactId>fscontext</artifactId>
    <version>4.4.2</version>
</dependency>
```

>If you run the following example it will generate a `.bindings` file under the given provider url. This file will contains the reference addresses.

```java
Properties properties = new Properties();
properties.setProperty(INITIAL_CONTEXT_FACTORY, "com.sun.jndi.fscontext.RefFSContextFactory");
properties.setProperty(PROVIDER_URL, "file:<path_in_your_system>");
Context jndi = new InitialContext(properties);

MyService myService = new MyService();

jndi.bind("my-service", myService);
```

alternatively, as mentioned before properties can be set in `resources/jndi.properties`:

```
java.naming.factory.initial = com.sun.jndi.fscontext.RefFSContextFactory
java.naming.provider.url = file:/Users/sergio/workspace/jndi-example
```

```java
@Data
public class MyService implements Referenceable {

    private String name;
    private String status;
    private boolean isActive;

    @Override
    public Reference getReference() {
        Reference reference = new Reference(MyService.class.getName());

        reference.add(new StringRefAddr("name", this.name));
        reference.add(new StringRefAddr("status", this.status));
        reference.add(new StringRefAddr("isActive", Boolean.toString(this.isActive)));

        return reference;
    }
}
```

Three addresses are store in `MyService` class (`name`, `status`, `isActive`) and each of them refers to a field of the class. In _JNDI_ reference addresses can be created by two different implementations: `StringRefAddr` or `BinaryRefAddr`.

>`@Data` is a convenient shortcut annotation from [Lombok](https://projectlombok.org/features/Data){:target="_blank"} to autogenerate constructors, getters, setters...

Moreover, you can unbind objects by calling `unbind()`. 

e.g.

```java
jndi.unbind("my-service");
```

### Lookup

{% include elements/figure.html image="https://lh3.googleusercontent.com/ONdAx2u6lQztsURwxkMcbMImteZQQk5p_4FtP5WxGGKuKycN1EDmJKeS1M2oSjc0DtOrlvlut9dKNH8kSQw9F0Re7ktp22CqJzlQPMCVQaTxJ0Zyfw6_PckhqbsiFf4fBQ89526sVA=w800" caption="Reference Handling in Lookup operations" %}

You can search for an object given a name with the `lookup()` method. This method will return a `Reference` object.

```java
Reference reference = (Reference) jndi.lookup("my-service")
```

Given the previous example, `lookup("my-service")` call returns:

```
Reference Class Name: com.sergiomartinrubio.jndiexample.MyService
Type: name
Content: null
Type: status
Content: null
Type: isActive
Content: false
```

As you can see the `lookup()` method returns a `Reference` class, which is not very useful. In order to retrieve `MyService` object you will have to create a class that implements `ObjectFactory` and override the method `getObjectInstance()` as follows:

```java
public class MyServiceFactory implements ObjectFactory {

    @Override
    public Object getObjectInstance(Object obj, Name name, Context nameCtx, Hashtable<?, ?> environment) throws Exception {
        if (!(obj instanceof Reference)) {
            return null;
        }

        Reference reference = (Reference) obj;
        if (!MyService.class.getName().equals(((Reference) obj).getClassName())) {
            return null;
        }

        MyService myService = new MyService();
        Enumeration<RefAddr> addresses = reference.getAll();

        while (addresses.hasMoreElements()) {
            RefAddr address = addresses.nextElement();
            switch (address.getType()) {
                case MyService.NAME:
                    myService.setName((String) address.getContent());
                    break;
                case MyService.STATUS:
                    myService.setStatus((String) address.getContent());
                    break;
                case MyService.IS_ACTIVE:
                    myService.setActive(Boolean.parseBoolean((String) address.getContent()));
                    break;
            }
        }

        return myService;
    }
}
```

then you have to update your class that implements `Referenceable` to add the factory to the `Reference` object:

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class MyService implements Referenceable {

    public static final String NAME = "name";
    public static final String STATUS = "status";
    public static final String IS_ACTIVE = "isActive";

    private String name;
    private String status;
    private boolean isActive;

    @Override
    public Reference getReference() {
        // factoryLocation is null because the factory class is in the project classpath
        Reference reference = new Reference(MyService.class.getName(), MyServiceFactory.class.getName(), null);

        reference.add(new StringRefAddr(NAME, this.name));
        reference.add(new StringRefAddr(STATUS, this.status));
        reference.add(new StringRefAddr(IS_ACTIVE, Boolean.toString(this.isActive)));

        return reference;
    }
}
```

now you can retrieve your object:

```java
MyService myServiceFromJNDI = (MyService) jndi.lookup("my-service");
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/jndi-example.git" text="Examples" %}
</p>
