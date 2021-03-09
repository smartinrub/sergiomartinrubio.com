---
title: Troubleshooting Tools for Microservices Architecture
image: https://lh3.googleusercontent.com/CjOHPO6WrWY7oNGe7mhcHBUftlEbkLppOK-xU9godBxsemdsI72V-DIACAMLddNjH28AGQTw2gEs3WHyV8woK6oG6Uh0KuT_QhRq5TfC3O2fMR3idA-D2QdOil-JUbGdyJbU6PVQhElRMRQ763IqkBXKxwqziXB1LE80V5idEztOEW3pmTt8DYPGUVe_I6USlJhk7_TvgRyhQ51GFQVoj6ZksQPuc235hXMQhbpeA65tq5Q6dXlt8ntYmuxoVVCKM1Oo32rZvYBH4LTo1CM8iuFMaWjw8NUVcwZ6bu5r3iNmkJ3DAQ5s81yAabpVSiWahobzY02yqLdJDmsz54SkmdCKlpk0ggAiAlufJXIiONz9JwU13AYx2yqe4UEpqASVrQ6IDyIsGOLJjVPTZDZaAlNSm9VFZUt3dqhq-CGmfYz-uHH1G83GxNGykkN4GoFnPIjKhOdn3Q9r-9E7s39kzsHMcKkpVHuAqb2wO0eCoDRIcT-fcZEtn3P2ijch_9FoR9hHC9oh9UsPdaeS7d4ehKBam0kQlx-PKs6_TFOR4uvkC_BBe_4kV6yKY3MvRYqmrwrZ7KPCF5X4hIZhPx0YlcJr75WdKFqYebnzgW9jOD-9lmBPmeHVoUjPkmdxdMELFObk2Wb7y4bBftT_LbDAJ3szgyfrumU6VnENAuHF7Xng7JgCDH4m5zKXSJmZ=w640-h426-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Troubleshooting
mermaid: false
layout: post
---

## Introduction

One of the biggest challenges when transitioning to a **microservices architecture** is **troubleshooting** and **debugging**. When the number of microservices grows, a simple _HTTP_ request can hit dozens of applications, and in case something goes wrong or the performance is not as good as expected, it might be quite tricky to know where the issue is.

**Logging** and **instrumentation** are very important tools to understand what is going on in a microservices architecture.

- **Logging**: it is used to show what is happening inside your application (errors, exceptions, timeouts, _HTTP_ status…).
- **Instrumentation**: it should be used mainly to capture [Request rate (count), Error rate (count), and Duration of requests](https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/){:target="_blank"}.

One of the most famous tools to capture and process logs is [Splunk](https://www.splunk.com/){:target="_blank"}. Basically, this tool allows us to centralize and manage application logs. It also provides metrics, generates reports and alerts for a particular search, and offers a friendly user interface.

On the other hand, for instrumentation we can use [Zipkin](https://zipkin.io/){:target="_blank"}, which provides a distributed tracing system, and will help you troubleshoot latency issues.

In order to understand how these tools can help us, we are going to build a _Spring Boot_ application which will be fully integrated with **Zipkin** through [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth){:target="_blank"}.

## Example

This [web application](https://github.com/smartinrub/spring-boot-zipkin-sleuth-splunk){:target="_blank"} will only contain an endpoint which will log a simple string with **[Slf4j](https://www.slf4j.org/)**.

```java
@Slf4j
@RestController
@SpringBootApplication
public class SpringBootZipkinSleuthSplunkApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootZipkinSleuthSplunkApplication.class, args);
    }
    @GetMapping
    public String helloWorld() {
        log.info("Hello World Endpoint");
        return "Hello World!";
    }
}
```

The `application.properties` file will contain the configuration for **Sleuth** and **Zipkin**.

```properties
spring.application.name=spring-boot-zipkin-sleuth-splunk
spring.sleuth.sampler.probability=1.0
spring.zipkin.service.name=spring-boot-zipkin-sleuth-splunk
spring.zipkin.base-url=http://zipkin:9411
```

> Note: In order to explicitly provide a different service name for all _spans_ coming from your application we can set `spring.zipkin.service.name` to the desired name.

> `spring.sleuth.sampler.probability` value is 1, which is 100% (default: 0.1, which is 10 percent)

> _localhost_ cannot be used in this example, since the **Spring Boot** application will run inside a **Docker** container, so we have to use the name specified for **Zipkin** on our docker compose file (_http://zipkin:9411_)

**Sleuth** sends its tracing data to **Zipkin** by default, if the following dependency is added to your project.

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

This library is also responsible for adding the trace IDs and store them locally in order to continue the trace.

A [Logback](https://logback.qos.ch/){:target="_blank"} configuration file is also necessary in order to visualize the traces ID in **Splunk**.

```xml
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    ​
    <springProperty scope="context" name="appName" source="spring.application.name"/>
    ​
    <appender name="logstash" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.json.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        "severity": "%level",
                        "service": "${appName:-}",
                        "trace": "%X{X-B3-TraceId:-}",
                        "span": "%X{X-B3-SpanId:-}",
                        "parent": "%X{X-B3-ParentSpanId:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "message": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>
    ​
    <root level="INFO">
        <appender-ref ref="logstash"/>
    </root>
</configuration>
```

**Zipkin** defines **three kind of ids**:

- **Trace Id**: ID shared by every span in a trace
- **Span Id**: ID of a particular span which might be the same as the trace id if only one service is involved during the tracing.
- **Parent Id**: ID present on child spans to know which was the previous span ID. If the span does not have a parent id is considered the root of the trace.

Finally, here is the `docker-compose.yml` that powers the whole setup.

```yaml
version: '2'
services:
  app:
    image: com.thedeveloperhive/spring-boot-zipkin-sleuth-splunk
    environment:
      - LOGGING_FILE=/logs/app.log
    ports:
      - '8080:8080'
    volumes:
      - log_volume:/logs
  splunk:
    image: splunk/splunk
    hostname: splunk
    environment:
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_ENABLE_LISTEN=9997
      - SPLUNK_PASSWORD=password
    ports:
      - '8000:8000'
  splunkforwarder:
    image: splunk/universalforwarder:6.5.3-monitor
    hostname: splunkforwarder
    environment:
      - SPLUNK_START_ARGS=--accept-license --answer-yes
      - SPLUNK_FORWARD_SERVER=splunk:9997
      - SPLUNK_ADD=monitor /logs
      - SPLUNK_PASSWORD=password
    restart: always
    depends_on:
      - splunk
    volumes:
      - log_volume:/logs
  zipkin:
    image: openzipkin/zipkin
    ports:
      - '9411:9411'
volumes:
  log_volume:
```

> Note: **Splunk** works on the client-server model. **Splunk Forwarder** is used to collect the machine generated data from client side and forward to Splunk server.

## Running Application

Build application and run docker compose:

```bash
mvn clean install
docker-compose up
```

> Note: A _Docker_ image will be built during the Maven install stage, since the Spotify docker-maven-plugin was added on the pom file.

If we hit the web service at _[http://localhost:8080](http://localhost:8080)_ a log entry will be generated and forwarded to **Splunk** and **Zipkin**.

```
app_1 | 2019-03-22 21:55:35.647  INFO [-,73d169f5b5e76599,73d169f5b5e76599,false] 1 --- [or-http-epoll-3] .SpringBootZipkinSleuthSplunkApplication : Hello World Endpoint
```

As we can see the **trace ID** is the same as the **span ID**, since only one service was involved during the tracing, and there is **no parent ID**.

**Zipkin** interface is available at _[http://localhost:9411](http://localhost:9411)_

{% include elements/figure.html image="https://lh3.googleusercontent.com/Idvgyeh6Fa7ryJMsC1JaebEf-HRCm7o6Ha_BFUTcwP79Qgv95TOcJYnhPBLhCwCd4YMNELlNXAWrCM-ojJh_0EXoaTwhJapZ8vIMaeVKbyhKUjeCSlAzVQuq0rZk7gr3pBoqTbnb=w2400" caption="Zipkin UI." %}

Now login into _Splunk_ web console ([_http://localhost:8000_](http://localhost:8000)) and search for “_73d169f5b5e76599_”.

{% include elements/figure.html image="https://lh3.googleusercontent.com/61G5AkQxY04-ZUX69Khlb5nSkzfPs2p5aId2utTRFbXwQmGY0c-RsvsalyCafZPiLnB-UyarqcYkTw-qwC5E6qBgDUWiIxADHbxqyeE9dWFUbSzqQ_ZiHHPdYXB_WJpSENCWdT46=w2400" caption="Splunk UI" %}

## Conclusion

Both **logging** and **instrumentation** are essential in any enterprise microservices architecture, and tools like Splunk and **Zipkin** can be excellent allies to act fast and precisely when issues arise.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/spring-boot-zipkin-sleuth-splunk" text="Source Code" %}
</p>
