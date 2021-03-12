---
title: JPA. Introduction to Java Persistence API
image: https://lh3.googleusercontent.com/pw/ACtC-3dnwtfNw9yvzqlceXcVXKxvtRR-p0lN-YGonkCRcfWp2YZCt1VAB_34Y4ooSSvupCDFyo8FTf6Vx-tMuqKWbZHNySei8s-iSqU_GEl2dPV0WczUyQBUF8xd-7C05HTKcyxEgzSkudO3DnNmk1MCJh4h=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java Framework
    - Database
mermaid: false
layout: post
---

The **Java Persistence API** (*JPA*) is responsible for performing CRUD operations and is built on top of *Hibernate*. JPA has been designed to replace EJB 2.1 entity beans and has started as a part of the EJB 3 specification. JPA is now outside of an [EJB](https://sergiomartinrubio.com/articles/ejb-what-it-is-why-it-exists-and-how-it-works) container and has its own specification, but it's still part of the [EJB specification](https://sergiomartinrubio.com/articles/ejb-what-it-is-why-it-exists-and-how-it-works), since a compliant EJB 3 container has to provide a JPA implementation, which integrates into the transaction handling of the container.

## Entities

Entities are used in data query methods to perform *CRUD* (Create, Retrieve, Update and Delete) operations. 

Entities are basically POJO classes that represent a single table or multiple tables in a database and each entity instance represents a single row in a table.

You can define an entity as follows:

```java
@Entity
@IdClass(FooPK.class)
@Table(name = "FOO")
public class Foo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Id
    @Basic(fetch = FetchType.EAGER)
    private String name;

    // getters and setters
    // equals, hashCode, toString...
}
```

Entity class requirements:

- The `@Entity` annotation is required to identify the POJO class as an entity, so the container will check other annotations on the class and will enable other query features. 
- Define a primary key field using the `@Id` annotation, so the container knows the primary key of the entity.
- Public or protected no-arguments constructor.
- The class, methods or fields cannot be declared `final`.
- Entity instance variable cannot be public.
- If the entity is passed to another EJB container, it must implement `Serializable` interface.

Other entity annotations are:

- `@Table`: The table name defaults to the name class unless `@Table` is used with a given name.
- `@Column`: Column names default to field names declared in the entity class unless `@Column` is used with a given name. This annotation also support other attributes like `unique`, `nullable`, `insertable` or `updatable`.
- `@Basic`: This annotation is used to mark a field as a basic type so Hibernate will use the standard mapping. The `fetch` property can have `FetchType.EAGER` or `FetchType.LAZY` (use with expensive fields).
- `@Transient`: This annotation is used to indicate that a field should not be persisted.
- `@GeneratedValue`: This annotation is used to auto generate a primary key value with a specified generator strategy (`TABLE`, `SEQUENCE`, `IDENTITY` or `AUTO`).
- `@IdClass`: A composite primary key is also possible by annotating each field in the composite key with `@Id` and defining a primary key class that has those same fields and is passed as attribute in `@IdClass`.
- `@EmbeddedId` and `@Embeddable`: These two annotations are an alternative to `@IdClass`. It requires that you create a class that is annotated with `@Embeddable` and a field in the entity of that class that is annotated with `@EmbeddedId`. 
- `@Version`: This annotation is used to guarantee optimistic locking. Every time an entity is updated in the database the version field will be increased by one and if the same entity has already been updated by another thread, then the persistence provider will throw an `OptimisticLockException`.

## Entity Relationships

### One to One Relationship

You can have a bidirectional one to one relationship between two tables with `@OneToOne` and `@JoinColumn` annotations.

On one side we will have:

```java
@Entity
@Table(name = "BAR")
public class Bar {
    
    @Id
    @GeneratedValue
    @Column(name = "id")
    private Long id;
    
    // other fields

    // fetch strategy is eager by default
    @OneToOne
    @JoinColumn(name = "bar_child_id", referencedColumnName = "id")
    private BarChild barChild;
    
    // getters, setters...
}
```

where `name = bar_child_id` matches the foreign key column name on the `BAR` table; `referencedColumnName = id` matches the `BAR_CHILD` column name. 

On the other side we will have:

```java
@Entity
@Table(name = "BAR_CHILD")
public class BarChild {
    
    @Id
    @GeneratedValue
    @Column(name = "id")
    private Long id;

    // other fields

    @OneToOne(mappedBy = "barChild")
    private Bar bar;
}
```

### One to Many and Many to One Relationships

We will use the `@OneToMany` annotation on a collection pointing to another class that does not have a relationship field, or it has a field that is not a collection and is annotated with `@ManyToOne` 

```java
@Entity
@Table(name = "FOO")
public class Foo {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    // other attributes
    
    // fetch strategy is eager by default
    @ManyToOne
    @JoinColumn(name = "bar_id", nullable = false)
    private Bar bar;
    
    // getters, setters...
}
```

where `name = "bar_id"` matches the column name of the foreign key in the database.

```java
@Entity
@Table(name = "BAR")
public class Bar {
    
    @Id
    @GeneratedValue
    @Column(name = "id")
    private Long id;
    
    // other attributes
    
    // fetch strategy is lazy by default
    @OneToMany(mappedBy = "bar")
    private List<Foo> foos;
    
    // getters, setters...
}
```

### Many to Many Relationship

A many-to-many relationship is joining two collections from two different entities. This requires a `@ManyToMany` annotation in both sides.

On one side we will have:

```java
@Entity
@Table(name = "FOO")
public class Foo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;
    
    // other attributes

    // lazy fetch strategy by default
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "foo_child",
            joinColumns = @JoinColumn(name = "id"),
            inverseJoinColumns = @JoinColumn(name = "foo_id"))
    private List<FooChild> fooChildren;
    
    // getters, setters...

}
```

`Foo` will own the associations.  `"foo_child"` is the join table name; `@JoinColumn(name = "id")` is the foreign key column of the `Foo` table; `@JoinColumn(name = "foo_id")` is the foreign key column of the `FooChild` table which is referencing the id column  in the `Foo` table .

On the other side we will have:

```java
@Entity
@Table(name = "FOO_CHILD")
public class FooChild {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", unique = true, nullable = false)
    private Long id;
    
    // other attributes

    // lazy fetch strategy by default
    @ManyToMany(mappedBy = "fooChildren", fetch = FetchType.EAGER)
    private List<Foo> foos;
    
    // getters, setters...
}
```

### Field Bindings

`@OneToMany` and `@ManyToMany` has `FetchType.LAZY` as the default value for the annotation attribute `fetch`, however, `@ManyToOne`, `@OneToOne` use an eager strategy as default,  `FetchType.EAGER`. When an eager strategy is used it could trigger an *N+1 query issue*.

The *N+1 query issue* could also happen even with `FetchType.LAZY` if you do something like:

```java
for(Foo foo: foos) {
	System.out.println(foo.getFooChild().getName());
}
```

You can use a `JOIN FETCH` clause to fix the N+1 query issue.

> The *JOIN FETCH* clause is JPA-specific. It tells the persistence provider to not only join two tables within a single query but to also initialize the association on the returned entity, so you will not need to do new queries and hit the database to retrieve the associated entities.

### Cascade types

JPA provides cascade types to propagate an operation from a parent to a child entity.

- `ALL`: propagates all operations
- `PERSIST`: children entities are also saved
- `MERGE`: updates children entities
- `REMOVE`: children entities associated to the same id are also removed
- `REFRESH`: removes both the parent and child entity from the persistence context
- `DETACH`: children entities get reloaded when the parent entity is refreshed

## Persistence Unit

A persistence unit is the configuration required by JPA and allows you to instantiate an entity manager. Persistence units are declared in a file `META-INF/persistence.xml` where you can configure things like the name of each persistence unit, managed classes included in your persistence unit, how classes are mapped to database tables, the datasource use to connect to the database... however, you can also configure a persistence unit programatically by implementing the `PersistenceUnitInfo` interface.

There are two transaction types:

- `RESOURCE_LOCAL`: you are responsible for managing the `EntityManger` lifespan. You need to take care of beginning and ending the transaction logic.
- `JTA`: the transactions are managed by the application server. It requires the application server provides support. This limits the flexibility over transactions.

You can define a container-managed persistence unit as follows:

```xml
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" version="2.2"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="MyPersistenceUnit" transaction-type="JTA">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <jta-data-source>jdbc/jpa_example</jta-data-source>
        <class>com.sergiomartinrubio.jpa.javaee8.model.Foo</class>
        <class>com.sergiomartinrubio.jpa.javaee8.model.FooChild</class>
        <class>com.sergiomartinrubio.jpa.javaee8.model.Bar</class>
        <class>com.sergiomartinrubio.jpa.javaee8.model.BarChild</class>
        <exclude-unlisted-classes>true</exclude-unlisted-classes>
        <properties>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="show_sql" value="true"/>
            <property name="hibernate.transaction.jta.platform"
                      value="org.hibernate.service.jta.platform.internal.SunOneJtaPlatform"/>
        </properties>
    </persistence-unit>
</persistence>
```

You need to select the `jta-data-source` by using the JNDI name defined in the web server.

>  `hibernate.transaction.jta.platform` might be required to interact with a JTA system. If you use Payara as the web server you will have to set it to `SunOneJtaPlatform`. [Other JtaPlatform implementations can be found here](https://docs.jboss.org/hibernate/orm/5.1/userguide/html_single/chapters/transactions/Transactions.html).

## Managing Entities

The `EntityManager` instance is responsible for managing entities and provide support for CRUD operations. Each `EntityManager` instance is associated with a *persistence-context*. The *persistence-context* holds the fetched data and acts as a cache. You will usually have a *persistence-context* per transaction. When the data is modified in the *persistence-context* the changes will be saved in the database either manually or automatically with `COMMIT` or `AUTO` flush strategies.

You can get an instance of the `EntityManager` from the EJB container, from outside a the EJB container or via a JNDI lookup.

An EntityManager can be used like this:

```java
@Stateless
public class FooRepository {

    @PersistenceContext(unitName = "MyPersistenceUnit")
    private EntityManager entityManager;
    
    public List<Foo> findAll() {
    return entityManager.createNamedQuery("Foo.findAll", Foo.class)
                .getResultList();
    }

    public void save(Foo foo) {
		entityManager.persist(foo);
    }
}
```

In the previous example we got an instance of the `EntityManger` that is bound to the `MyPersistenceUnit` unit.

## Entity States

Entities go through different states:

- **New**: This happens the a new entity instance is created with the default constructor.
- **Managed**: This happens when the entity is passed to the `EntityManager.persist()` , so the entity is made persistent and added to the persistence context.
- **Detached**: When the entity is removed from the database, the persistence context disappears or is passed as a value to a client, then the entity is not in a managed state anymore and it becomes a detached entity instance.
- **Removed**: When the `EntityManager.remove()` method is called the entity goes to a removed state.

## JPA Queries

JPA provides three types of queries:

1. **Java Persistence Query Language (JPQL)**
2. **Criteria API Query**
3. **Native Query**

|                 | JPQL                   | Criteria API Query | Native Query      |
| --------------- | ---------------------- | ------------------ | ----------------- |
| **Readability** | Similiar to SQL syntax | Requires objects   | SQL syntax        |
| **Typesafe**    | No                     | Yes                | No                |
| **Performance** | Worst                  | Intermidiate       | Best              |
| **Verbosity**   | Intermidiate           | Worst              | Best              |
| **Portability** | Portable               | Portable           | Not Portable      |
| **Versatility** | Restricted             | Restricted         | Full DB potential |



### JPQL

JPQL queries are invoked by the `EntityManager` on the persistence context. There are two types of JPQL queries:

- `TypedQuery`: We use the `EntityManger` to execute the queries.

```java
TypedQuery<Foo> typedQuery = entityManager
        .createQuery("SELECT f FROM Foo f WHERE f.id = :id", Foo.class);
```

- `NamedQuery`: We can define named queries on specific methods or in the entity classes itself.

  Declare `NamedQuery`:

```java
@Entity
@Table(name = "FOO")
@NamedQuery(name = "Foo.findAll", query = "SELECT f FROM Foo f")
public class Foo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;
}
```

​		Invoke `NamedQuery`:

```java
Query namedQuery = entityManager.createNamedQuery("Foo.findAll");
```

​		You can declare multiple named queries with `@NamedQueries` annotation.

### Criteria API Query

Criteria API queries can be complex and difficult to understand but give you some safety advantages.

```java
CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
CriteriaQuery<Foo> criteriaQuery = criteriaBuilder.createQuery(Foo.class); // create query object
Root<Foo> from = criteriaQuery.from(Foo.class); // get query root

CriteriaQuery<Foo> select = criteriaQuery.select(from); // set result type
TypedQuery<Foo> typedQuery = entityManager.createQuery(select); // prepare query for execution
List<Foo> foos = typedQuery.getResultList(); // execute query
```

### Native Query

```java
Query nativeQuery = entityManager
        .createNativeQuery("SELECT * FROM foo WHERE id=:id", Foo.class);
nativeQuery.setParameter("id", "1");
```

### Binding Parameters

You can bind parameters with `:` followed by the parameter name or `?` followed by an index.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/jpa-javaee8.git" text="Examples" %}
</p>

Image by <a href="https://pixabay.com/users/pexels-2286921/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1850170">Pexels</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=1850170">Pixabay</a>
