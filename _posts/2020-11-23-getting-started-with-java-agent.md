---
title: Getting Started with Java Agent
image: https://lh3.googleusercontent.com/lcWSbMkhMhj3S-RXnJZ0MjeiMgV2NigFRKfD2c9Nwb4GzBkxsvgjZO63QcVUByszRt_JG--eV45GcrcyYjwrsAbVs8WkWEkatRRxmzFaXUhBGCGwvcQtmGmXM32zp_1cR2zQqqU6cmwdnCD02yUB3IyTdnGkvdWUJnqpAC_MWZGlztDYugjVhXVEYbbtGh5CThA7ipba4KZbjp2pNvRKdiXoefHlWYGqg-GvkHkvSyf0-UG0bJuqqbhBMlcMetFIyG4HK5MMHjp2jSWkX9N9-QU5O1ZneHpZqH____p3XLImmSbZSOgjLSLKIDYC1AEUqkeY0SU4FDipltTIabcdQgfSln0nAFq2Baosa-HkJjdREs93KLu4TqO5otCfu-ErLcBIPXJ4B-a95BFxZyENOctPww3bkx1HZ7ShSqKmWHU15JB90YLkxVR1K6HqTpWUoK8hinJLPwiZ-pCL0beljGUshPyUtPELlW5qlUaDb8LWrlGn1a6ZQel5rDK_ZZ6rZnECTC9MrzWV-1Malj-4TyokyZom9KJJ3QOktssio4HvpwUoAUcY6jGI3qIn5AtSuPe8C5Wxdm0s8tZXHV3D4ExNP7EZ1QZq-O82T9O1GUEs0X4bCTMxIvkPBISF0ZSrE71QXrXirWN4tBv85ZyzgAHGygKO0FCfdn5zUhieu27mb8XowdkcNNGERMJ0=w640-h384-no?authuser=1
categories:
    - Java
mermaid: false
layout: post
---

## Introduction

[Java Agent](https://docs.oracle.com/javase/1.5.0/docs/api/java/lang/instrument/package-summary.html){:target="_blank"}  is an instrumentation API which is usually overlooked and allows you to modify bytecode at runtime. This means that is a powerful feature but "with great power there must also come great responsibility".

 The **Java Agent** feature was added as part of *JDK 1.5* and has some use cases like metrics generation, for testing, in [Aspect Oriented Programming](https://sergiomartinrubio.com/articles/start-using-aspect-oriented-programming-with-spring-aop) and in any other scenario that requires modifying the bytecode at runtime.

A Java Agent is simply a class with a `premain` method that runs when the *JVM* is launched. We will show how to create a simple Java Agent in the next section.

## Implementation

1. Create a class with a `premain` method like this:

```java
public class MyJavaAgent {

    public static void premain(String agentArgs, Instrumentation inst) {
        System.out.println("This is a Java Agent");
    }
}
```

2. Create a `resources/META-INF/MANIFEST.MF` file where you specify the location of the agent class.

```
Manifest-Version: 1.0
Main-Class: com.sergiomartinrubio.javaagentexample.Main
Premain-Class: com.sergiomartinrubio.javaagentexample.MyJavaAgent
```

> The `MANIFEST.MF` is a file that contains information about the files included in the `.jar` file. i.e. You can set up the entry point of the application with `Main-Class` or a Java agent class with `Premain-Class`.

3. Compile the application. You can use Maven with `maven-jar-plugin` in your `pom.xml` file.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.0</version>
            <configuration>
                <archive>
                    <manifestFile>src/main/resources/META-INF/MANIFEST.MF</manifestFile>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

4. Run the generated Jar file with `-javaagent` that points to the `.jar` file that contains the Java Agent (in this case is the same Jar file).

```bash
java -javaagent:/Users/sergiomartinrubio/workspace/java-agent-example/target/java-agent-example-1.0-SNAPSHOT.jar -jar target/java-agent-example-1.0-SNAPSHOT.jar
```

> It is important that the `-javaagent` parameter goes before the `-jar` parameter.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-agent-example.git" text="Examples" %}
</p>



