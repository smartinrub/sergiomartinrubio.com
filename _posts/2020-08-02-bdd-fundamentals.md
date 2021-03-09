---
title: BDD Fundamentals
image: https://lh3.googleusercontent.com/-QqGaTORp-09k7Fy1Nr_WSSupGfao_WzpMl0X5YOrnFXxVKbLtvjX0Hz12B2BRWUw5NKqMyI5MBI7-DgNzwxsxntPJs0qDAUZ748s-O5OCr6c957-w68ABs_R4owCb1NQysCdVBXl0DrFVREBCKtCA4Qiwazo6PIP1cyE-YohngdYFWVVHf0L79_AKzWdS61Obr4m94RtM_EAnczk4JP3bgLVHuEcSF3z8Nxs9mqx_owq3-4r3hTYmCLgmHYcHbW6pJsQNl0vSEvNg_akHPpzvThLVwI4eGa9_PsNLcwv8UlChj_WWWDOUdTatW3h1gTjoZWBYkWJLBQCmGMJRy7-AEtIj2UPkMMdX9626mCUTbvLLmUTwrMNMqzszpZxML5qU64vEHZFg4JbdqoAg7vmkslMSk6W1Ce9hTPm1uss8IJLeHlRvwtVrqHaXhAjGZFB__M3ZJ0Zjtu0UELHCTvnKKUv4EOUgjfEJKMps56JI99wnXdwe_aqP3tN0cc_-gLHvCCMGTfwW5YGyRdSs8yqrtaSLDC3WMpcnwAti1cJZbvlDUE3f3pbWflPPy7jfHVJnJR1L_pAQNYaGQNh3JMXOp-Z8v70Pi006ACFhoUcV9lDgoWDg_xExheUpg0YcTaf_2a2W3Q8IHFTgGKUswLP6pQSEsie1GRAECAl2ZFIIt40P7HQ_BAejnaJTq_=w640-h427-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Testing
mermaid: false
layout: post
---

*Behavior Driven Development* (BDD) provides techniques that can help you manage software development uncertainty and risk. BDD is about building the right software.

> BDD was originally defined by [Dan North](https://dannorth.net){:target="_blank"} in 2009 in London as an extension of TDD (Test Driven Development) that describes a cycle of interactions with particular outputs.

> TDD is a technique used to make sure the code you write does what you expect. It consist of writing a failing test first and implementing the minimal amount of code that makes the test pass afterward. Then you will refactor the code to make it clean and apply design patterns. You will have to repeat the process for other scenarios or edge cases.

BDD techniques consist in identifying business goals and looking for features that will help deliver these goals, so that you should prioritize those features that will deliver the most value.

Behavior Driver Development can be applied to the UI layer and non-UI layers. When BDD is applied to the non-UI in a MVC application, you can easily apply these techniques to the controller layer.

**BDD advantages**:

- Allows developers to focus on building features that will provide business value.
- Reduces the number of bugs and the cost of fixing them.
- Makes it easier to make changes in your application, since the specification can be used by the stakeholders to understand what the application does.
- Reduces the risk of regression tests.
- Speeds up the release cycle since it reduces the amount of testing required by QA engineers.

**BDD disadvantages**:

- BDD requires that stakeholders, business analysts and QA engineers collaborate from the beginning of a project.
- Requirements are needed upfront.
- It requires that tests are carefully designed so they can be easily maintained.

## Gherkin Syntax

Many *BDD* frameworks use a syntax known as *Gherkin*. This syntax is used to describe the requirements and expectations of a particular feature. Gherkin syntax usually follows the order of **Given... When.. Then**...:

- **Given** will contain the preconditions and the elements required to run the test case.
- **When** contains the action that triggers the test case.
- **Then** contains the expectations or expected result.

Additionally, you can use **And** or **But** when multiple conditions or expectations are needed. e.g.

```gherkin
Given a transaction that is stored in our system with date before today
When I check the status from ATM channel
Then the system returns the status SETTLED
And the amount substracting the fee
```

## BDD Steps

1. Define **business goal**
2. Find out what **features** are required to achieve the business goal
3. Describe **examples** that will help understand how the features will work
4. Write **executable specifications** to organize development and testing
5. Write **business logic**

### Business Goal

A Business goal is about delivering value. They should be translated into increasing revenues, decreasing costs, saving time...  It's important to have a fully understanding of the business context so you can react quickly if the requirements change. You can write business goals following any of these two templates: 

*In order to ... As a ... I want to ...*

```
In order to see the transaction status, amount and fee
As a user
I want to be able to retrieve a transaction.
```

*As a ... I want ... so that ...*

```
As an user
I want to be able to retrieve a transaction
So that I can see the transaction status, amount and fee
```

>These templates are commonly use in story cards.

------

Goals should be [SMART](https://en.wikipedia.org/wiki/SMART_criteria){:target="_blank"}:

- **Specific**: A goals should say what you're trying to achieve.
- **Measurable**: You should be able to determine if the goal is actually achieved when the work is done.
- **Achievable**: You should make a goal tangible with realistic expectations.
- **Relevant**: A goal should deliver value.
- **Time-bound**: A goal should be time-boxed.

------

Most of the time when you start a project you don't really know what value you are supposed to deliver, that's why it's important to start conversations with product owners or stakeholders and ask the following questions:

- **Why** do we want to work on the feature? e.g. *increase transactions*
- **Who** is going to be the consumer of the feature? e.g. *bank customers*
- **How** can we achieve the goal? e.g. *showing transactions*
- **What** do we need to build to achieve the goal? e.g. *provide an API*

### Features

A **feature** is what we deliver to provide functionalities. A feature can be something like "*get transaction status for a particular transaction*". 

Developers and product owners should determine what features are needed to achieve a goal. When the features are discussed you need to describe them in more detail and prioritize those ones that will deliver the most value.

In an *Agile* project features are usually broke down into user stories. User stories will help us plan how we can deliver a feature and usually contain a short description of something small enough to deliver in a small iteration. Therefore, you might need several user stories to build an entire feature. We can also use an *epic* to group a set of stories that have something in common.

> An epic describes a more general task and is usually implemented during a few Scrum sprints.

```
TP-101
Providing Transaction Status When Sending Transaction ID

As an user
I want to be able to retrieve a transaction
So that I can see the transaction status, amount and fee
```

When you implement a story you should get together with product owners, developers and testers to describe what the feature should do (*Three Amigos*, see below).

> **Three Amigos**: Is a session in which three team members, a developer, a tester and a product owner, get together to discuss and write up an initial draft of the scenarios of a feature. They will define examples, use cases, point out technical considerations...

As we've seen, user stories are used as a tool to organize how we are going to deliver a feature, and you should only create stories when you are ready to start implementing the feature. 

### Examples and Executable Specification

**Examples** can be defined as the different scenarios for a particular features. Examples consist of a **title** that summarise what the example is trying to achieve; a **description**; a **scenario title**; a **scenario body** which contains a *Gherkin* structure *given...when...then*.

```gherkin
Feature: Return transaction status # feature title
  # description
  As an user
  I want to be able to retrieve the status of a transaction
  So that I can see the transaction status, amount and fee
  Scenario: Get transaction stored in our system given a channel # scenario title
    # scenario body
    Given a transaction that is stored in our system with date today
    When I check the status from CLIENT channel
    Then the system returns the status PENDING
    And the amount substracting the fee
```

It's recommended to keep your scenarios simple by describing the behaviors with tables.

Executable specification acts as *acceptance criteria* and guideline for developers, so they are an excellent tool keep track of the progress of a feature. 

You can use tools like [Cucumber](https://sergiomartinrubio.com/articles/cucumber-a-bdd-framework-for-java) to write the executable specification.

```java
@Given("a transaction that is stored in our system")
public void a_transaction_that_is_stored_in_our_system() {
  	Transaction transactionAfterToday = createTransaction(DATE_AFTER_NOW);
  	transactionService.save(transactionAfterToday);
}
```

As part of the executable specification we should include the *living documentation*. The **living documentation** is the reference or source of truth of how a feature works. This documentation is usually generated as a report by tools like [Serenity BDD](https://serenity-bdd.github.io/theserenitybook/latest/index.html) (previously called *Thucydides*).

#### Tips When Writing Executable Specification

- Use an **outside-in or top down strategy** to give you a high level vision of the feature you are building so you can focus on the business value.

- **Automate BDD tests** by making them run as part of the build process, so they are triggered whenever a change is pushed to your version control system. These tests will became the acceptance tests and regression tests of your application.

  > Regression testing means running all the functional and non-functional tests to make sure new changes are not breaking the application or affecting current features.

- **Initialize the database** at the start of the test suite and **clean up** after each scenario.

- **Set up the scenarios** with the required data. You can define data tables as part of scenario body that lives in feature files.

- **Use a *persona***. A *persona* is a well-known entity that everyone on the team knows. e.g. John Smith

```gherkin
Scenario: Get transaction stored in our system given a channel
  Given a transaction that is stored in our system with date before today
  When John Smith checks the status from CLIENT channel
  Then the system returns the status SETTLED
  And John Smith should see the amount substracting the fee
```

### Business Logic

The implementation details will contain low level specification that interact with the system and is tested with unit-testing tools like [JUnit](https://sergiomartinrubio.com/articles/take-unit-testing-to-the-next-level-with-junit-5). BDD can be also applied to unit testing with tools like [Spock](http://spockframework.org).

## Steps to Implement a Scenario

As we mentioned before Developers should follow an outside-in strategy. You should start from the acceptance criteria and build the implementation details of what is required to make the acceptance criteria pass. Steps (the following examples are using [Cucumber](https://sergiomartinrubio.com/articles/cucumber-a-bdd-framework-for-java)):

- **Define the high-level acceptance criteria** with the Gherkin syntax.

```gherkin
Scenario: Get transaction stored in our system given a channel
  Given a transaction that is stored in our system with date before today
  When I check the status from CLIENT channel
  Then the system returns the status SETTLED
  And the amount substracting the fee
```

- **Implement the step definitions** and set them as pending.

```java
@Given("a transaction that is stored in our system with date before today")
public void a_transaction_that_is_stored_in_our_system_with_date_before_today() {
    throw new PendingException();
}
```

- **Write unit tests for the business logic** required for one of the step definitions.

```java
@Test
void shouldSaveTransaction() {
    // WHEN
    transactionService.save(TRANSACTION);

    // THEN
    verify(transactionRepository).save(TRANSACTION);
}
```

- **Implement the required code** for the unit test to make the tests pass.

```java
public void save(Transaction transaction) {
    transactionRepository.save(transaction);
}
```
