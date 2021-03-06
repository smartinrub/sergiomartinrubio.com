---
title: Quartz. De-facto Choice for Job Scheduling in Java
image: https://lh3.googleusercontent.com/pw/ACtC-3dtMF4Fq3oJlZjZixXvAB2bcqips9fS4uw1GiojzP_abjKiydIM4nwzdUllMzr3obBHl4K7Q2pubs--TMxXRvUdW9M45oO6UgPSgCSJcrpfQa8IUnfSWlGgCkpd1nNcuZTellCeTxiKOUpqrI4m7KSF=w640-h426-no?authuser=0
categories:
    - Java Framework
    - Spring
    - Scheduler
mermaid: false
layout: post
---

## Introduction

Schedulers are sometimes required for recurring tasks like reminding customers that they have to pay before a deadline, and [Quartz](http://www.quartz-scheduler.org){:target="_blank"}  is one of the best options out there when looking for a scheduling library for a Java enterprise application.

**Why Quartz?** There are other good alternatives like [db-scheduler](https://github.com/kagkarlsson/db-scheduler){:target="_blank"}  that provides a simpler API however Quartz is a more mature library with a larger community.

## Features

- Schedule jobs at a particular time
- Repeat job executions
- Suports [JTA transactions](https://sergiomartinrubio.com/articles/jpa-introduction-to-java-persistence-api#persistence-unit)
- Store jobs in a relational database via [JDBC](https://sergiomartinrubio.com/articles/working-with-jdbc-api)
- Supports clustering
- [Quartz Spring integration](https://docs.spring.io/spring-boot/docs/2.1.x/reference/html/boot-features-quartz.html)
- [Liquibase](https://github.com/liquibase/liquibase) integration

## Quartz Components

### Scheduler

The `Scheduler` interface is the main class of Quartz scheduler and its role is to take care of the lifecycle of the jobs. It allows you to schedule a new job, check if a job exists, retrieve the details of an existing job, delete a job...

### Job

The `Job` interface is implemented by the classes that contain the business logic that we schedule with the `Scheduler`.

### JobDetails

The `JobDetails` is created through the `JobBuilder` and it holds all the data related to the job.

### Trigger

The `Trigger ` is created through the `TriggerBuilder` and it defines the job execution details like the starting time.

## Getting Started

We are going to build a simple web application that will use Quartz in combination with Spring Boot.

### Setup

First of all we need to include in our `pom.xml` the Spring Boot starter dependency that provides the integration with Quartz and the JDBC Spring Boot starter dependency.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

now we can decide how we will store the schedule jobs. We can use an in-memory store or a JDBC-based store. If you are planning to run your application in production we recommend you use the JDBC option.

```yaml
### Quartz configuration
spring:  
  quartz:
    job-store-type: jdbc
    jdbc:
      initialize-schema: never
```

When using a *jdbc* store type you will also need to specify the schema initialization strategy: embedded (only initialize embedded datasource - default value), always (always initialize datasource), never (do not initialize datasource))

If you don't initialize the schema you might use something like liquibase and provide [the Quartz Liquibase changelog](https://raw.githubusercontent.com/quartz-scheduler/quartz/master/quartz-core/src/main/resources/org/quartz/impl/jdbcjobstore/liquibase.quartz.init.xml){:target="_blank"}  that contains all the required tables.

> You can convert a Liquibase XML file to .yaml with [Liquify](https://github.com/daquino/liquify){:target="_blank"} 

Furthermore, if you are planning to run multiple instances of your scheduler application you should enable the clustering feature.

```yaml
### Quartz configuration
spring:
  quartz:
    properties:
      org:
        quartz:
          jobStore:
            isClustered: true
          scheduler:
            instanceId: AUTO # instance ID is generated for you
```

### Implementation

You can create a `Job` implementation as follows.

```java
@Slf4j
public class MyJob implements Job {
    private static final String JOB_NAME_KEY = "jobName";
    private static final int MAX_RETRIES = 3;

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        JobDataMap jobDataMap = jobExecutionContext.getMergedJobDataMap();
        int refireCount = jobExecutionContext.getRefireCount();
        MyJobData jobData = new MyJobData(jobDataMap.getString(JOB_NAME_KEY));

        try {
            if (refireCount > MAX_RETRIES) {
                throw new JobExecutionException(format("Job execution retries exceeded for job name %s",
                        jobData.getJobName()));
            }

            log.info("Job execution for name: {}", jobData.getJobName());
        } catch (Exception e) {
            JobExecutionException jobExecutionException = new JobExecutionException(e);

            // fire it again
            jobExecutionException.setRefireImmediately(refireCount <= MAX_RETRIES);
            throw jobExecutionException;
        }

    }
}
```

In the `JobExecutionContext` we can find the data store during the job creation and other information about the job execution like the number of retries, so you can control how many times a job will execute in case of failure.

You can schedule or delete a Job with the `Scheduler` interface.

```java
private static final String MY_JOB_GROUP = "my-quartz-job";
private static final String JOB_NAME_KEY = "jobName";

private final Scheduler scheduler;

public void scheduleJob(String jobName, SchedulerParams schedulerParams) throws SchedulerException {
    JobKey jobKey = JobKey.jobKey(jobName, MY_JOB_GROUP);
    MyJobData myJobData = new MyJobData(jobName);

    if (scheduler.checkExists(jobKey)) {
        throw new JobAlreadyExistsException(jobName);
    }

    log.info("Updating job for name {} and time {}", jobName, schedulerParams.getJobDateTime());
    JobDataMap jobDataMap = getJobDataMap(myJobData);
    JobDetail jobDetail = getJobDetail(jobKey, jobDataMap);
    Trigger trigger = getTrigger(schedulerParams.getJobDateTime());

    scheduler.scheduleJob(jobDetail, trigger);
}
```

When you schedule a job you will usually provide a `Trigger` and  `JobDetails`. 

```java
private Trigger getTrigger(ZonedDateTime jobDateTime) {
    return TriggerBuilder.newTrigger()
            .withIdentity(UUID.randomUUID().toString(), MY_JOB_GROUP)
            .startAt(Date.from(jobDateTime.toInstant()))
            .build();
}
```

In the trigger you will set things like start time or job groups.

```java
private JobDetail getJobDetail(JobKey jobKey, JobDataMap jobDataMap) {
    return JobBuilder.newJob(MyJob.class)
            .withIdentity(jobKey)
            .usingJobData(jobDataMap)
            .requestRecovery(true)
            .build();
}

private JobDataMap getJobDataMap(MyJobData jobData) {
    JobDataMap jobDataMap = new JobDataMap();
    jobDataMap.put(JOB_NAME_KEY, jobData.getJobName());
    return jobDataMap;
}
```

On the other hand `JobDetail` is used to set the class that implements the `Job` interface and pass some information  via `JobDataMap`, that can be consumed by the job later on. We use a `JobKey` to uniquely identify a  `JobDetail`. 

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/spring-boot-quartz.git" text="Examples" %}
</p>


