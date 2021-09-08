---
title: Getting Started with Spring Boot and Java 9 Modules
image: https://lh3.googleusercontent.com/pw/AM-JKLVlwAS-QsxgSZf9zsygWlY24S_rtKsray2c4Fm7bckRXKJQiNp4b_UW69WpT6JdDGlukq4j52vk2SyDIB2LfBFJEd3oCbiZUXoswK9XvbSKKaOGla0-yuyVRErVrz_LJb1gq4ZpVTe6lNjKt8Nsqoad=w640-h512-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
    - Java Framework
    - Spring
mermaid: false
layout: post
---

The **Java Module System** is a feature that was released with *Java 9* and was part of the project *Jigsaw*.

The reason for having the the Java Module System is to offer a greater level of granularity in terms of class visibility. As a result, this allows a better separation of concerns. Before Java 9 the only mechanism to control which classes can see which other classes was through Java packages, but this is not enough, specially when building large applications because sometimes you might want to make classes available only for specific purposes. 

For example, let's say you have a `PlayerRepository` class that should be only used by the service layer, but it shouldn't be used directly by the api layer. With Java Modules you can simply create separate modules for each layer or concern and define the boundaries of each class. The solution is to have a module for your *API* layer and other modules that will export the specific classes required by the *API* layer and nothing else. As you can see this increases greatly the level of encapsulation.

When designing modules you can choose the level of granularity, from having each package defined in a separate module to having all the packages in a single module.

## Getting Started

You can define a module by creating a `model-info.java` file with the following estructure:

```java
module com.sergiomartinrubio.persistence {
    exports com.sergiomartinrubio.persistence.entity;
}
```

This file will be place in the root of the module's source-code:

```
|-persistence
	|-src
		|-main
			|-java
				|-module-info.java
				|-com.sergiomartinrubio.persistence.entity
					|-Player.java
```

Then you compile the Java classes into byte-code class files with:

```bash
javac -d target/mods/com.sergiomartinrubio.persistence persistence/src/main/java/module-info.java persistence/src/main/java/com/sergiomartinrubio/persistence/entity/Player.java
```

where the first parameter points to the destination of the byte-code class files; the second parameter points to the `module-info.java` file; the third parameter points to the module we want to export.

We can run the module with:

```bash
java --module-path target/mods -m com.sergiomartinrubio.persistence/com.sergiomartinrubio.persistence.entity.Player
```

> To be able to run a module the exported class requires a main method.

You can also include modules inside other modules:

```
module com.sergiomartinrubio.model {
    requires com.sergiomartinrubio.persistence;

    exports com.sergiomartinrubio.model.service;
}
```

`com.sergiomartinrubio.persistence` is an external module that we want to use in the `model` module.

We can generate the byte-code for the `model` module as follows:

```bash
javac -d target/mods/com.sergiomartinrubio.model model/src/main/java/module-info.java model/src/main/java/com/sergiomartinrubio/model/service/PlayerService.java --module-path target/mods
```

in this case we have to specify `--module-path ` which contains the module required by the `player` module.

Then we can run it:

```bash
java --module-path target/mods -m com.sergiomartinrubio.model/com.sergiomartinrubio.model.service.PlayerService
```

## Module Directives

So far only `requires` and `exports` clauses were mentioned, however the Java Modules System provides other directives.

### requires

`requires` defines what modules are needed for a particular module. Required modules must be defined as `exports` in the external module in order to be able to use it.

### exports

`exports` makes packages public to other modules. By default, no package is exported.

### requires transitive

By using `requires transitive` you don't have to declare two modules, instead you simply declare the module that already declares the other required module.

### exports to

This is a directive to have an extra level of visibility control. It allows you to restrict which modules can use a particular module.

### open

This is to give a module reflective access to all its packages.

### opens to

This is similar to `open` but with an extra level of visibility control. Instead of opening an entire module to reflection, you can specify which packages within a module you want to open for reflection.

## Automatic Modules

You might want to use third party libraries within your *Java Modules* application and as you can imagine not all the libraries out there are migrated to *Java Modules System*. Fortunately, Java automatically convert *JARs* into automatic modules. A `module-info.java` is generated automatically for *JAR* files on the module path and all the packages are exported by default.

## Java Modules with Spring Boot

Java Modules can also be a nice addition to your Spring Boot applications.

To demonstrate how to use the Java Modules System we are going to build a [microservice](https://sergiomartinrubio.com/articles/spring-boot-microservices-cookbook/) that exposes an API and uses a database connection.

The project structure will be the following:

```
|-api
	|-src
		|-main
			|-java
				|-module-info.java
				|-com.sergiomartinrubio.api
					|-PlayerController.java
					|-TeamController.java
	|-pom.xml
|-application
	|-src
		|-main
			|-java
				|-module-info.java
				|-com.sergiomartinrubio.application
					|-Application.java
					|-SpringContextConfig.java
			|-resources
				|-application.properties
				|-data.sql
				|-schema.sql
	|-pom.xml
|-model
	|-src
		|-main
			|-java
				|-module-info.java
				|-com.sergiomartinrubio.model.service
					|-PlayerService.java
					|-TeamService.java
	|-pom.xml
|-persistence
	|-src
		|-main
			|-java
				|-module-info.java
				|-com.sergiomartinrubio.persistence.entity
					|-config
						|-PersistenceSpringContextConfig
					|-entity
						|-Player.java
						|-Team.java
					|-repository
						|-PlayerRepository.java
						|-TeamRepository.java
	|-pom.xml
|-pom.xml
```

As you can see the application is split by business layers with the help of Maven modules and Java Modules System.

The parent Maven module specifies the different Java Modules as well as common dependencies:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sergiomartinrubio</groupId>
    <artifactId>spring-boot-java-modules</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.4</version>
        <relativePath/>
    </parent>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <modules>
        <module>application</module>
        <module>api</module>
        <module>model</module>
        <module>persistence</module>
    </modules>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <skip>true</skip>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>

</project>
```

### application

This module will be responsible for the Spring context configuration and initialization. 

In order to make Spring work with the Java Module System we have to explicitly add some Spring packages to the `module-info.java`. `spring.core` requires opening the package for reflection.

```java
module com.sergiomartinrubio.application {
    requires spring.boot;
    requires spring.boot.autoconfigure;
    requires spring.context;

    // requires deep reflection
    opens com.sergiomartinrubio.application to spring.core;
}
```

The **Spring Boot version 2.5.4** **does not seem to be compatible with the *Java Modules system***. `@SpringBootApplication` should be able to find beans declare in other modules other than the one that initializes the context, however this is not happening. The solution is to create a configuration that explicitly points to the location of the beans to be loaded into the Spring context:

```java
@Configuration
@ComponentScan(basePackages = {"com.sergiomartinrubio.api", "com.sergiomartinrubio.model", "com.sergiomartinrubio.persistence"})
public class SpringContextConfig {
}
```

> **Spring Boot version 2.5.4** is not compatible with the Java Modules System.

Additionally, application properties also must be defined inside this module:

```java
spring.datasource.url=jdbc:postgresql://localhost:5432/nba
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.username=user
spring.datasource.password=password
spring.datasource.initialization-mode=always
spring.jpa.hibernate.ddl-auto=none
spring.jpa.open-in-view=false
```

### api

This module provides the *API* layer where we define endpoints.

The `module-info.java` requires some internal modules like `com.sergiomartinrubio.model` and the `spring.web` module, since we are using some *Spring* annotations like `@RestController` or `@GetMapping`.

```java
module com.sergiomartinrubio.api {
    requires com.sergiomartinrubio.model;
    requires com.sergiomartinrubio.persistence;

    requires spring.web;

    exports com.sergiomartinrubio.api;
}
```

### model

This module defines the business logic of the application and should only require the persistence layer module and the *Spring* context module for marking the classes as beans. Here we can set the boundaries of the usage of this module to the api module. By doing this we avoid using this module in the wrong layer (e.g. this module should not use on the `persistence` module).

```java
module com.sergiomartinrubio.model {
    requires com.sergiomartinrubio.persistence;

    requires spring.context;

    exports com.sergiomartinrubio.model.service to com.sergiomartinrubio.api;
}
```

### persistence

The persistence layer is define on this module. Here we create the repository interfaces and entities.

This module requires packages for configuring *JPA*, marking *POJOs* as entities and serialization.

```java
module com.sergiomartinrubio.persistence {
    requires spring.context;
    requires spring.boot.autoconfigure;
    requires spring.data.jpa;
    requires java.persistence;
    requires com.fasterxml.jackson.databind;

    exports com.sergiomartinrubio.persistence.repository;
    exports com.sergiomartinrubio.persistence.entity;
}
```

Again, Spring has problems to automatically scan classes. This time *Spring* is not loading classes marked as entities and configure JPA repositories, so we have to explicitly point to them.

```java
@Configuration
@EntityScan("com.sergiomartinrubio.persistence.entity")
@EnableJpaRepositories(basePackages = "com.sergiomartinrubio.persistence.repository")
public class PersistenceSpringContextConfig {
}
```

## Conclusion

The **Java Modules System** is a nice addition to your Java or Spring projects if you want to have an **extra level of granularity**. However, the **lack of compatibility with the Spring framework** makes it a little bit awkward to use, and having to add additional configuration for loading beans into the Spring context is not ideal.

For small microservices the Java Module System might be overkilling and it would be only recommended for medium to large applications.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/spring-boot-java-modules.git" text="Source Code" %}
</p>

Photo by [Derek Sutton](https://unsplash.com/@dereksutton?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/modules?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

