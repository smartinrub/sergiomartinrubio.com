---
title: Micronaut, a New Contender for the Microservices Era
image: /assets/images/shubham-beeharry-223969.jpg
author: Sergio Martin Rubio
categories:
    - Java Framework
mermaid: false
layout: post
---

## Introduction

[Micronaut](https://micronaut.io/){:target="_blank"} is a web framework similar to **Spring Boot**, which started at [OCI](https://objectcomputing.com/){:target="_blank"} and was initially developed by _Groovy_ and _Rails_ developers from **Pivotal**.

_Micronaut_ is really a competitor for _Spring Boot_, and if you are comfortable with _Spring Boot_, _Micronaut_ provides a Spring-like way of creating web applications. It uses _JAX-RS_ annotations and specific _Micronaut_ annotations similar to the _Spring Boot_ ones (sometimes the only difference is the package name (`io.micronaut`)).

## Getting Started

A **CLI tool** similar to the one provided by _Spring_ is available to create your _Micronaut_ applications. The simplest way to start using the _CLI_ tool is to install it through _SDKMAM_.

```shell
curl -s https://get.sdkman.io | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install micronaut
mn --version
```

Now you can create your first _Micronaut_ project:

```shell
mn create-app com.sergiomartinrubio.micronaut-example --build maven
```

By default, it uses **Gradle** build system and _Java_ as the main language, however, you can tweak these setting by using these two flags: `--build` [gradle, maven] and `--lang` [java, groovy, kotlin]. You can also add features to the project, like **GraalVM** support, [**Zipkin**](http://sergiomartinrubio.com/articles/troubleshooting-tools-for-microservices-architecture) for tracing, type of rendering...

>**GraalVM**: it is an ecosystem which supports many languages and allows you to run your applications like a native one, so in the case of _Java_, it will translate your bytecode into machine code.

When `nm` command is run, it esentially creates a base folder with the build system file chosen (if you choose Gradle, it also provides a Gradle wrapper, so it downloads it for you), a **Dockerfile** ready to run, and a _logback_ file for log configuration.

## Features

You can crate controllers by using `@Controller` annotation and define your endpoints with http method annotations like `@Get`. When you run the application, you can hit the endpoint at `http://localhost:8080`, so it will feel as a **Spring Boot** application.

```java
@Controller
public class UpperCaseController {

    @Get("/touppercase/{input}")
    public String toUpperCase(@PathVariable("input") String input) {
        return input.toUpperCase();
    }
}
```

Tests are also created in a similar way as the **Spring Boot** way. Instead of 
`@SpringBootTest` annotation, it uses `@MicronautTest`, and `RxHttpClient` annotated with `@Client` instead of autowiring `TestRestTemplate`, to make _HTTP_ requests to your controller.

```java
@MicronautTest
class UpperCaseControllerTest {

    @Inject
    @Client("/")
    RxHttpClient client;

    @Test
    public void toUpperCase_givenStringPathParam_whenParamIsValid_thenReturnsStringInUpperCase() {
        HttpRequest request = HttpRequest.GET("/touppercase/test");
        String response = client.toBlocking().retrieve(request);

        assertNotNull(response);
        assertEquals("TEST", response);
    }
}
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/micronaut-example" text="Source Code" %}
</p>

_Micronaut_ provides a compile-time _AOP API_ that does not use reflection. For those who do not know what reflection is, it is basically a way to the program to introspect itself, so it asks itself if a class has specific methods, annotations, parameters... and _Spring_ uses intensively reflection to search for controllers, endpoints, beans... so there is a performance penalty that you pay, and most of this work is done during startup. Therefore, _Micronaut_ has a great advantage in the serverless model, where startup times are very important.

_Micronaut_ relies on annotations, it uses these annotations in a different way as _Spring_. During compile-time, it scans all the different annotations and add extra code to hook everything up. This means that once the _Java jar_ is compiled, annotations are not needed anymore, so there is no need to do any reflection, because the code is already baked into the `jar`. Bear in mind that _Micranaut_ doesn't modify the original classes, instead it creates extra classes, so if you need to debug your code, it will stop on the line you are expecting to stop.

{% include elements/figure.html image="https://lh3.googleusercontent.com/nGgRatrpzZdn-_ZvPwut4KXsFHZqmpOy-EGr7QuDAPuafOhqCQKE4Xon8XxvXV25g9F_BthT9XC6CNP3raJoSk6O10WoHLivEhiWnFRQax5ImF81t4RABMBsjxm6p45wBq8Ml_M4jQ=w400" caption="Micronaut Additional Classes" %}

_Micranaut_ tries to make it compatible with **GraalVM** as much as possible, and even has a feature to generate a project with Graal support. This is done by creating a _Docker_ image with _GraalVM_, _JDK_ and the _Java_ binaries.

**Micronaut only supports Netty** and **there is no way at the moment to switch to Tomcat**, so you have to considerer if _Spring_ metrics and threading handling of Tomcat is a must. However, it is compatible with **Micrometer**, **provides refresh and healthcheck endpoints, cache endpoints...**.

Other features:

- **Bean configuration and lifecycle modifications**.
- A number of different formats for your **application properties** like `application.yaml`, `application.properties` or `application.groovy`.
- _AOP_.
- **Environment specific configuration**.
- **AWS Lambda**.
- [OpenFaaS](https://github.com/openfaas/faas){:target="_blank"}, [ProjectFn](http://sergiomartinrubio.com/articles/serverless-hello-world-with-fn-framework){:target="_blank"} and [riff](https://github.com/projectriff/riff){:target="_blank"}.
- **Cloud Native support**: _Consul_, _Eureka_, _Kubernetes_, _Spring Cloud Config_...
- **Tracing** with **Zipkin**.
- **Ribbon**.
- _Kafka_ and _RabbitMQ_.
- **Support for data access configuration**: _mongoDB_, _Hibernate_, _Redis_, _Cassandra_...
- **Authentication**: _LDAP_...
- **Circuit Breaker** (_Hystrix_).
- **Websockets**.
...

## Mironaut vs Spring Boot performance

Three metrics will be measured to decide which framework performs better. Both, **Spring Boot** and **Micronaut** applications contain same features: http web support with only one class where a controller is defined, and a single endpoint, which converts a string to upper case. _Spring_ uses _Tomcat_ as the embedded web server, whereas the _Micronaut_ one uses _Netty_.

### Startup Time

```shell
docker run -p 8090:8090 spring-boot-example

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.4.RELEASE)

2019-04-30 11:18:47.729  INFO 1 --- [           main] c.s.s.SpringBootExampleApplication       : Starting SpringBootExampleApplication v0.0.1-SNAPSHOT on 960c5b1f8f80 with PID 1 (/spring-boot-example.jar started by root in /)
2019-04-30 11:18:47.733  INFO 1 --- [           main] c.s.s.SpringBootExampleApplication       : No active profile set, falling back to default profiles: default
2019-04-30 11:18:49.503  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8090 (http)
2019-04-30 11:18:49.546  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-04-30 11:18:49.546  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.17]
2019-04-30 11:18:49.634  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-04-30 11:18:49.635  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1828 ms
2019-04-30 11:18:49.974  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-04-30 11:18:50.247  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8090 (http) with context path ''
2019-04-30 11:18:50.252  INFO 1 --- [           main] c.s.s.SpringBootExampleApplication       : Started SpringBootExampleApplication in 3.001 seconds (JVM running for 4.299)
2019-04-30 11:21:49.215  INFO 1 --- [nio-8090-exec-7] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2019-04-30 11:21:49.216  INFO 1 --- [nio-8090-exec-7] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2019-04-30 11:21:49.247  INFO 1 --- [nio-8090-exec-7] o.s.web.servlet.DispatcherServlet        : Completed initialization in 30 ms
```

It takes **~3 seconds** to start the _Spring Boot_ application.


```shell
docker run -p 8080:8080 micronaut-example  
11:19:08.888 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 1352ms. Server Running: http://bca092ae4eb6:8080

```

In only **~1.3 seconds** the _Micranaut_ app is up and running.

_Spring Boot_ startup is more than 50% slower.

**Winner**: **Micronaut**

### Initial Memory Consumption


```shell
docker ps

CONTAINER ID        IMAGE                 COMMAND                  CREATED              STATUS              PORTS
                             NAMES
b883c35548e1        spring-boot-example   "/bin/sh -c 'java -X…"   About a minute ago   Up About a minute   8080/t
cp, 0.0.0.0:8090->8090/tcp   tender_booth
ba1be364bd13        micronaut-example     "/bin/sh -c 'java -X…"   About a minute ago   Up About a minute   0.0.0.
0:8080->8080/tcp             crazy_blackwell
```

```shell
docker stats

CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
ba1be364bd13        crazy_blackwell     0.11%               47.36MiB / 7.676GiB   0.60%               7.99kB / 168B       34.9MB / 4.1kB      29
b883c35548e1        tender_booth        0.27%               70.78MiB / 7.676GiB   0.90%               5.87kB / 0B         20.4MB / 4.1kB      45

```
where _b883c35548e1_ is the _Spring Boot_ app and _ba1be364bd13_ is _Micronaut_.

_Micronaut_ consumes around 30% less memory than _Spring Boot_.

**Winner: Micronaut**

### Heavy Load Memory Consumption

```shell
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
ba1be364bd13        crazy_blackwell     23.80%              339.1MiB / 7.676GiB   4.31%               3.24MB / 2.45MB     38MB / 4.1kB        52
b883c35548e1        tender_booth        22.52%              123.4MiB / 7.676GiB   1.57%               3.23MB / 2.35MB     21.9MB / 4.1kB      170

```

Surprisingly, under heavy load _Micronaut_ performs much worse in terms of memory consumption. _Micronaut_ uses 65% more memory than _Spring_ under the same conditions.

**Winner: Spring Boot**

### Response Time

```shell
Micronaut Request	9994	2	2	4	5	9	0	50	0.0	97.26710008953945	12.823299328210767	12.443349718486004
Spring Boot Request	9994	2	2	3	4	8	0	63	0.0	97.27467393420285	11.114391455372786	12.444318638066965
```

Both frameworks perform in the same way.

**Winner: tie**

### Conclusion

_Micronaut_ startup time and initial memory consumption are much better than _Spring Boot_ ones, however, the high memory consumption of _Micronaut_ during heavy load is quite concerning and it seems memory management is not that great.

### When should I use Micronaut?

**Micronaut** team advertise it as a low memory consumption framework with a quick startup time, so it is very suitable for the _serverless_ world.

This framework is also a good choice if you are moving your application to the cloud or you are creating a project from scratch.

### Caveats

- It is still fairly new, and the community is quite small compare to the _Spring_ one, though big enterprise companies like ThoughtWorks has already included it on their tech radar.
- _Netty_ is the only client web server available.
- For new developers used to _Spring_, it might be a little big challenging, because, under the hood it works in a completely different way.
