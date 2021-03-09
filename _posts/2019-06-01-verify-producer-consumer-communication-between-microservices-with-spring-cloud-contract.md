---
title: Verify Producer-Consumer Communication between Microservices with Spring Cloud Contract
image: /assets/images/shubham-beeharry-223969.jpg
author: Sergio Martin Rubio
categories:
    - Testing
    - Spring
    - Java Framework
mermaid: false
layout: post
---

Before **microservices** became popular, we did not have to worry about making sure that different components of your application were using the same POJO class, and any change in a particular entity will affect all its users straight away.

In a microservices architecture, some **testing** aspects have changed, and now our applications **require a different strategy**. **Contract Testing** is one of the solutions which consists of writing tests to ensure that the contracts of our microservices are satisfied and work as expected.

When we talk about Contract Tests, there are **two roles**:

- **Producer**: it is the application providing a service.
- **Consumer**: it is entity consuming the producer API.

## What to test

Contracts should define API specifications offered by microservices, so there is no need for us to test service availability, integrations of layers or load tolerance. 

Our contract should contain elements like _HTTP_ status (e.g. 200, 400...), headers (content-type, cookies...) or response body, for a particular request (method type + path + ...).

## Tools

[Spring Cloud Contract](https://spring.io/projects/spring-cloud-contract){:target="_blank"} provides a simple way to test contracts by implementing the [Consumer-Driven Contracts](https://martinfowler.com/articles/consumerDrivenContracts.html){:target="_blank"} pattern. In this technique, consumers need to satisfy producers expectations, therefore providers are responsible for defining these contracts and share them with its consumers.

## Spring Cloud Contract

### Producer

We need to add this dependency to the producer _POM_ file:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-verifier</artifactId>
    <version>${contract.version}</version>
    <scope>test</scope>
</dependency>
```

Additionally, a plugin in the producer is required to generate tests and stubs for you.

```xml
<plugin>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-maven-plugin</artifactId>
    <version>${contract.version}</version>
    <extensions>true</extensions>
    <configuration>
        <baseClassForTests>com.sergiomartinrubio.reviewservice.BaseClass</baseClassForTests>
        <testMode>EXPLICIT</testMode>
    </configuration>
</plugin>
```

>A base class might need to be created to mock responses or set the context, and this class needs to be set in _baseClassForTests_.

Now we can start defining the contract. Add contracts written in _Groovy_ or _YAML_ to `/src/test/resources/contracts/` package. For example:

    ```groovy
    package contracts
    
    import org.springframework.cloud.contract.spec.Contract
    
    Contract.make {
    
        description("should return all Reviews")
    
        request {
            method(GET())
            url("/reviews")
        }
    
        response {
    
            status(200)
            headers {
                contentType(applicationJsonUtf8())
            }
    
            body("""
                [
                    {"id":"1", "author":"Sergio", "message":"content"}
                ]
            """)
        }
    
    }
    ```

Base class:

    ```java
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, properties = "server.port=0")
    @RunWith(SpringRunner.class)
    @Import(ProducerRestConfiguration.class)
    public class BaseClass {
    
        @LocalServerPort
        private int port;
    
        @MockBean
        private ReviewRepository reviewRepository;
    
        @Before
        public void setUp() {
    
            RestAssured.baseURI = "http://localhost:" + this.port;
    
            when(reviewRepository.findAll())
                    .thenReturn(Flux.just(new Review("1", "Sergio", "content")));
    
        }
    }
    ```

Once the contract and base class are defined, we can run Maven to generate tests and stubs.

    ```shell
    mvn clean install
    ```

Tests will be generated in `/target/generated-test-sources/contracts/`. For the given example the output will be:

    ```java
    public class ContractVerifierTest extends BaseClass {
    
        @Test
        public void validate_shouldReturnAllReviews() throws Exception {
            // given:
                RequestSpecification request = given();
    
            // when:
                Response response = given().spec(request)
                        .get("/reviews");
    
            // then:
                assertThat(response.statusCode()).isEqualTo(200);
                assertThat(response.header("Content-Type")).matches("application/json;charset=UTF-8.*");
            // and:
                DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
                assertThatJson(parsedJson).array().contains("['message']").isEqualTo("content");
                assertThatJson(parsedJson).array().contains("['author']").isEqualTo("Sergio");
                assertThatJson(parsedJson).array().contains("['id']").isEqualTo("1");
        }
    
    }
    ```

The stub will be located under `stubs/mappings`:

    ```json
    "id" : "ba196f42-7f5f-47b3-a1ce-863fe8b96423",
    "request" : {
        "url" : "/reviews",
        "method" : "GET"
    },
    "response" : {
        "status" : 200,
        "body" : "[{\"id\":\"1\",\"name\":\"Sergio\",\"text\":\"text\"}]",
        "headers" : {
        "Content-Type" : "application/json;charset=UTF-8"
        },
        "transformers" : [ "response-template" ]
    },
    "uuid" : "ba196f42-7f5f-47b3-a1ce-863fe8b96423"
    }
    ```

The _Maven Plugin_ will also create a jar with stubs to be uploaded to a central repository (e.g. `/target/review-service-0.0.1-SNAPSHOT-stubs.jar`), so you will be able to run it during the integration test phase.

### Consumer

The consumer will use a stub runner that allows you to automatically download stubs generated by the producer (or pick those from the classpath), start WireMock servers and feed them with stub definitions.

Consumer dependency:

    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
        <scope>test</scope>
    </dependency>
    ```

Consumer test suite:

    ```java
    @RunWith(SpringRunner.class)
    @AutoConfigureStubRunner(ids = "com.sergiomartinrubio:review-service:+:8080",
            stubsMode = StubRunnerProperties.StubsMode.LOCAL)
    @Import({ReviewConsumerApplication.class, ReviewClient.class})
    public class ReviewWireMockTest {
    
        @Autowired
        private ReviewClient reviewClient;
    
        @Test
        public void clientShouldReturnReview() {
            StepVerifier.create(reviewClient.getAllReviews())
                    .expectNextMatches(review -> review.getAuthor().equals("Sergio") && review.getMessage().equals("content"))
                    .verifyComplete();
        }
    }
    ```

`@AutoconfigurationStubRunner` will set: 

* ids:
    * `groupId` of the artifact.
    * `artifactId` of the producer.
    * producer version (+ means latest).
    * port on which the generated stub will run.
* stubsMode:
    * `StubRunnerProperties.StubsMode.CLASSPATH` (default value) - will pick stubs from the classpath.
    * `StubRunnerProperties.StubsMode.LOCAL` - will pick stubs from a local storage (e.g. _.m2_).
    * `StubRunnerProperties.StubsMode.REMOTE` - will pick stubs from a remote location.

As we mentioned, the consumer uses the stub generated by the producer, so if the producer changes the API the contract will break.

## Conclusion

Making sure that our APIs work as advertised is very important in the microservices era, since many teams might work in different part of our system and we need to align producers and consumers to avoid misunderstandings at a late stage of the development process. Using _Wiremock_ in isolation is not enough, because producers may change their specification and we might not be aware of these changes, therefore it is very important to test the contract defined between producers and consumers at every stage.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/testing-spring-microservices" text="Source Code" %}
</p>
