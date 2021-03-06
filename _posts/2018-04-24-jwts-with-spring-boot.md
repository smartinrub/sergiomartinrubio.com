---
title: JWTs with Spring Boot
image: https://lh3.googleusercontent.com/v8KpwGmYimkQUGpSDKxFcsx7mc6DDXdUQtNyAVItxT8jnXHgo0Rtj2lXC-H0inMJkCyCeTZV0PS_QAiPL0pCciX0C6ilh3ZkMyXZ9hrRes1O-eWGOZxyY6VJr0MibwuJw-JY9Jq4gxMHWmifdoSd8haoUAs3G-tdgqrYndXZT5CAwcBtubxoPVb8QzyGKUus6uFBmL75VvKrHFxyONnCjWYS8VjBQAK8prnCIt1JNClkz1lw4hd0_RMN2UsDk2VU4Y_oLzI6IiMw2we1eeGePmZtNKPhSkW60iuieYsEsEqnGO2xd3W3wxEz3JClPjGC6SV-4_PVgfH3-vXycnTNS8_Ctz5K-na6ZcN2xgS6v5xqPUKe2wfGP3oqAcWeOUC0Rx2AtWRqBdBf2TM-UjjstL4Xw7FIfcxXqpxFp1c2SH8Odvun0DLlOTYdAhs34TQl8D_5P6JYTN400gOncjt0WCGbmVFWCgKxxdPb_JyYa631c2CoQTFIMn1881rzmrzKv964K9SQFBhEJVm3P4hu9T6oMeMZA94TwJudlBDvBmYXoHmqpGviS9qr1lmGoq0anVRwhncUv_TpVVPYDEqRLx0OOTDBL-ENRbgiQqTm38ZDOJxvhlyVaQ8gMHf_9SlRXR9EbfcyhIpc9lFglLBC_NtI0WP69XOQSww_g8cktJ840GtGi_xbiW1N8uMp=w1920-h1285-no?authuser=1
categories:
    - Java Framework
    - Spring
    - Security
mermaid: false
layout: post
---

## Introduction

[JWT](https://jwt.io/){:target="_blank"} (_JSON Web Token_) is an open source standard commonly used to transmit data between two services in a compact and secure way. This standard offers a wide range of libraries to generate _JWTs_ and includes libraries for platforms such as _.NET_, _Python_, _Node.js_, _Java_, _JavaScript_, _Perl_, _Ruby_, _Elixir_, _Golang_, _Groovy_, and _Haskell_.

## Features

* **Secure**: _JWTs_ are signed by using a secret or a **public/private key pair**.
* **Self-contained**: all information is stored inside the _JWT token_. No need for further database calls.
* **Unmodifiable information, but exposed**: Do not put secrets inside _JWTs_.
* **Compact format**: _JWTs_ are lightweight.

## How Does It Work?

{% include elements/figure.html image="https://lh3.googleusercontent.com/6C42ufo6FCCFM40a24XGl4SkJ7a4IDlM-8C5iGGTwfB2W7oj6JBSZPx9QeF4QZnba4TXQTdNZsldVeIkWQ=w800" caption="JWT Communication" %}

1. _JWTs_ can be added to the URL, sent via _POST_ as part of a parameter or added to the HTTP request header.
2. It contains a payload with information about the sender, issued date, expiration date...
3. _JWTs_ are usually generated during the authentication phase, and once the user is logged in, the _JWT_ previously generated allows the user to access to resources without having to make additional calls to databases.
4. They can be used to exchange information between services in a secure way, making sure that it was not modified during the transmission. This is possible because the signature is calculated using the header and payload.
5. The token is stored locally or in a _cookie_.
6. When the user wants to use the token, the convention is to add it to the header in the `Authorization` field using the `Bearer` prefix followed with the token value.

e.g.

```shell
Authorization: Bearer
```

### JWT format

**JWTs** are structured in three parts separated by dots:

- **Header**: contains the **token type** (_jwt_) and **hashing algorithm** like _SHA256_ or _RSA_.
- **Payload**: contains claims, which is the metadata (**subjects**, **senders**, or **expiration date**). There are three kinds of claims: **registered**, **public**, and **private**.
- **Signature**: the combination of **header**, **payload**, **secret**, and the **algorithm** specified in the header. The signature ensures that nothing has changed.

As a result, the format is `xxx.yyy.zzz`

### Java Integration

The `jjwt` library provides all you need to generate signed tokens in a _Java_ application.

**Maven dependency**:

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
```

**Generate JWT Token**:

```java
String jwt = Jwts.builder()
    .setClaims(claims)
    .setSubject(email)
    .setIssuedAt(new Date())
    .setExpiration(new Date(System.currentTimeMillis()+ JWT_EXPIRATION_TIME))
    .signWith(SignatureAlgorithm.HS256, JWT_SECRET)
    .compact();
```

- **Issuer** (iss): `getIssuer()` and `setIssuer(String)`
- **Subject** (sub): `getSubject()` and `setSubject(String)`
- **Audience** (aud): `getAudience()` and `setAudience(String)`
- **Expiration** (exp): `getExpiration()` and `setExpiration(Date)`
- **Not Before** (nbf): `getNotBefore()` and `setNotBefore(Date)`
- **Issued At** (iat): `getIssuedAt()` and `setIssuedAt(Date)`
- **JWT ID** (jti): `getId()` and `setId(String)`

All these parameters are available through **Claims** getters.

**Verify JWT Token**:

```java
final Claims claims = Jwts.parser()
    .setSigningKey(JWT_SECRET)
    .parseClaimsJws(compactJws)
    .getBody();
```

- `JWT_SECRET` is the secret used when we generate the token.
- `compactJws` is the _JWT token_ pass in the header.

If the verfication succeed we will be able to store the metadata in a claims variable that is available to consume later. On the other hand, If the verification fails `SignatureException` is thrown.

### Troubleshooting

In case you are using **Java 9+**, make sure you add an explicit dependency to the _POM file_.

```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
</dependency>
```

`jaxb-api` is missing in _Java 9_, and until **jjwt** adds the dependency or removes the usage of _JAXB classes_, we have to add it manually.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/jwt-example" text="Source Code" %}
</p>
