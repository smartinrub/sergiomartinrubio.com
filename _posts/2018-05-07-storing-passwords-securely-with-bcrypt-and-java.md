---
title: Storing Passwords Securely with Bcrypt and Java
image: https://lh3.googleusercontent.com/I6K7F4_3ExltrjpiIU-r4LYbAqHL-m4zrPOmJ_AlR-NhGqKBe6eXi2V-e620bY7lEy5fVzQEuCaF37PqW9l4wh6L65ViAIHLOVfLC3q0aLkm2AlTpt4Kgt9AaxsXW85T8U2ZhWletsOzpzEqki-57m0xQE55l9oUOdh9oiSSGjH3fJBhfi1MTqQLTsJOLZLxKOEMhPf1Kd0IujbscPRhLakP-zO7E5ZNpI48eRyPpZXu9YOTzmJ9w0l2j5Ln4MMri1UwXG4pkXa816leOjn9eruPVpHkcCsh0PWHsHKZSG_DzT-PGf0I_r7hIbLylwclR5BfvQivuZ7ZmZGQtI1NaPZyIhdl_4aqR_dfSWfhc5p13amvejB3jVb8bdiP-UAOatbtU0mx8mrLbMrT5dOu-jX_64JrEWOvm2_ir8gjv6-Va0NjtAG1J1NeIxTk0qubV9cPFTwE6TLemm2oMaUQwBS-S73mrhzzpqqOearFT94BeFTYzw0p1SmsaYr5crpEnZRtQ9sEO06CiWzO5EWNiGCLtvpYMfO8pdawggvWuVgx7eJFZYAmb3aQTPdwTV9-46FsFPvdYUmrcMLEkQRpPRbO96lEP0rsnYlEM4xrtz98mxVL2JUimL_8d8cGlCC1t6qoeIqj5Zf8gww6jru5F-dcJBZoPPMHlz0OOOxndmxydHbU-wwtE2xFeWAG=w411-h288-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Java Framework
    - Spring
    - Security
mermaid: false
layout: post
---

Storing passwords is a difficult task when we need to satisfy all the data protection laws, and it is getting even tougher with the rise of [GDPR](https://eugdpr.org/){:target="_blank"}, the new European regulation in data privacy. Therefore, we have to make sure all our sensitive data is protected, and for passwords specifically that means hashing rather than encrypting them.

>Hashing is not encryption. Encryption is reversible — whoever holds the key can recover the original value — whereas a hash is a one-way function with no way back. That is exactly the property you want for passwords: you never need to read a password back, only to check whether a supplied one matches, so a leaked database of hashes does not hand the attacker the passwords themselves.

When we want to hash passwords to store them in a database, Bcrypt is the way to go and there are many libraries for different languages.

## Java Implementation

If your language of choice is Java you can use BCrypt with the library included in the **Spring Security** module.

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

    where `plainTextPassword` is the password we want to hash and `BCrypt.gensalt()` is a salt generated randomly.

    In case we want to increment the complexity, an optional parameter (`log_rounds`) has to be provided to `BCrypt.gensalt()`, which determines the computational complexity of the hashing. `log_rounds` is exponential ($$2^{log\_rounds}$$) and it specifies how many times to run the internal hash function. The default value is 10, and the valid values are between 4 and 31.

    The output strings after running `hashpw()` will look like:

    `$bcrypt_id$log_rounds$128_bits_salt184_bits_hash`

    `hashpw()` is smart enough to extract the salt from the string, so you do not need to worry about the salt value anymore, only the hashed string.

2. Check hashed password: 

    ```java
    BCrypt.checkpw(unencryptedPassword, hashPassword)
    ```

    This method hashes the supplied plaintext password using the salt it reads out of `hashPassword`, then compares the two hashes and returns whether they match.

## Conclusion

*BCrypt* is a very good hash algorithm for preventing **rainbow table attacks**, because it keeps the *salt* as part of the output from the *BCrypt* function. The idea is that every password gets a unique *salt* that is incorporated into the hash, so the same password produces a different hash for every user. A precomputed table is only useful against one salt, so an attacker would have to build a separate table per password — which is what makes the approach infeasible rather than merely slow.

> A Rainbow table is a precomputed structure that maps hash values back to the plaintext that produced them, letting an attacker reverse a stolen hash by lookup instead of by guessing. It trades memory for time by storing chains of repeated hash-and-reduce steps rather than every pair outright. Hackers can use it for cracking unsalted hashed passwords stored in a database.

**Bcrypt** is 10,000 times slower than _sha1_ to run. If we have a machine that is able to run it in 100ms, this is probably fast enough for login, but it might be too slow if we want to execute _Bcrypt_ against a long list of passwords. In consequence, if a hacker wants to run _Bcrypt_ a billion times by using the same computational power, it will take 27,777 hours.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/bcrypt-service" text="Source Code" %}
</p>
