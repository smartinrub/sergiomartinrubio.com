---
title: Working with JDBC API
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Database
    - Java Framework
mermaid: false
layout: post
---

## Introduction

[JDBC](https://docs.oracle.com/javase/tutorial/jdbc/basics/index.html){:target="_blank"} is a [SQL](http://sergiomartinrubio.com/articles/mysql-guide){:target="_blank"} (Structured Query Language) API that was release for the first time in 1996 and since then is one of the most commonly use **Java** libraries. The purpose of this library is to allow programmers to use standard SQL statements in **Java** language.

> SQL is usually pronounced as sequel and you can learn how to use this language [here](http://sergiomartinrubio.com/articles/mysql-guide){:target="_blank"}.

## Design

The **JDBC architecture** is based on a driver manager which allows third-party drivers to connect to specific databases providers, so programmers only need to learn how to use the **JDBC API**.

## JDBC Configuration

1. Choose _JDBC_ driver: _MySQL_, _PostgreSQL_, _IBM DB2_, _Microsoft SQL Server_... You need to get the _JAR_ file in which the driver for your database is located. For example, _JDBC_ Type 4 driver for MySQL can be found in the [Maven Repository](https://mvnrepository.com/artifact/mysql/mysql-connector-java){:target="_blank"}
2. Create database.
3. Create database connection. Syntax:

MySQL Connector/J Database URL

```
jdbc:mysql://[host][,failoverhost...]
    [:port]/[database]
    [?propertyName1][=propertyValue1]
    [&propertyName2][=propertyValue2]...
```

Java DB Database Connection URLs

```
jdbc:[driver]:[subsubprotocol:][databaseName]
    [;attribute=value]*
```

e.g.

```
jdbc:mysql://localhost:3306/myDB?connectTimeout=5000&socketTimeout=30000
```

```java
private static Connection getConnection() throws SQLException {
    String url = "jdbc:mysql://127.0.0.1:3306/test";
    String username = "root";
    String password = "";
    return DriverManager.getConnection(url, username, password);
}
```

4. Execute _SQL_ statements:

```java
public static void main(String[] args) throws SQLException {
    try (Connection connection = getConnection();
            Statement statement = connection.createStatement()) {

        // Creates SQL table
        statement.executeUpdate("CREATE TABLE user(" +
                "email VARCHAR(30) NOT NULL," +
                "name VARCHAR(20) NOT NULL," +
                "age INTEGER," +
                "date_time TIMESTAMP" +
                ")ENGINE=InnoDB;");

        // Creates record in user table
        statement.executeUpdate("INSERT INTO user(email, name, age) " +
                "VALUES('econsergio@gmail.com', 'Sergio', 29)");

        // Query all records from user table
        try (ResultSet result = statement.executeQuery("SELECT * FROM user")) {
            while (result.next()) {
                System.out.println(result.getString(1));
                System.out.println(result.getString(2));
                System.out.println(result.getString(3));
                System.out.println(result.getString(4));
            }
        }

        // Deletes user table
        statement.executeUpdate("DROP TABLE user");
    }
}
```

## JDBC Statements

### Statements

In this section you will see how to use the _JDBC_ `Statement` to run _SQL_ commands. As you noticed in the previous code snippet we used the `DriverManager` connection to create a `Statement`.

```java
Statement statement = connection.createStatement()
```

### Prepared Statements

When your queries involve variables you should use prepared statements so you do not have to worry about quotes and you can avoid injection attacks. Use `PrepartedStatement` class to bind variables to actual values with a `set` method. 

e.g.

```java
String selectByEmail = "SELECT * FROM user WHERE email = ?";

try (Connection connection = getConnection();
        PreparedStatement preparedStatement = connection.prepareStatement(selectByEmail)) {

    ...

    // Set a string to a user email
    preparedStatement.setString(1, "econsergio@gmail.com");

    ...
}
```

Multiple `Statement` objects can be created at the same time so we can run statements concurrently, but remember to check how many statements are allowed by the chosen database vendor.

> IMPORTANT: `Connection`, `Statement` and `ResultSet` should be closed at the end. You can use a **try-with-resources** statement to automatically close the resources for you.

### Execute Queries

You can call `statement.executeUpdate()` or `statement.executeQuery()` to execute SQL statements. The first method is used to run statements like `INSERT`, `UPDATE`, `DELETE`, `CREATE TABLE` or `DROP TABLE`, whereas `executeQuery()` is used to run `SELECT` queries.

`executeQuery()` can be used with a number or a string argument. With a number it will return a column by index, on the other hand when a string argument is provided it returns a column by name.

> Remember that database column numbers start at 1.

Another alternative is `execute`. This returns `true` if the first object that the query returns is a `ResultSet` object. You can use this method when you expect one or more `ResultSet`.

```java
// Query records from user table by email
try (ResultSet result = preparedStatement.executeQuery()) {
    while (result.next()) {
        System.out.println(result.getString("email"));
        System.out.println(result.getString("name"));
        System.out.println(result.getInt("age"));
        System.out.println(result.getString("date_time"));
    }
}
```

### Process Result

`ResultSet` contains the result of the query and you can iterate through it. The pointer is positioned before the first row so `next()` is required to be called the first time to get the first row. Every time you call `next()` a row is returned, so when multiple rows are expected we can iterate through all of them by calling `next()` until it returns `false`.

## SQL Exceptions

Each `SQLException` contains another `SQLException` that can be retrieved with `getNextException()`. You can iterate through the exception with a for each loop as follows:

```java
catch (SQLException sqlException) {
    System.out.println(sqlException.getSQLState());
    System.out.println(sqlException.getErrorCode());

    for (Throwable throwable : sqlException) {
        System.out.println(throwable.getMessage());
    }
}
```

You can also retrieve the _SQL State_ and _Error Code_ with `getSQLState()` and `getErrorCode()`. In addition to exceptions the database driver also provides warnings.

```java
SQLWarning warning = statement.getWarnings();
while (warning != null) {
    System.out.println(warning.getNextWarning());
}
```

## Transactions

To run transactions first we need to disable the auto-commit mode.

```java
connection.setAutoCommit(false);
// Creates record in user table
statement.executeUpdate("INSERT INTO user(email, name, age) " +
        "VALUES('econsergio@gmail.com', 'Sergio', 29)");
// Creates another record in user table
statement.executeUpdate("INSERT INTO user(email, name, age) " +
        "VALUES('jose@gmail.com', 'Jose', 61)");
connection.commit();
```

In the previous example both `INSERT` statements are executed as a single transaction.

You can also roll back transactions with `connection.rollBack()`, or even roll back to a particular point if you previously created a save point.

```java
Savepoint savePoint = connection.setSavepoint();

...

connection.rollback(savePoint);
```

> It's recommended to set auto commit back to true (`connetion.setAutoCommit(true)`) when it's not needed.

> When you do not need a save point anymore it's recommended to release it `connection.releaseSavepoint(savePoint)`

### Batch Statements

When you are planning to run many `INSERT`, `UPDATE` or `DELETE` statements at the same time you can create batch actions.

You should create batch actions as a single transaction, so in case something goes wrong you can start again from the beginning.

```java
// Turn off auto commit to create a single transaction
connection.setAutoCommit(false);

String createTable = "CREATE TABLE user(" +
        "email VARCHAR(30) NOT NULL PRIMARY KEY," +
        "name VARCHAR(20) NOT NULL," +
        "age INTEGER," +
        "date_time TIMESTAMP" +
        ")ENGINE=InnoDB;";

// Add create table statement to batch
statement.addBatch(createTable);

String firstInsert = "INSERT INTO user(email, name, age) " +
        "VALUES('econsergio@gmail.com', 'Sergio', 29)";
String secondInsert = "INSERT INTO user(email, name, age) " +
        "VALUES('jose@gmail.com', 'Jose', 61)";

// Add insert statements to batch
statement.addBatch(firstInsert);
statement.addBatch(secondInsert);

// Commit batch
int[] counts = statement.executeBatch();
connection.commit();
connection.setAutoCommit(false);
```

## Scrollable & Updatable Result Sets

By default, result sets are not scrollable or updatable. You can create scrollable and updatable result sets by setting the type and concurrency when you create the _statement_.

```java
Statement statement = connection.createStatement(
    ResultSet.TYPE_SCROLL_SENSITIVE,
    ResultSet.CONCUR_UPDATABLE
)
```

| Type Value                | Description                                                     |
| ------------------------- | --------------------------------------------------------------- |
| `TYPE_FORWARD_ONLY`       | `ResultSet` is not scrollable                                   |
| `TYPE_SCROLL_INSENSITIVE` | `ResultSet` is scrollable but not sensitive to database changes |
| `TYPE_SCROLL_SENSITIVE`   | `ResultSet` is scrollable and sensitive to database changes     |

| Concurrency Value  | Description                                       |
| ------------------ | ------------------------------------------------- |
| `CONCUR_READ_ONLY` | `ResultSet` cannot be used to update the database |
| `CONCUR_UPDATABLE` | `ResultSet` can be used to update the database    |

Move forward, backward and to a absolute or relative position:

```java
try (ResultSet result = statement.executeQuery("SELECT * FROM user")) {
    result.next();
    System.out.println("Row: " + result.getRow());
    System.out.println(result.getString(1));
    System.out.println(result.getString(2));
    System.out.println(result.getInt(3));
    System.out.println(result.getString(4));
    System.out.println();

    result.next();
    System.out.println("Row: " + result.getRow());
    System.out.println(result.getString(1));
    System.out.println(result.getString(2));
    System.out.println(result.getInt(3));
    System.out.println(result.getString(4));
    System.out.println();

    // return to previous row
    result.previous();
    System.out.println("Row: " + result.getRow());
    System.out.println(result.getString(1));
    System.out.println(result.getString(2));
    System.out.println(result.getInt(3));
    System.out.println(result.getString(4));
    System.out.println();

    // go to second row
    result.absolute(2);
    System.out.println("Row: " + result.getRow());
    System.out.println(result.getString(1));
    System.out.println(result.getString(2));
    System.out.println(result.getInt(3));
    System.out.println(result.getString(4));
    System.out.println();

    // go one row back
    result.relative(-1);
    System.out.println("Row: " + result.getRow());
    System.out.println(result.getString(1));
    System.out.println(result.getString(2));
    System.out.println(result.getInt(3));
    System.out.println(result.getString(4));
    System.out.println();
}
```

Update a particular row:

```java
try (ResultSet result = statement.executeQuery("SELECT * FROM user")) {
    result.next();
    System.out.println(result.getString(1));
    System.out.println(result.getString(2));
    System.out.println(result.getInt(3));
    System.out.println(result.getString(4));

    // Update row 1 result
    System.out.println("Updating row " + result.getRow());
    System.out.println();
    result.updateString("email", "luis@gmail.com");
    result.updateString("name", "Luis");
    result.updateInt("age", 25);
    result.updateRow();
}
```

> Make sure to call `updateRow()` after updating columns.

You can also add a new row to the database with `result.moveToInsertRow()` to move the cursor to a particular position and then call `result.insertRow()`; delete a row with `result.deleteRow()`.

## Row Sets

A `RowSet` allows you to close a database connection and come back later or use in a different part of your application so it is more flexible and easier to use than a result set.

The **JDBC API** provides the following implementations:

- `JdbcRowSet`
- `CachedRowSet`
- `WebRowSet`
- `JoinRowSet`
- `FilteredRowSet`

For example, you can populate a `CachedRowSet` from a result set:

```java
ResultSet result = ...;
RowSetFactory factory = RowSetProvider.newFactory();
CachedRowSet cachedRowSet = factory.createCachedRowSet();
cachedRowSet.populate(result);
// now we can close the connection
connection.close();

// Sets query statement
cachedRowSet.setCommand("SELECT * FROM user");

// This creates a databaase connection, runs the query,
// populates the row set and disconnects
cachedRowSet.execute();
```

> If the result is very large you can use pagination with `cachedRowSet.setPageSize(10)` and `cachedRowSet.nextPage()`.

## Metadata

You can retrieve database metadata with:

```java
DatabaseMetaData metaData = connection.getMetaData();
```

and then you can call `getCatalogs()`, `getTables()`, `getProcedures()`, `getMaxConnections()`...

e.g.

```java
DatabaseMetaData metaData = connection.getMetaData();
ResultSet resultSet = metaData.getTables(null, null, null, new String[]{"TABLE"});
while (resultSet.next()) {
    // This returns all table names
    System.out.println(resultSet.getString(3));
}
```

## Enterprise Applications

The **JDBC** configuration used previously does not scale up in enterprise applications, since you keep opening and closing database connections and this is costly, that is why web containers solutions like _Java EE_ or _Spring_ provide a connection pool which allows you to acquire a connection from a source of pooled connections and when you are done with the connection you simply return the connection to the pool.

Moreover, the configuration of database connections is usually integrated with the [JNDI (Java Naming and Directory Interface)](https://sergiomartinrubio.com/articles/jndi-overview), so you keep the properties of the database centralized in a property file. In this kind of environment the JNDI is responsible for getting the database connection (data source) instead of talking to the `DriverManager` directly.

```java
Context jndi = new InitialContext();
DataSource dataSource = (DataSource) jndi.lookup("java:comp/env/jdbc/test");
Connection connection = dataSource.getConnection();
```

> _JNDI_ is an API specified in Java technology that provides naming and directory functionality to applications written in the Java. _JNDI_ provides methods for performing standard directory operations, such as associating attributes with objects and searching for objects using their attributes.

In a web container provider like **Spring** you might have something like this:

```java
@Configuration
public class DbConfig {

    @Bean
    public DataSource getDataSource()
    {
        DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.driverClassName("com.mysql.jdbc.Driver");
        dataSourceBuilder.url("jdbc:mysql://localhost:3306/test");
        dataSourceBuilder.username("root");
        dataSourceBuilder.password("");
        return dataSourceBuilder.build();
    }
}
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-jdbc.git" text="Examples" %}
</p>
