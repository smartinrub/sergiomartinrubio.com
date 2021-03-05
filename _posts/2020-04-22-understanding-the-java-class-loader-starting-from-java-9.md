---
title: Understanding the Java Class Loader Starting from Java 9
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java
mermaid: false
layout: post
---

## Introduction

The **Java Class Loader** is a fundamental component of the **JVM**, so it is important to have a basic understanding of how it works and how you can create your own `ClassLoader`.

## What is the Java Class Loader?

The _Java_ compiler creates binary files with the `.class` extension for each source file. Each class file contains the definition and implementation code and are loaded into memory on demand. The **Java Class Loader** is responsible for loading the class files into memory.

## Class Loaders

Class loaders provided by the _Java_ run-time:

1. **Bootstrap Class Loader**: is built-in to the _Java Virtual Machine_ and represented by `null` in the _ClassLoader API_. Classes previously stored in `jre/lib/rt.jar` are now stored in a more efficient format in the `lib` directory.
2. **Platform Class Loader**: starting from _Java 9_ the extension class loader has been renamed to platform class loader. All classes in the _Java SE Platform_ are guaranteed to be visible through the platform class loader and can be used as the parent of a `ClassLoader` instance.
3. **System Class Loader** (aka _application class loader_): is usually the class loader used to start the application and used to define classes on the class path or module path, and loads a few JDK modules like `jdk.compiler`, `jdk.javadoc` or `jdk.jshell`.

>In JDK 8 and earlier _Bootstrap Class Loader_ loads classes in `jre/lib/rt.jar` and the extension mechanism allows runtime environment to find and load extension classes without specifically naming them on the class path and loads the code in the extensions directories (`<JAVA_HOME>/jre/lib/ext`, or any other directory specified by the `java.ext.dirs` system property).

The following code will print out the loaders of each module in the _JVM_.

```java
ModuleLayer layer = ModuleLayer.boot();
layer.modules().forEach(module -> {
    ClassLoader classLoader = module.getClassLoader();
    String classLoaderName = isNull(classLoader) ? "bootstrap" : classLoader.getName();
    System.out.println(classLoaderName + ": " + module.getName());
});
```

## Class Loading Steps

1. **Loading**: finds and imports the binary data for a type by its name and creating a class or interface from that binary representation.
2. **Linking**: performs verification, preparation and, optionally, resolution.
    - **Verification**: checks the correctness of the imported type.
    - **Preparation**: allocates memory for class variables and initialize the memory to default values.
    - **Resolution**: transforms symbolic references from the type into direct references.
3. **Initialization**: execute the code that initializes class variables to their starting values.

## Class Loading Mechanism

The **Java class loading mechanism** is based on **class-loading delegation**. Class loaders have a parent/child relationship and every class except for the bootstrap one has a parent class loader. Class loaders are created with one parent to whom they can delegate the loading. Steps:

1. Check if the class has been loaded previously by looking up in cache. If not, it will ask the parent to load the class. 
2. The previous step repeats recursively.
3. If the parent returns `null` or throws a `ClassNotFoundException`, then the class loader searches for the class on the class path.

>A class loader only see classes loaded by itself or its parent; it cannot see classes loaded by its children.

## Writing You Own Class Loader

You can simply create your own class loader by extending the `ClassLoader` class and override the method `findClass`.

When you create you own `ClassLoader` you have to:

1. Load the class as byte-code from your class path or any other place.
2. Call `defineClass` from the `ClassLoder` superclass convert of an array of bytes into an instance of a class.

```java
public class PathFileClassLoader extends ClassLoader {

    public static final String NAME = "com.sergiomartinrubio.javaclassloader.TestClassFromPath";

    @Override
    protected Class<?> findClass(String filePath) {
        // 1. Load bytes from file
        byte[] bytes = loadClassBytesFromFile(filePath);

        // 2. Create class from bytecode
        return defineClass(NAME, bytes, 0, bytes.length);
    }

    private byte[] loadClassBytesFromFile(String filePath) {
        File file = new File(filePath);
        try {
            return Files.readAllBytes(file.toPath());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return new byte[]{};
    }
}
```

then you can create your new class loader, create an instance of a class file and execute methods

```java
PathFileClassLoader pathFileClassLoader = new PathFileClassLoader();
// Load class from the root of the project
Class<?> classFromPath = pathFileClassLoader.findClass("TestClassFromPath.class");

Object classObject = classFromPath.getDeclaredConstructor().newInstance();

Method myMethod = classFromPath.getMethod("hello");
myMethod.invoke(classObject);
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-class-loader.git" text="Examples" %}
</p>
