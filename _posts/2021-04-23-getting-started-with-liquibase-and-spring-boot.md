---
title: Getting Started with Liquibase and Spring Boot
image: https://lh3.googleusercontent.com/pw/ACtC-3eedkF-BJTMIwxSEiK_tlKAa03ey17QlkPlOQH7_i-vGvwrKifbUiq6g1WIe6JvF3mnjRRDTpm7pRMHCRt4j37GMzwdw4EcAG-xLnWCiXPKnTRLEb9unFbWzhyzxt3_3E2Q464l7qimeImvdH6T3kKp=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Databases
mermaid: false
layout: post
---

[Liquibase](https://www.liquibase.org){:target="_blank"} is a popular library for versioning and deploying database changes. This is an open source library and widely use in the *Java ecosystem* or as part of a  *Jenkins* build.

Manual *SQL* schema updates are commonly used but they are not recommended if you want to keep track of the changes in your database. With Liquibase you can know what changes have been applied and when they were made so you could easily roll back the SQL schema to a previous state.

## The Internals

Liquibase simply runs [SQL commands](https://sergiomartinrubio.com/articles/mysql-guide/) and adds records of the changes on a table named `DATABASECHANGELOG` which is responsible for keeping track of all the changes on a particular database. Every time a new change is applied through Liquibase a new record is added to the  `DATABASECHANGELOG`.

Each change log record has its own *MD5 hash* and they are checked against the Liquibase changeset files when loading the *Liquibase* configuration. If a change is made on a changeset that is already applied, Liquibase will interrupt the execution and log a validation error in order to prevent modifying the history of changes.

## How to Use

We are going to use Liquibase in combination with *Maven* and *Spring*.

1. Create a Spring Boot project at the [Spring initializr site](https://start.spring.io){:target="_blank"}.

1. Add the required dependencies to your `pom.xml` file. You will need at least three dependencies: the database connector (e.g. `postgresql`, `mysql`...), the [JPA](https://sergiomartinrubio.com/articles/jpa-introduction-to-java-persistence-api/) dependency and the liquibase dependency.

   ```xml
   <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <scope>runtime</scope>
   </dependency>
   
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-jpa</artifactId>
   </dependency>
   
   <dependency>
      <groupId>org.liquibase</groupId>
      <artifactId>liquibase-core</artifactId>
   </dependency>
   ```

2. Add the database configuration to your `application.properties` or `application.yml`. e.g.:

   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/my-database
   spring.datasource.driver-class-name=org.postgresql.Driver
   spring.datasource.username=postgres
   spring.datasource.password=postgres
   ```

3. Create a master change set file (`db.changelog-master.yaml`) in the default Liquibase changelog folder (`src/main/resources/db/changelog`).

   ```yaml
   databaseChangeLog:
     - include:
         file: db/changelog/changeset/document-table.yaml
     - include:
         file: db/changelog/changeset/person-table.yaml
   ```

   You can organize you change sets in different files, for example, one for each SQL table.

   e.g. `src/main/resources/db/changelog/changeset/document-table.yaml`:

   ```yaml
   databaseChangeLog:
     - changeSet:
         id: craete-document-table
         author: Sergio Martin Rubio
         changes:
           - createTable:
               tableName: document
               columns:
                 - column:
                     name: customer_id
                     type: varchar(255)
                     constraints:
                       primaryKey: true
                       nullable: false
                 - column:
                     name: document_url
                     type: varchar(255)
                     constraints:
                       nullable: false
   ```

4. Get your database up and running. For example you can use Docker to spin up a PostgreSQL database:

   ```yaml
   version: '3'
   services:
     postgres:
       container_name: postgres
       image: "postgres:latest"
       ports:
         - "5432:5432"
       environment:
         POSTGRES_USER: postgres
         POSTGRES_PASSWORD: postgres
         POSTGRES_DB: my-database
   ```

5. Run your *Spring Boot* application from the command line with `./mvnw spring-boot:run`

>  Liquibase also provides an API to configure things like the location of the master changeset file (`spring.liquibase.change-log`). A [full list of all the properties can be found on the Spring documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#spring.liquibase.change-log){:target="_blank"}.

## Liquibase Directives

Some of the most common Liquibase directives are:

- [Create a table](https://docs.liquibase.com/change-types/community/create-table.html){:target="_blank"}:

  ```yaml
  - changeSet:
      id: craete-person-table
      author: Sergio Martin Rubio
      changes:
        - createTable:
            tableName: person
            columns:
              - column:
                  name: id
                  type: varchar(255)
                  constraints:
                    primaryKey: true
                    nullable: false
              - column:
                  name: customer_id
                  type: varchar(255)
                  constraints:
                    nullable: false
                    foreignKeyName: fk_person_document
                    references: document(customer_id)
              - column:
                  name: last_name
                  type: varchar(255)
                  constraints:
                    nullable: false
              - column:
                  name: first_name
                  type: varchar(255)
                  constraints:
                    nullable: true
              - column:
                  name: aliases
                  type: varchar(255)
                  constraints:
                    nullable: true
              - column:
                  name: category
                  type: varchar(255)
                  constraints:
                    nullable: true
              - column:
                  name: position
                  type: varchar(255)
                  constraints:
                    nullable: true
              - column:
                  name: keywords
                  type: varchar(255)
                  constraints:
                    nullable: true
              - column:
                  name: entered
                  type: date
                  constraints:
                    nullable: true
              - column:
                  name: updated
                  type: date
                  constraints:
                    nullable: true
  ```

- [Modify a column](https://docs.liquibase.com/change-types/community/modify-data-type.html){:target="_blank"}:

  ```yaml
  - changeSet:
      id: change-aliases-column-type
      author: Sergio Martin Rubio
      changes:
        - modifyDataType:
            columnName: aliases
            newDataType: clob
            tableName: person
  ```

- [Add primary key](https://docs.liquibase.com/change-types/community/add-primary-key.html){:target="_blank"}: 

  ```yaml
  - changeSet:
      id: add-primary-key
      author: Sergio Martin Rubio
      changes:
        - addPrimaryKey:
            columnNames: id
            tableName: person
  ```

- [Drop primary key](https://docs.liquibase.com/change-types/community/drop-primary-key.html){:target="_blank"}:

  ```yaml
  - changeSet:
      id: drop-primary-key
      author: Sergio Martin Rubio
      changes:
        - dropPrimaryKey:
            dropIndex: true
            tableName: person
  ```

- [Add foreign key](https://docs.liquibase.com/change-types/community/add-foreign-key-constraint.html){:target="_blank"}: 

  ```yaml
  - changeSet:
      id: add-foreign-key
      author: Sergio Martin Rubio
      changes:
        - dropForeignKeyConstraint:
          baseColumnNames: customer_id
          baseTableName: person
          constraintName: fk_person_document
          referencedColumnNames: customer_id
          referencedTableName: customer
  ```

- [Drop foreign key](https://docs.liquibase.com/change-types/community/drop-foreign-key-constraint.html){:target="_blank"}: 

  ```yaml
  - changeSet:
      id: drop-foreign-key
      author: Sergio Martin Rubio
      changes:
        - dropForeignKeyConstraint:
            baseTableName: person
            constraintName: fk_person_document
  ```

> A full list of changes that can be applied can be found on the [official Liquibase documentation](https://docs.liquibase.com/change-types/home.html){:target="_blank"}.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-liquibase-example.git" text="Examples" %}
</p>
