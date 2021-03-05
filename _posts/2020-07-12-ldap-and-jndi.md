---
title: LDAP and JNDI
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java Framework
    - Data Store
mermaid: false
layout: post
---

## Introduction

[LDAP](https://ldapwiki.com/wiki/LDAP){:target="_blank"} (Lightweight Directory Access Protocol) is a directory service in which you can store data. The information stored in LDAP is structured in a tree hierarchy and each directory object is called an entry. An entry contains the following components:

- `DN` (_Distinguished Name_): identifies an entry and consists of single entries names separated by a comma and ordered right-to-left.
- Object classes: An object class is the schema of an entry that organizes and determines the content of the entry.
- Attributes: contain the actual data.

Entry's attributes are created like this:

```
<attributeName>=<attributeValue>
```

e.g.

```
cn=Sergio
```

## LDAP Server

1. [Download Apache Directory Studio](http://directory.apache.org){:target="_blank"}

    >Apache Directory Studio association 2.0.0-M15 requires Java 8. Make sure you do not have any other _Java_ version under `/Library/Java/JavaVirtualMachines` on MacOS.

2. Go to _LDAP Servers_ tab in the bottom left corner and click on _New LDAP Server_.
3. Select latest version and click on _Finish_.
4. Create a connection by right clicking on the new LDAP server.
5. Start up the server.
6. Now we can go to the _LDAP Browser_ tab.
7. Import some data

{%- gist bf2cd826ea1a252bf9bec34da34b60e2 %}

## JNDI Integration

LDAP is one of the service providers that comes with JDK out-of-the-box, so you can use the [JNDI API](https://sergiomartinrubio.com/articles/jndi-overview) to perform operations on LDAP server.

### Getting Started

You only need the `javaee-api` dependency to starting using LDAP with JNDI.

```xml
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>8.0.1</version>
    <scope>provided</scope>
</dependency>
```

### LDAP Context

The LDAP context can be created as follows:

```java
Properties properties = new Properties();
properties.setProperty(INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
properties.setProperty(PROVIDER_URL, "ldap://localhost:10389/dc=example,dc=com");
DirContext context = new InitialDirContext(properties);
```

>The port of LDAP server can be found in _Apache Directory Studio_.

### Organizational Units

You can see all the organizational units with:

```java
Properties properties = new Properties();
properties.setProperty(INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
properties.setProperty(PROVIDER_URL, "ldap://localhost:10389/dc=example,dc=com");
DirContext context = new InitialDirContext(properties);

NamingEnumeration<Binding> bindings = context.listBindings("");
while (bindings.hasMoreElements()) {
    System.out.println(bindings.next().getName());
}
```

### Attributes

To access the entries in a organization unit:

```java
Properties properties = new Properties();
properties.setProperty(INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
properties.setProperty(PROVIDER_URL, "ldap://localhost:10389/dc=example,dc=com");
DirContext context = new InitialDirContext(properties);

DirContext peopleContext = (DirContext) context.lookup("ou=people");

NamingEnumeration<Binding> people = peopleContext.listBindings("");
while (people.hasMoreElements()) {
    String bindingName = people.next().getName();
    Attributes personAttributes = peopleContext.getAttributes(bindingName);
    Attribute description = personAttributes.get("description");
    Attribute mailsAttribute = personAttributes.get("mail");
    Attribute personName = personAttributes.get("cn");

    NamingEnumeration<?> personNames = personName.getAll();
    System.out.println("Person names:");
    while (personNames.hasMoreElements()) {
        System.out.println(personNames.next());
    }
    System.out.println();

    System.out.printf("Description: %s\n\n", description.get());

    NamingEnumeration<?> mails = mailsAttribute.getAll();
    System.out.println("Mails:");
    while (mails.hasMoreElements()) {
        System.out.println(mails.next());
    }
}
```

This will print out the following given the example ldif file:

```
Person names:
Robert Smith
Robert J Smith
bob  smith

Description: swell guy

Mails:
r.smith@example.com
rsmith@example.com
bob.smith@example.com
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/ldap-jndi-example.git" text="Examples" %}
</p>
