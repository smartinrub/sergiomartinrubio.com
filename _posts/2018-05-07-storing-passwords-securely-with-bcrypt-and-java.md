---
title: Storing Passwords Securely with Bcrypt and Java
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java Framework
    - Spring
    - Security
mermaid: false
layout: post
---

## Introduction

Storing passwords is a difficult task when we need to satisfy all the data protection laws, and it is getting even tougher with the rise of [GDPR](https://eugdpr.org/){:target="_blank"}, the new European regulation in data privacy. Therefore, we have to make sure all your sensitive data is encrypted and, to do that, we can use hashing algorithms.

When we want to hash passwords to store them in a database, Bcrypt is the way to go and there are many libraries for different languages.

## Java Implementation

If you language of choice is Java you can use BCrypt with the library included in the **Spring Security** module.

```xml
<dependency>      
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
</dependency>
```

**JBcrypt** hashes passwords using a version of _Bruce Schneier's Blowfish_ block cipher with modifications designed to raise the cost of off-line password cracking. The computation cost of the algorithm is parameterized, so it can be increased as computers get faster.

Operations:

1. Hash password:

    ```java
    BCrypt.hashpw(plainTextPassword, BCrypt.gensalt())
    ```

    where `plainTextPassword` is the password we want to hash and `BCrypt.gensalt()` is a salt to autogenerate every time.

    In case we want to increment the complexity, an optional parameter (`log_rounds`) has to be provided to `BCrypt.gensalt()`, which determines the computational complexity of the hashing. `log_rounds` is exponential ($$2^{log\_rounds}$$) and it specifies how many times to run the internal hash function. The default value is 10, and the valid values are between 4 and 31.

    The output strings after running `hashpw()` will look like:

    `$bcrypt_id$log_rounds$128_bits_salt184_bits_hash`

    `hashpw()` is smart enough to extract the salt from the string, so you do not need to worry about the salt value anymore, only the hashed string.

2. Check hashed password: 

    ```java
    BCrypt.checkpw(unencryptedPassword, hashPassword)
    ```

    This method checks that an unencrypted password matches the one that was previously hashed and ensures that it is stored somewhere.

## Conclusion

**Bcrypt** is 10,000 times slower than _sha1_ to run. If we have a machine that is able to run it in 100ms, this is probably fast enough for login, but it might be too slow if we want to execute _Bcrypt_ against a long list of passwords. In consequence, if a hacker wants to run _Bcrypt_ a billion times by using the same computational power, it will take 27,777 hours.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/bcrypt-service" text="Source Code" %}
</p>
