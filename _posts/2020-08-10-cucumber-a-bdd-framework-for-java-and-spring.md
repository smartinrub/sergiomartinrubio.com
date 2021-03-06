---
title: Cucumber, a BDD Framework for Java and Spring
image: https://lh3.googleusercontent.com/9I4vKZ-7a0gn1r3CgdYbaAla6S9CNe8m295FFqFaJ3peYHI7szViQcM9ldXxjwjJ6bo5ydjcQjBDUyi4VPeq2TeMWaw1bXGR-KxyuOcAgLuLsFQmIt84ZsI2Q5ChakLglHwQNYJbwdN8w0JMV-vpahhXO6qyKw2-UQoGAbLrgOigZcAXwkvyFH1i6m_KhszURICETewLLM_JEV1tI8M_RRp6YkM-eROtD_2MSle-B-rXVEp27BujpMcio5mj6Z9GFc-b8IdGJWWF9j4-DqbwrAA2g1msGfj3ZdJaan2t2OBcgbp-LLFTOaIIFUSElKAHDxC2jJxjAo_Ijs4nrD8VDZDMbfbds_gL92ikmN5-J04IXO5do2qqYVzJbG7H_KVmgy-vxenLHUaZ5MmJNp62X_wACeOS82aT54veuSJqznFhLEeJPU21Sl2e8Jofj_NWWAmNOXNzpy9rV65CtnvJC_eb7SSs82VUBx2-Ib4at9GJtQ0Ct285im7j2KysxWxU0LpqvwQRxBvyvxIboyFGd3oBqA9LJT6_h73rzFOSRoT3N7g6FTU6EFPOlgRCwRO7oxXx9hg7ahA6Ung8B6Sgk6-yilNAgDCw4jUflrdRX_m9tXR33iqcHOuhq_oRO4c3OtT_FjzLcj-5X9Yvoxz_FSQE8o0pgZAt4LtjCHvrJXGX2cVsZgl3pEKaDYjD=w640-h427-no?authuser=1
categories:
    - Testing
    - Spring
    - Java Framework
mermaid: false
layout: post
---

## Introduction

The best way to start using [BDD](https://sergiomartinrubio.com/articles/bdd-fundamentals) with Java is by using one of the BDD test frameworks like Cucumber which allows you to write your test cases using the Gherkin syntax

[Cucumber](https://cucumber.io){:target="_blank"}  is an open source BDD framework that supports many languages like Java, JavaScript, Ruby, C++, Golang or Kotlin.

When using *BDD* with a framework like *Cucumber* you will have to write executable specification which translates to the requirements. This specification is written in a language called *Gherkin* (introduced in the previous section).

## Dependencies

Cucumber will be used in combination with [JUnit 5](https://sergiomartinrubio.com/articles/take-unit-testing-to-the-next-level-with-junit-5) and Spring.

```xml
<!-- Cucumber dependencies -->
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit-platform-engine</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-spring</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
```

To run Cucumber with JUnit 4 you will have to replace  `cucumber-junit-platform-engine` with `cucumber-junit` and also some additional configuration is required.

```xml
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
```

You can run the Cucumber tests with ` mvn clean test `.

## Files Structure

BDD specification is defined in files with the extension `.feature`. A feature file should define a single feature and this file will contain all the examples that describe how the feature works, including edge cases and alternative paths. It is recommended to create subdirectories for each capability and tags to identify related parts of the application, so you can organize the test execution.

By default Cucumber will look for`.feature` files on the classpath. However, you will need to locate your `.feature` files in the same location as the step definition package. e.g. if the step definitions are in `src/test/java/com/sergiomartinrubio/transactionsservice/bdd/stepdefs/ ` the feature file should be located in `src/test/resources/com/sergiomartinrubio/transactionsservice/bdd/stepdefs/`.

If you are using ``cucumber-junit-platform-engine` you can add some [configuration](https://github.com/cucumber/cucumber-jvm/tree/master/junit-platform-engine) in `test/resources/junit-platform.properties`.

```properties
cucumber.plugin=pretty, html:target/cucumber-reports.html
```

`pretty` provides a more verbose login output.

`html:target/cucumber-reports.html` generates an HTML report at the location given.

`json:target/cucumber.json` generates a *JSON* report at the given location that can be used by third party tools like *Jenkins*.

In case you are using `cucumber-junit` you can have:

```java
@CucumberOptions(
        plugin = {"pretty", "html:target/cucumber-reports.html"},
        features = "classpath:features"
)
public class RunCucumberTest {
}
```

## Runner Configuration

If you are using `cucumber-junit-platform-engine` ([*JUnit 5*](https://sergiomartinrubio.com/articles/take-unit-testing-to-the-next-level-with-junit-5)) you will need to create a configuration class that will be used by your Cucumber step definition classes.

```java
@Cucumber
public class RunCucumberTest {
}
```

For `cucumber-junit` (JUnit 4):

```java
@RunWith(Cucumber.class)
public class RunCucumberTest {
}
```

## Spring Configuration

We are going to use Cucumber to test our controller layer of a MVC application, so we have to configure an application context that cucumber will use.

```java
@CucumberContextConfiguration
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class CucumberSpringContextConfiguration {

    @Autowired
    protected TestRestTemplate restTemplate;

}
```

## Executable Specification

Executable specification is plain text definitions of test scenarios that are stored in a `.feature` file. The specification should contain multiple scenarios. The only requirement is that scenarios follow the Gherkin syntax.

In *Cucumber*, you use the `Feature` keyword to mark the feature's title, and any text between the title and the first scenario is considered as the **feature description**.

Scenarios are defined with the `Scenario` keyword followed by a title. The title should summarize what the scenario does in a short and declarative sentence.

The executions steps are made up of three parts (the same as the ones defined in the Gherkin language):

- **The Given step** (`Given`): includes preconditions, context or background (even if no action is executed).
- **The When step**(`When`): includes the action or event you want to test. This action will usually return or generate an outcome.
- **The Then step**(`Then`): is used to compare the outcome from the **When** step with the expectations.

Other Cucumber keywords are `And` and `But` that can be placed after any of the steps, and they will be considered the same as the previous step. The goal is to improve readability and reusability of scenarios, so you do not put two conditions in the same step.

```gherkin
Feature: Return transaction status # feature title
  # feature descrition
  As an user
  I want to be able to retrieve the status of a transaction
  So that I can see the transaction status, amount and fee
  
  Scenario: Get transaction stored in our system given a channel # scenario title
    Given a transaction that is stored in our system with date before today # given step
    When I check the status from INTERNAL channel # when step
    Then the system returns the status PENDING # then step
    And the amount # additional condition to then
    And the fee # additional condition then
```

> Comments are also allowed in `.feature` files by using the character `#`.

The `Background` keyword is used to specify steps that will run before each scenario in the feature.

Tables are also another way of expressing scenarios, so you avoid duplicating the same specification multiple times for different values.

## Step Definitions

Cucumber step definitions are classes with methods annotated with `@Given` `@When` `@Then` `@And` or `@But`.

```java
@Given("a transaction that is stored in our system with date before today")
public void a_transaction_that_is_stored_in_our_system_with_date_before_today() {
  	Transaction transactionBeforeToday = createTransaction(DATE_BEFORE_NOW);
  	transactionService.save(transactionBeforeToday); // you can inject dependencies to setup the scenario
}

@When("I check the status from {channel} channel")
public void i_check_the_status_from_a_given_channel(Channel channel) {
    params.setChannel(Channel.valueOf(channel));
    params.setReference(TRANSACTION_REFERENCE);
    HttpHeaders httpHeaders = new HttpHeaders();
    httpHeaders.add(CONTENT_TYPE, APPLICATION_JSON_VALUE);
    HttpEntity<TransactionStatusParams> statusRequest = new HttpEntity<>(params, httpHeaders);
    response = restTemplate.exchange("/status", HttpMethod.POST, statusRequest, TransactionStatus.class);
}

@Then("the system returns the status {status}")
public void the_system_returns_the_status(String status) {
		assertThat(response.getBody().getStatus()).isEqualTo(status);
}

@And("the amount")
public void and_the_amount() {
  	assertThat(response.getBody().getAmount()).isEqualTo(AMOUNT);
}
```

The previous example shows how we can test the controller layer. 

### Parameter Types

Cucumber allows you to pass variables into your step implementations and use parameter types to convert the text between curly braces into the required type. For instance the string `INTERNAL` will be converted to the type `Channel`. Cucumber also provides some out-of-the-box conversations for types like `BigDecimal`, `Integer`, `Double`, `Float`... however for `Enum` or custom classes you will have to define your own custom parameter type.

```java
@ParameterType("CLIENT|ATM|INTERNAL")
public Channel channel(String channel) {
		return Channel.valueOf(channel);
} 
```

> `@ParameterType` accepts regular expression like `".*"` which allows any string

### Global Variables

A scenario sometimes requires that some information is shared between steps. You can do this by using global variables (e.g. `response`). This global variables are not shared between scenarios, so there is not risk of the values being overridden.

### Pending Scenarios

Scenarios can also be pending when the feature is not implemented yet or you are still working on it. You can mark pending scenarios by throwing `PendingException()`.

```java
@Given("a transaction that is not stored in our system")
public void a_transaction_that_is_not_stored_in_our_system() {
	throw new PendingException();
}
```

### Data Initialization and Clean up

Sometimes tests use a database that need to be initialise before running your tests and clean up after running each scenario. `@Before` and `@After` can be use to do clean up tasks like removing entries from the database. Do no confuse these annotations with the ones from *JUnit*.

```java
@After
public void teardown() {
		transactionRepository.deleteAll();
}
```

### Data Tables

Cucumber also provides embedded data tables in your scenarios. Tables can be used to combine similar scenarios in a single scenario so you avoid duplication and improve maintainability.

You can define the tables in a `.feature` file as follows:

```gherkin
Scenario: Get transaction with date before today given a channel
  Given a transaction that is stored in our system with date before today
  When I check the status from a channel:
    | channel  |
    | CLIENT   |
    | ATM      |
    | INTERNAL |
  Then the status, amount and fees are:
    | status  | amount | fee  |
    | SETTLED | 190.20 |      |
    | SETTLED | 190.20 |      |
    | SETTLED | 193.38 | 3.18 |
```

Then in your step definitions class you will have to create methods annotated with `@DataTableType` in order to allow Cucumber to map the table defined in the `.feature` file with a `Map`.

```java
@DataTableType
public Channel channelEntry(Map<String, String> entry) {
    return Channel.valueOf(entry.get("channel"));
}
```

```java
@DataTableType
public TransactionStatus transactionStatusEntry(Map<String, String> entry) {
    return TransactionStatus.builder()
            .status(Status.valueOf(entry.get("status")))
            .amount(new BigDecimal(entry.get("amount")))
            .fee(Optional.ofNullable(entry.get("fee"))
                    .map((BigDecimal::new))
                    .orElse(null))
            .build();
}
```

Finally you can define the steps definitions that will contain the data from the tables. 

```java
@When("I check the status from a channel:")
public void i_check_the_status_from_a_channel(List<Channel> channels) {
    for (Channel channel : channels) {
        params.setChannel(channel);
        params.setReference(TRANSACTION_REFERENCE);
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.add(CONTENT_TYPE, APPLICATION_JSON_VALUE);
        HttpEntity<TransactionStatusParams> statusRequest = new HttpEntity<>(params, httpHeaders);
        ResponseEntity<TransactionStatus> response = restTemplate
                .exchange("/status", HttpMethod.POST, statusRequest, TransactionStatus.class);
        responses.add(response);
    }
}
```

```java
@Then("the status, amount and fees are:")
public void the_status_amount_and_fee_are(List<TransactionStatus> transactionStatusList) {
    for (int i = 0; i < transactionStatusList.size(); i++) {
        assertThat(responses.get(i).getBody().getStatus()).isEqualTo(transactionStatusList.get(i).getStatus());
        assertThat(responses.get(i).getBody().getAmount()).isEqualTo(transactionStatusList.get(i).getAmount());
        assertThat(responses.get(i).getBody().getFee()).isEqualTo(transactionStatusList.get(i).getFee());
    }
}
```

### Doc String Types

A doc string type can be a json file that is passes as parameter. You can use `@DocStringType` to convert the *string* type to a particular type with the help of `ObjectMapper`.

```gherkin
  Scenario: Get transaction with date after today given a channel
  	...
    Then the status, amount and fees
      """json
      {
         "status": "FUTURE",
         "amount": "190.20",
         "fee": "3.18"
      }
      """
```

```java
private final ObjectMapper objectMapper = new ObjectMapper();

@DocStringType
public TransactionStatus transactionStatus(String docString) throws IOException {
  	return objectMapper.readValue(docString, TransactionStatus.class);
}
```

```java
@Then("the status, amount and fees")
public void the_status_amount_and_fees(TransactionStatus transactionStatus){
 		// assert results
}
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/transactions-service.git" text="Examples" %}
</p>

