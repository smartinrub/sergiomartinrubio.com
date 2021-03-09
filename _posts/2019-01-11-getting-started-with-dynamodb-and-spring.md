---
title: Getting Started with DynamoDB and Spring
image: /assets/images/shubham-beeharry-223969.jpg
author: Sergio Martin Rubio
categories:
    - Database
    - NoSQL
    - Spring
    - Java Framework
mermaid: false
layout: post
---

## Introduction

[DynamoDB](https://aws.amazon.com/dynamodb/?sc_channel=PS&sc_campaign=acquisition_UK&sc_publisher=google&sc_medium=dynamodb_hv_b&sc_content=dynamodb_e&sc_detail=dynamodb&sc_category=dynamodb&sc_segment=62042493529&sc_matchtype=e&sc_country=UK&s_kwcid=AL!4422!3!62042493529!e!!g!!dynamodb&ef_id=EAIaIQobChMIncaip-jP3wIVmoXVCh3v2QHsEAAYASAAEgLIzPD_BwE:G:s){:target="_blank"} is a NoSQL database provided by AWS, and in the same way as [MongoDB](https://www.mongodb.com/){:target="_blank"} or [Cassandra](http://cassandra.apache.org/){:target="_blank"}, it is very suitable to boost horizontal scalability and increase development speed.

## Main Features

- Fully managed _NoSQL_.
- Document or Key-Value.
- Scales to any workload. _DynamoDB_ allows you to auto-scaling, so the throughput adapts to your actual traffic.
- Fast and consistent.
- Provides access control.
- Enables _Event Driven Programming_.

## DynamoDB Components

- **Tables**. Catalog
- **Items**. Group of attributes
- **Attributes**. Data elements
- **Partition Key**. Mandatory, Key-Value access pattern. Determines data distribution
- **Sort Key**. Optional. Model 1:N relationships. Enables rich query capabilities

## Guidelines

* Understand the use case.
    - Nature of the application.
    - Define the [E/R Model](https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model){:target="_blank"}
    - Identify the data life cycle (_TTL_, Backups…).
* Identify the access patterns.
    - Read/Write workloads.
    - Query dimensions.
* Avoid relational design patterns, and instead, use one table to reduce round trips and simplify access patterns. Identify _Primary Keys_ and define indexes for secondary access patterns.
* Select a strong _Partition Key_ with a large number of distinct values. Do not use things like _Status_ or _Gender_. Use `UUID`, `CustomerId`, `DeviceId`...
* Items are uniformly requested and randomly distributed.
* Select _Sort Keys_ which follows a model `1:n` and `n:n` relationships.
* Use efficient and selective patterns for _Sort Keys_. Query multiple entities at the same time to avoid many round trips.

The official [DynamoDB documentation for best practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html){:target="_blank"} provides more information.

However, **DynamoDB** is not for everyone. In _NoSQL_, you are tuning the data to the access particular patterns. Therefore, if you do not know what queries are going to be performed, or your system is very normalized, _NoSQL_ databases might not be a good candidate.

## Spring Integration

[One of the languages supported by DynamoDB is Java](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/basics.html){:target="_blank"}, and you can develop you own API to query _DynamoDB_. However, you will realize soon that there is a lot of boilerplate for very simple queries.

Spring provides a community [module to integrate DynamoDB on Spring](https://github.com/derjust/spring-data-dynamodb){:target="_blank"}, which is built on top of [Spring Data](https://spring.io/projects/spring-data){:target="_blank"}. This module supports:

* **CRUD** operations for DynamoDB entities.
* **Lookup strategy** from query method names (only supported from `5.1.0-SNAPSHOT`).
* **Integration with custom repositories**.
* **Spring annotations**.
* **REST support** via spring-data-rest.

## Getting Started

To start using this module, you just need to add these two dependencies:

[spring-data-dynamodb](https://mvnrepository.com/artifact/com.github.derjust/spring-data-dynamodb){:target="_blank"}
[aws-java-sdk-dynamodb](https://mvnrepository.com/artifact/com.amazonaws/aws-java-sdk-dynamodb){:target="_blank"}

```xml
<profiles>
    <profile>
        <id>allow-snapshots</id>
        <activation><activeByDefault>true</activeByDefault></activation>
        <repositories>
        <repository>
            <id>snapshots-repo</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            <releases><enabled>false</enabled></releases>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        </repositories>
    </profile>
</profiles>
```

### DynamoDB Set Up

First of all, you need a DynamoDB web service instance on AWS or run locally a downloadable version of _DynamoDB_. For testing, you can simply use a dockerized _DynamoDB_ version:

```shell
docker run -p 8000:8000 amazon/dynamodb-local
```

### Spring Configuration

Firstly, you need to add `@EnableDynamoDBRepositories` annotation on our configuration class. And you can have two _Spring_ configuration classes, one for testing and another for production.

```java
@Profile("dev")
@Configuration
@EnableDynamoDBRepositories(basePackages = "org.smartinrubio.springbootdynamodb.repository")
public class DynamoDBConfigDev {
    @Value("${amazon.dynamodb.endpoint}")
    private String amazonDynamoDBEndpoint;
    @Value("${amazon.dynamodb.region}")
    private String amazonDynamoDBRegion;
    @Bean
    public DynamoDBMapperConfig dynamoDBMapperConfig() {
        return DynamoDBMapperConfig.DEFAULT;
    }
    @Bean
    public DynamoDBMapper dynamoDBMapper(AmazonDynamoDB amazonDynamoDB, DynamoDBMapperConfig config) {
        return new DynamoDBMapper(amazonDynamoDB, config);
    }
    @Bean
    public AmazonDynamoDB amazonDynamoDB() {
        return AmazonDynamoDBClientBuilder
                .standard()
                .withEndpointConfiguration(
                        new AwsClientBuilder
                                .EndpointConfiguration(amazonDynamoDBEndpoint, amazonDynamoDBRegion))
                .build();
    }
    @Bean
    public DynamoDB dynamoDB() {
        return new DynamoDB(amazonDynamoDB());
    }
}
```

The first bean (`DynamoDBMapperConfig`) is created to override default behaviors so you can set things like types of loading data (`LAZY_LOADING`, `EAGER_LOADING`, `ITERATION_ONLY`), read configuration (`EVENTUAL` (default), `CONSISTENT`), or how the mapper should deal with attributes during save operations (`UPDATE`, `CLOBBER`). [More info](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.OptionalConfig.html).

**AmazonDynamoDB** bean configures the connection with your _DynamoDB_ host to perform query operations.

Lastly, _DynamoDB_ bean was created to provide another instance with more refined query operations.

Additionally, in case you want to point to the web version, you will need to set up:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_KEY`
- `AWSCredentials` bean
- Set credentials on `AmazonDynamoDB` bean.

```java
@Profile("prod")
@Configuration
@EnableDynamoDBRepositories(basePackages = "org.smartinrubio.springbootdynamodb.repository")
public class DynamoDBConfigProd {
    @Value("${amazon.dynamodb.accesskey}")
    private String amazonDynamoDBAccessKey;
    @Value("${amazon.dynamodb.secretkey}")
    private String amazonDynamoDBSecretKey;
    @Bean
    public AWSCredentials amazonAWSCredentials() {
        return new BasicAWSCredentials(amazonDynamoDBAccessKey, amazonDynamoDBSecretKey);
    }
    @Bean
    public DynamoDBMapperConfig dynamoDBMapperConfig() {
        return DynamoDBMapperConfig.DEFAULT;
    }
    @Bean
    public DynamoDBMapper dynamoDBMapper(AmazonDynamoDB amazonDynamoDB, DynamoDBMapperConfig config) {
        return new DynamoDBMapper(amazonDynamoDB, config);
    }
    @Bean
    public AmazonDynamoDB amazonDynamoDB() {
        return AmazonDynamoDBClientBuilder
                .standard()
                .withCredentials(amazonAWSCredentialsProvider())
                .withRegion(Regions.US_WEST_2)
                .build();
    }
    @Bean
    public DynamoDB dynamoDB() {
        return new DynamoDB(amazonDynamoDB());
    }
    public AWSCredentialsProvider amazonAWSCredentialsProvider() {
        return new AWSStaticCredentialsProvider(amazonAWSCredentials());
    }
}
```

### Repositories

Repositories need to be annotated with `@EnableScan` to use method name as queries.

```java
@EnableScan
public interface HotelRepository extends CrudRepository<Hotel, String> , CustomHotelRepository{
    List<Hotel> findAllByName(String name);
}
And as you can see on the previous code snippet, method query operations like findAllByName are allowed.

For custom queries we can create an interface to support the additional queries.

public interface CustomHotelRepository {
    void createTable();
    void loadData() throws IOException;
}
```

### Entities

`DynamoDBMapper` allows you to convert _DynamoDB_ items to _POJOs_, and generate table definitions.

```java
@Data
@DynamoDBTable(tableName = "Hotels")
public class Hotel {
    @DynamoDBHashKey
    @DynamoDBGeneratedUuid(DynamoDBAutoGenerateStrategy.CREATE)
    private String id;
    @DynamoDBAttribute
    private String name;
    @DynamoDBAttribute
    @DynamoDBTypeConverted(converter = GeoTypeConverter.class)
    private Geo geo;
}
```

For this particular example, the _Partition Key_ is the field annotated with `@DynamoDBHashKey`, which will be generated randomly.

In case you have inner objects, you will need to use `@DynamoDBTypeConverted` with a converter since this Spring module is not able to serialize objects by itself and only allows strings as values.

There are also other useful annotations such as `@DynamoDBIndexRangeKey` for secondary indexes on a _DynamoDB table_.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/spring-boot-dynamodb" text="Source Code" %}
</p>

## Conclusion

- **NoSQL does not mean non-relational**.
- **E/R diagrams are still valid**.
- **NoSQL databases do not replace RDBMS**.
- _NoSQL_ is good for **On-line Transaction Processing** (_OLTP_) or **Decision Support Systems** (_DSS_) at scale.
- _RDBMS_ is good for **On-line Analytical Processing** (_OLAP_) or _OLTP_ when scale is not important.
