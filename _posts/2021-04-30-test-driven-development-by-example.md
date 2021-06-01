---
title: Test-driven Development by Example
image: https://lh3.googleusercontent.com/pw/ACtC-3ePXwza8pvlQgfenEDyYMRWaaOdi6vX9l-jOzOyuIXSoFxvd5vQIqC7wG6oKeuK5b1m7Og3hWBJ7Tl422kIhn0NA75pyAJyI6AprtczqhqOVKyMHd2L4WCz879kfZNm8djth07kpFVzbxmfRGARocrl=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Testing
    - Java
    - Spring
mermaid: false
layout: post
---

**Test-driven development** commonly known as TDD is a testing development technique which is based on **writing tests before writing the implementation details**. TDD practitioners claim that this technique improves not only the code quality but also **boosts developers mood** while writing code. This increase of mood comes from the psychological fact of seeing things working. On the contrary, if you procrastinate the creation of tests, the creation of tests will become a painful chore.

On the other hand, this technique forces you to have **fast feedback**, since you have to create tests before introducing any new change, so you can deliver value and go to production quickly, and as a result, TDD is very recommended in combination with other [Agile techniques](https://sergiomartinrubio.com/articles/agile-development-best-practices/#test-driven-development-tdd).

TDD can be also combined with other techniques like **pair programming** and this is called **Ping Pong Programming** (also known as *Red, Green, Refactor*): one developer writes a red test and another deliver writes the code that makes the test pass.

**ATDD** (*Acceptance Test Driven Development*) is a variant of TDD and is focused on starting testing from the user's point of view. Testing frameworks like [Cucumber](https://sergiomartinrubio.com/articles/cucumber-a-bdd-framework-for-java-and-spring/) are commonly used for these kind of tests. This is also called [Behaviour Driven Development (BDD)](https://sergiomartinrubio.com/articles/bdd-fundamentals/).

## Key Factors for Using TDD

These are some takeaways from [The Three Laws of TDD](https://www.youtube.com/watch?v=qkblc5WRn-U) by *Uncle Bob*.

- Produces **documentation**. Tests are examples of how to use the code you write.
- **Decouples the code**. You will write testable code and as a result you will have to decouple your code to make it testable.
- You make sure **your system works**.

## "Reasons for not Using TDD"

- **We have to go fast** because maybe we have deadlines, but this means <u>you will pay for the shortcuts later on</u> (and with interests!).
- **We want to simply solve the problem** and <u>leave a mess behind us</u>. We probably won't go back and clean it up.

As you can assume, these reasons are not really good reasons for not using TDD, and <u>they will result in future headaches and fear of refactoring the code</u> because we might break the existing code and there is not a test suite that ensures that the changes are not breaking the original business logic.

## Getting Started

You can start using TDD by following the next steps:

1. Write a *red* test that satisfy a particular feature.
2. Write the minimum business logic that makes the test pass.
3. Refactor the business logic applying architectural patterns, *SOLID* principles, *DRY*...

## TDD by Example

{% include elements/video.html id="0F0etcoSfgE" %}

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/add-by-example.git" text="Source Code" %}
</p>
