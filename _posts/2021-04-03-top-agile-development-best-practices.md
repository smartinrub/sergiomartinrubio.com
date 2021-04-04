---
title: Top Agile Development Best Practices
image: https://lh3.googleusercontent.com/pw/ACtC-3e7ZsCnA6WQAIx0d1cFDmCzKg2IshqL5CfVgJ61wANZ-tbokGccHjt04V1ddeLRMiJ1QBTaIiUaHyyl-751lXNfPDC4mDNz9Qgd2IrL741SL2gGo9Czu-YHNAIzS3TmVLfj9qCWxOzDu4sbROAySB9f=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Software Development Methodology
    - Agile
mermaid: false
layout: post
---

I've been practicing *Agile* development methodologies for a while in different teams and companies and I clearly see the benefits of them. Agile methodologies were formally described in 2001 and they emerged as a *Software Development* Manifesto. This manifesto consist of a set of methodologies like **Extreme Programming**, **SCRUM** or **Feature-Driven Development**. The goal was to achieve a faster feedback  from the user by developing features in small iterations, in contrast to the waterfall model, where everything is planned at the start of the project before writing any code.

## Agile Methods and Practices

### SCRUM

SCRUM is one of the Agile frameworks which goal is to help teams develop and deliver complex features in short iterations. Some SCRUM concepts are:

- ***Sprint***. It's a period of time of around **2 week long** in which a small team of **4 to 8 members** participate in an end to end lifecycle of a feature or product. Why is the length two weeks? 2 weeks is usually enough time to deliver a feature that the team can *demo* (we will talk about *demo* later on) and provide some value to the business. Also, it's important that the team is not either too small or too big. 

- **Team structure**: Scrum teams should be self-organized to avoid the traditional hierarchical roles such as lead developer and decisions should be taken by a common consensus. After my experience in different companies that range from big enterprises to start-ups I consider an ideal agile team those with the following team members:

  - **3-5 engineers**, including either backend engineers or frontend engineers. You might want to have a mix of personality types, so they complement each other.

  - **Product owner**. A product owner is a person whose got a business and technical mind. He is responsible for defining the sprint goal and making sure that the team is on track to deliver value. He will be in continuous communication with the business side of the company (*stakeholders*).

  - **Scrum master (Optional)**.  From my point of view the scrum master role should be present if the team is quite inexperience in Agile practices, so he will make sure that the sprint lifecycle goes smoothly.

  - **Quality Assurance (Optional)**. There are two types of Quality Assurance (QA) roles: technical and non-technical. Those who are QA engineers build functional tests with automation tools. On the other hand, the non-technical QA role is focused on manual testing. Some teams might require this role due to the risk and complexity of the product, however for most of the teams, engineers should be the best people to write acceptance tests.

    > Stakeholders are not part of the SCRUM team, however requirements and feedback are provided by them during "demo" meetings.

- **Scrum meetings**: These meetings are performed every *sprint* lifecycle and will facilitate reaching the *sprint* goal.

  - **Planning**. Sprint planning is a meeting that is used to kick off the *sprint*. During this meeting the team discuss the goal of the *sprint*, how it will be achieved and what *user stories* (we will talk about user stories later) will be part of the *sprint*. It shouldn't be longer than an hour.

  - **Grooming or refinement**. During this session the Scrum team will go through the items on the backlog to add details to the existing items, remove the ones that are not relevant anymore or create new ones. Also, as part of this meeting items are estimated with *story points*. The estimation process is called *planning poker*. **Planning poker** consist of using numbered cards to provide size of each individual task. The estimation given by each team member is discussed in detail to identify the reasons behind their estimation in order to reach a group consensus for the task. **Story points** represent the difficulty of implementing a story and is a combination of complexity, amount of work and uncertainty. I have seen some teams using time (number of days, hours...) as story points and this is a huge mistake for multiple reasons:

    - Dates are not taking into account things like **auxiliary work** engineers need to do to complete tasks, such as meetings, emails or conversations with other engineers. 
    - Team members should be **rewarded by the completion of tasks based on difficulty**, not time spent.
    - The time to complete a task differs depending on the **seniority of the team member**. For instance, a junior engineer might take three times longer to complete a task compare to a senior engineer.
    - Using time can be perceived as micromanagement and a way to put **pressure on team members**.

    You can use as story points the *Fibonacci sequence* (1, 2, 3, 5, 8, 13...), *T-shirt sizes* (e.g. S, M, L, XL...) or any other measure that establishes a proper distinction between relative sizes, so it's clear if a story is significantly smaller or bigger than others. This also gives rough estimates so it's easy for stakeholder to know how big a task is.

    > The backlog contains all the tasks that are pending but are not part of the sprint because either they are not well defined or they are low priority.

  - **Daily stand-up**. These meetings are performed everyday and are meant to keep everyone in the team updated on what other members are working on. Each team member usually provides updates in about a minute and the session shouldn't take longer than 15 minutes in total.

  - **Sprint review or demo**. The main goal of this session is to showcase to stakeholder and business people the work done during the sprint. Stakeholders provide feedback that will be used to understand if the project is heading the right direction. This session happens at the end of every sprint.

  - **Retrospective**. This session should happen the last day of the sprint and is used to spot inefficiencies and issues during the sprint, as well as, celebrate things that are going well. This should be an informal session (a coffee shop outside office is an excellent place for this) where team members share any technical and non-technical difficulties, conflicts, achievements or recognitions. Some examples are: "too many meetings", "inaccurate estimations", "good feedback from stakeholders", "great contribution by X member"... This session is usually structured in multiple rounds and a board with two or three columns (e.g. "went well", "went not so well", "should change").

- **Sprint velocity**: This is mainly used during the sprint planning meeting to decide how many story points should go to the next sprint. It is a productivity measure. The purpose of velocity is to make corrections on estimations.

- **User story**. In an *Agile* project, features are usually broken down into user stories. User stories will help us plan how we can deliver a feature and contain a short description of something small enough to deliver in a small iteration. You can use [BDD techniques](https://sergiomartinrubio.com/articles/bdd-fundamentals/) to write good user stories. [User Story Mapping: Discover the Whole Story, Build the Right Product](https://www.amazon.com/User-Story-Mapping-Discover-Product/dp/1491904909){:target="_blank"} is a fantastic book to learn more about writing good user stories.

### Continuous Deployment

**Continuous deployment** is the process of deploying changes to production, like bug fixes and new features, as frequent as possible. This allows the team to get quick feedback from end users.

### Continuous Delivery

**Continuous delivery** is based on automating the software release process with CI (Continuous Integration) tools like [Jenkins](https://sergiomartinrubio.com/articles/hands-on-cicd-with-jenkins-x/) or Bamboo.

### Extreme Programming

**Extreme programming** is a software development framework that aims to improve customer satisfaction throughout project development. This is achieved by delivering small peaces of software. The continuous feedback from the client and end users can be used to identify how to achieve the business goal.

### Pair Programming

**Pair programming** is a technique mainly used by software engineers in order to share knowledge between team members. This practice is specially  useful for junior engineers and new team members. Pair programming consist of having two programmers, one of them writing the code and the other  providing feedback and/or guidance. 

*Mob programming* is a similar software development technique in which more than two engineers work on the same piece of code at the same time and on the same computer.

### Test Driven Development (TDD)

*TDD* is a software development technique to improve code quality, maintainability and testability. This practice consist of writing a failing test first, then making the test past by writing the implementation details and finally refactoring the solution.

I've been using this technique for quite a while and encourages and promotes you to follow some **SOLID** principles.

- Write decoupled code so you can mock what you need (*Dependency Inversion Principle*).
- Write clear and short tests so you don't have to change too much in the test (*Single Responsibility Principle*).
- Write lighter interfaces (*Interface Segregation Principle*) that make mocking easier.

### Behavior Driven Development (BDD)

[BDD](https://sergiomartinrubio.com/articles/bdd-fundamentals/) is another Agile software development practice that is closely related to user stories. BDD encourages collaboration between software engineers and product owners by using a testing framework based on test scenarios and a *given-when-then* structure. The product owner will provide test scenarios that engineers will implement.

Image by <a href="https://pixabay.com/users/ritae-19628/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=2490817">RitaE</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=2490817">Pixabay</a>
