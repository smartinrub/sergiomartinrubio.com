---
title: Spring Boot Microservices Cookbook
image: https://lh3.googleusercontent.com/pw/AM-JKLUZi72lxo7gTLlAfKz49r8whpiPsJG2Xt0UF48KtP0ZsSvbAkM5hJTgPwmBTh74S9ZMV_GM3RNByG3U8Lrb6uLnwOKDAISEo_BKNYX-FdhhU8_DY6od59Q5aTZuuyu1WvzTXlzt2UaPUwqC7kOsyY2B=w640-h427-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
    - Java Framework
    - Spring
mermaid: false
layout: post
---

Nowadays Microservices architecture is the facto choice for building medium or large web applications, and this tendency is very align with Agile methodologies since having services that are loosely coupled give engineers more freedom to make faster modifications to them and as a result you can get quick feedback.

I've been developing microservices with Java and Spring for quite a while and I'd like to condense on an article the fundamental ingredients for creating a Java microservice from scratch.

## Before You Start

When creating a microservice there are a few **things to need to take into account**:

1. **Identify the business domain**. <u>It's very important to understand the business domain so the microservice has the right level of responsibility</u>. 
   - A **too** **course-grained** microservice will make the service difficult to maintain and will add constraints when multiple engineers are working on a particular feature, mainly because code changes might overlap.
   - A **too** **fine-grained** microservice will increase the overall complexity of the system and a service can become a simple abstraction layer with no logic.
2. **Decide API communication style**: some common *API* architecture styles are:
   - **REST** (Representational state transfer): is probably the most common communication style nowadays for microservices communication and the reason of this can be because of low complexity and the formats allowed (*XML*, *JSON*, plain text...). 
   - **RPC **(Remote Procedure Call): this is a protocol with a strict specification and at the same time is high performance. It's a good candidate for microservices but take into account that is difficult to debug, since it required special tools for using the protocol unlike *REST*, which is a simple *HTTP* request that you can even make from your browser.
   - [GraphQL](https://sergiomartinrubio.com/articles/moving-beyond-rest-with-graphql/): this can be seen as an evolution of *REST*, and it consist of a single *HTTP* endpoint that is used for executing operations, and solves the issue of  “overfetching” or “underfetching”. *GraphQL* might be a good candidate for edge services, since it provides a simple way to consume an API and it provides documentation capabilities. On the other hand, *GraphQL* might not be the best for the communication between microservices and the reason is that there is an extra overhead, caching is difficult to implement or adds more complexity.
   - **SOAP** (Simple Object Access Protocol): this is the least used protocol for new applications. It's a protocol itself unlike *REST*, which is using *HTTP* protocol, and provides a standardized communication. However, it only supports *XML* format and this makes the communication quite verbose, is more complex and heavy weight. Because of all of these disadvantages this protocol is not the best candidate for microservices communication.
3. **Configuration management**: this is about where you are planning to keep the configuration of the microservices. You might want to keep the configuration inside a file in your application, inject it as part of a deployment job with Jenkins, centralize the configuration with something like [Spring Cloud Config Server](https://sergiomartinrubio.com/articles/centralized-configuration-with-spring-cloud-config-server/)...
4. **Asynchronous communication**: distributed systems use asynchronous communication quite heavily and it's important to understand when **[eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency){:target="_blank"}** can be applied.
5. **Security**: how are you planning to integrate authentication and authorization? What credential management technology are you going to use?
6. **Logging**: having logs in your microservices is essential and you might want to aggregate those logs and put them in a queryable database. You can use tools like [Micrometer](https://micrometer.io){:target="_blank"} for exposing the logs, [Graphite](https://graphiteapp.org){:target="_blank"} or [Prometheus](https://prometheus.io/docs/guides/query-log/)_blank"} for exposing the logs, [Graphite](https://graphiteapp.org){:target="_blank"} for pulling the logs out of the microservice and [Grafana](https://grafana.com){:target="_blank"} for visualization.

## Building Microservices with Spring Boot

I'm choosing **Spring Boot** for building microservices because it's a mature Java-based framework, but there are other alternatives out there like [Micronaut](https://sergiomartinrubio.com/articles/micronaut-a-new-contender-for-the-microservices-era/) or [Quarkus](https://quarkus.io){:target="_blank"}.

### Skeleton

You can create the skeleton of your microservice at the [Spring Initilializr site](https://start.spring.io){:target="_blank"}. There you can select the dependencies you think you will need (don't worry too much about this since you can add more later), use a proper artifact name and group name. For the group name it's recommended using a reversed domain *name* you control (e.g. `com.sergiomartinrubio`) and for the name I usually use a kebab-case pattern (e.g. `authentication-service`). Also you need to select the desired Java version and project management tool (I usually use Maven, but Gradle is also perfectly fine).

I recommend using a **separate Git projects for each microservice instead of using a mono repo**. Using a separate Git projects makes easier using different branches at the same time across multiple services, better mapping with CI/CD tools or more clear Git history.

### Testing

There are many testing libraries for Java and I would like to highlight some of them:

- [JUnit 5](https://sergiomartinrubio.com/articles/take-unit-testing-to-the-next-level-with-junit-5/): it's a very mature unit testing Java library.
- [Mockito](https://sergiomartinrubio.com/articles/get-started-with-mockito/): excellent library for mocking the behavior of your classes.
- [AssertJ](https://joel-costigliola.github.io/assertj/){:target="_blank"}: provides fluent assertions. Use instead of [Hamcrest](http://hamcrest.org){:target="_blank"}.
- [Testcontainers](https://www.testcontainers.org){:target="_blank"}: it's very useful for integration tests when you want to simulate external services like databases.
- [Wiremock](http://wiremock.org){:target="_blank"}: for mocking external APIs.
- [Cucumber](https://sergiomartinrubio.com/articles/cucumber-a-bdd-framework-for-java-and-spring/): if you want to use BDD (Behavior Driven Design) this is an excellent tool.

> [TDD (Test Driven Development)](https://sergiomartinrubio.com/articles/test-driven-development-by-example/) is quite advised and that's why this sections is added to the top!

### Project Structure

I usually follow a layer structure because it helps me to write code by layers with TDD, but a business structure is also a good option as mentioned at the [Spring Boot documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.structuring-your-code){:target="_blank"}.

```
com
 +- sergiomartinrubio
     +- account-service
         +- Application.java
         |
         +- model
         |   +- Account.java
         |
 				 +- controller
 				 |   +- AccountController.java
 				 |
 				 +- service
 				 |   +- AccountService.java
 				 |
 				 +- repository
 				 |   +- AccountRepository.java
 				 |
 				 +- config
 				 |   +- SwaggerConfig.java
 				 |
 				 +- exception
         |   +- AccountException.java
         |   +- UnexpectedException.java
         |
         +- util
         |   +- UsernameUtils.java
```

A business structure would look like:

```
com
 +- sergiomartinrubio
     +- account-service
         +- Application.java
         |
         +- account
         |   +- Account.java
         |   +- AccountController.java
         |   +- AccountService.java
         |   +- AccountRepository.java
```

### Checkstyle

Check-style tools that run as part of the build are something I recommend to use as soon as possible so the code is properly formatted and everyone follows the same conventions.

A good one is `maven-checkstyle-plugin`:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>${maven-checkstyle-plugin.version}</version>
    <dependencies>
        <dependency>
            <groupId>com.puppycrawl.tools</groupId>
            <artifactId>checkstyle</artifactId>
            <version>${checkstyle.version}</version>
        </dependency>
    </dependencies>
    <executions>
        <execution>
            <id>checkstyle-validation</id>
            <phase>validate</phase>
            <inherited>true</inherited>
            <configuration>
                <configLocation>checkstyle/checkstyle.xml</configLocation>
                <suppressionsLocation>checkstyle/suppressions.xml</suppressionsLocation>
                <includeTestSourceDirectory>true</includeTestSourceDirectory>
            </configuration>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

The check-style rules are configurable and you can add suppressions.

### Logging

A logging library for customizing the logs is also very important. 

For example you can use Logstash:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>6.6</version>
</dependency>
```

And then configure it as desired:

`resources/logback-spring.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <springProfile name="!dev">
        <include resource="org/springframework/boot/logging/logback/defaults.xml" />
        <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}/}spring.log}"/>
        <include resource="org/springframework/boot/logging/logback/file-appender.xml" />

        <appender name="jsonConsoleAppender" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
        </appender>

        <root level="INFO">
            <appender-ref ref="jsonConsoleAppender"/>
        </root>
    </springProfile>

    <springProfile name="dev">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{ISO8601} %p %t %c{0}.%M - %m%n</pattern>
                <charset>utf8</charset>
            </encoder>
        </appender>

        <logger name="org.springframework.web" additivity="false" level="INFO">
            <appender-ref ref="CONSOLE" />
        </logger>

        <root level="INFO">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>

</configuration>

```

### Database

A rule of thumb when building a microservice is to **use a dedicated database for each microservice**. You are not likely to benefit from a *Microservices* architecture if all the services share the same database. Services that share the same database are tightly coupled and if a database table changes all the services will have to change.

For creating the database schemas I highly recommend using [Liquibase](https://sergiomartinrubio.com/articles/getting-started-with-liquibase-and-spring-boot/) because it provides a nice out-of-the-box versioning feature and abstracts the creation of raw SQL queries.

```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
    <version>4.3.1</version>
</dependency>
```

Alternatively you can create SQL files inside your project but at some point it might become messy and difficult to track changes.

### Monitoring

Metrics are important to understand how a microservice is behaving and tools like Micrometer can help.

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
    <version>${micrometer.version}</version>
</dependency>
```

Also you might want to add a heath endpoint to check the status of the service. Spring Actuator provides this out-of-the-box:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### API Documentation

For documenting your API you can either manually add a API usage section to the readme file or use tools like [Swagger](https://swagger.io){:target="_blank"} or [Spring REST Docs](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/){:target="_blank"}.

As a first step I recommend using Swagger since it's quite easy to integrate.

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

### Tracing

[Tracing](https://sergiomartinrubio.com/articles/troubleshooting-tools-for-microservices-architecture/) is related to logging, but it's more specific ant it's focused on the API requests.

[Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth)_blank"} or [Spring REST Docs](https://docs.spring.io/spring-restdocs/docs/current/reference/html5/){:target="_blank"} is a library that will automatically add **trace ID** to your *REST API* and will propagate those trace IDs to downstream services by including the IDs in the header of the *HTTP* requests.

```xml
<dependency>
  	<groupId>org.springframework.cloud</groupId>
  	<artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

### API Names

If you are using REST endpoint names matters. Make sure you establish standards for endpoints and they should clearly define the resource the endpoints are providing. You might have parent-child relationships between resources within your microservices and that's totally fine, however if URLs tend to be excessively long and nested, your microservice may be doing too much.

For versioning you can prefix your endpoints with `/v{number}` (`v1`), however this is not required in most of the cases for internal API since you know who the clients of a microservice are.

### Configuration

Separating the configuration from the code base is important and you should aim to have immutable application. This means that when the application is deployed only the injected configuration can change.

Configuration can be injected at startup time of the application through either environment variables or through a centralized repository the application reads on startup. 

There are multiple solutions for a configuration management system like [Etcd](https://etcd.io){:target="_blank"}, [ZooKeeper](https://zookeeper.apache.org){:target="_blank"} or [Spring Cloud Config](https://sergiomartinrubio.com/articles/centralized-configuration-with-spring-cloud-config-server/). If you are using Spring the integration with Spring Cloud Config is very straightforward.

Photo by [Markus Spiske](https://unsplash.com/@markusspiske?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/born?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
