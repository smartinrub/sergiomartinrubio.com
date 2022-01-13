---
title: Build a Docker JRE Image with Java 17 and Spring
image: https://lh3.googleusercontent.com/pw/AM-JKLXSuyzOqSXO-6jdRZ7I7JoihK47LOQxdXrFtIFec1jV8AmDC1fr3oPncFJLIVSjFiwNMXzT0ixAP4ckYuZXpRbt5ByvX5eGDLbZSlJ7h6lcxZKMJZ7xImOW8hsgn9IvRCp_AUnNcYlCn4onqvZcuMKt=w640-h427-no?authuser=2
author: Sergio Martin Rubio
categories:
    - Java
    - Docker
mermaid: false
layout: post
---

From Java 11 no *JRE* images have been released so far (2021-12-31) -- this translates into heavier Java images when building your application to be released to production. Before you could build a *Docker* image with a size of less than 80MB -- this was easily achieve with a multi-stage Docker build.

In the meantime the only choice for building a JRE is to use `jlink`.

> The [Eclipse Foundation seems to be developing a JRE](https://blog.adoptium.net/2021/12/eclipse-temurin-jres-are-back/){:target="_blank"} for Java 17 after a lot of feedback from the community. However, they still recommend using `jlink` since "it produces customized runtimes that only contain those Java modules that an application actually uses."

## Getting Started

### Create Project Structure

```bash
mkdir jre-build-example
cd jre-build-example
gradle init --type java-application // default selections should be fine
```

Once the project is created we can continue with adding some dependencies.

### Build a Minimal REST Application

We are going to use Spring Boot for building the REST API.

1. Add plugins and dependencies to your `build.gradle` file. It should look similar to this:

```groovy
plugins {
    id 'org.springframework.boot' version '2.6.2' // new
    id 'io.spring.dependency-management' version '1.0.11.RELEASE' // new
    id 'application'
}

repositories {
    mavenCentral()
}

java {
    sourceCompatibility = JavaVersion.VERSION_17 // new
    targetCompatibility = JavaVersion.VERSION_17 // new
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web' // new
    testImplementation 'org.springframework.boot:spring-boot-starter-test' // new
}

application {
    mainClass = 'com.sergiomartinrubio.jrebuildexample.App'
}

test {
    useJUnitPlatform()
}

```

2. Update the `App.java` with:

```java
@RestController
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @GetMapping
    public String helloWorld() {
        return "Hello World";
    }
}
```

3. Try it out:

```bash
./gradlew bootRun
curl localhost:8080
```

it should return `Hello World`.

### Dockerize Application

Create the following `Dockerfile`:

```dockerfile
FROM openjdk:17-jdk-slim-buster
WORKDIR /app

COPY app/build/lib/* build/lib/

COPY app/build/libs/app.jar build/

WORKDIR /app/build
ENTRYPOINT java -jar app.jar
```

You can check that it works by running:

```bash
docker build -t jre-build-example .
docker run -p 8080:8080 jre-build-example
```

and hit `localhost:8080`

If we run `docker images` **we can see that the size of the image is around 430MB**:

```bash
REPOSITORY                      TAG             IMAGE ID       CREATED         SIZE
jre-build-example               latest          e3488234026d   4 minutes ago   430MB
```

The next step is to reduce the size of the Docker image.

### Build a lighter Docker image

1. First of all we have to [modularize the application](https://sergiomartinrubio.com/articles/getting-started-with-spring-boot-and-java-9-modules/). We will create a `module-info.java` as follows:

   ```java
   module com.sergiomartinrubio.jrebuildexample {
       requires spring.boot.autoconfigure;
       requires spring.context;
       requires spring.boot;
   
       opens com.sergiomartinrubio.jrebuildexample to spring.core;
   
       exports com.sergiomartinrubio.jrebuildexample;
   }
   ```

2. Now we need to update the `build.gradle` with the gradle plugin `[org.beryx.jlink](https://badass-jlink-plugin.beryx.org/releases/latest/#user_guide){:target="_blank"}` that will run [jlink](https://docs.oracle.com/javase/9/tools/jlink.htm#JSWOR-GUID-CECAC52B-CFEE-46CB-8166-F17A8E9280E9){:target="_blank"}. 

   > *jlink* was released for the first time with Java 9 and the goal of this tool is to work with the [Java module system](https://sergiomartinrubio.com/articles/getting-started-with-spring-boot-and-java-9-modules/) (also introduced with Java 9) to provide a way to generate a modular run-time image, and as part of this the dependencies are assembled to provide an optimal image. 

   The updated gradle file is:

   ```groovy
   plugins {
       id 'org.springframework.boot' version '2.6.2'
       id 'io.spring.dependency-management' version '1.0.11.RELEASE'
       id 'application'
       id "org.beryx.jlink" version "2.24.4" // new
   }
   
   repositories {
       mavenCentral()
   }
   
   java {
       sourceCompatibility = JavaVersion.VERSION_17
       targetCompatibility = JavaVersion.VERSION_17
   }
   
   dependencies {
       implementation('org.springframework.boot:spring-boot-starter-web')
       testImplementation 'org.springframework.boot:spring-boot-starter-test'
   }
   
   application {
       mainModule = 'com.sergiomartinrubio.jrebuildexample' // new and name defined in module-info.java
       mainClass = 'com.sergiomartinrubio.jrebuildexample.App'
   }
   
   test {
       useJUnitPlatform()
   }
   
   // new
   jlink {
       options = ['--strip-debug', '--compress', '2', '--no-header-files', '--no-man-pages']
   
       targetPlatform('linux-aarch64', '/<PATH_TO_JAVA_HOME>/jdk-aarch64-17.0.1 ')
   
       launcher {
           name = 'app'
           jvmArgs = [
                   // com.sergiomartinrubio.jrebuildexample is defined in settings.gradle as rootProject.name
                   '--add-reads', 'com.sergiomartinrubio.jrebuildexample.merged.module=com.sergiomartinrubio.jrebuildexample',
                   '-cp', '../app/*',
                   '-Dlogback.configurationFile=./logback.xml'
           ]
       }
       mergedModule {
           additive = true
           uses 'ch.qos.logback.classic.spi.Configurator'
   
           excludeProvides servicePattern: 'reactor.blockhound.integration.*'
       }
       jpackage {
           imageName = 'App'
           skipInstaller = true
           installerName = 'App'
           installerType = 'pkg'
       }
       forceMerge('log4j-api', 'tomcat')
   }
   
   ```

   Within the *jlink* plugin definition we have to set configure a few things:

   - `options`: the jlink cli tool allows you to set some options that we can do with this plugin as well.
   - `tagetPlatform`: the target platform of the application must match the host architecture where the application will run. In this case I'm running the application will run in a linux ARM64 Docker image (this is because my Docker for Mac is running in a Mac M1 which is an ARM architecture). In case you are planning to build the Docker image from a x64 architecture we will have something like `targetPlatform('linux-x64', '/<PATH_TO_JAVA_HOME>/jdk-17.0.1')` . You can find the [JDK 17 for different platform on the Oracle site](https://www.oracle.com/java/technologies/downloads/){:target="_blank"}.
   - `launcher`: here we define things like the resulting name of the executable or the jvm arguments.
   - `mergeModule`: [this is to configure the module descriptor](https://badass-jlink-plugin.beryx.org/releases/latest/#_mergedmodule){:target="_blank"} (`module-info.java`).
   - `jpackage`: this is to customize the resulting image and installers.
   - `forceMerge`: This allows you to add dependencies to the merged module and it's quite useful for handling dependencies that are non-modular (e.g. `log4j`). 

3. Update the `Dockerfile` to contain the following:

   ```dockerfile
   FROM debian:stretch-slim
   
   COPY app/build/image/app-linux-aarch64 /app
   
   ENTRYPOINT /app/bin/app
   ```

4. Build and run the application one more time:

   ```bash
   ./gradlew clean build
   ./gradlew jlink // creates executable and jre
   docker build -t jre-build-example .
   docker run -p 8080:8080 jre-build-example 
   ```

   > The jlink executable and jre will be under `build/image/app-linux-aarch64/bin`. The are named `app` and `java`.

   and hit `localhost:8080`

   If you run `docker images` **we can see that the size of the image is now 115MB!** Before the image was 430MB.

   ```bash
   REPOSITORY                      TAG             IMAGE ID       CREATED          SIZE
   jre-build-example               latest          1b3daf650899   3 hours ago      115MB
   ```




## Next steps

You might want to build the *jre* and executable within a Docker container, so you don't depend on a local JDK. This can be done with a multi stage Docker build.

```dockerfile
FROM openjdk:17-jdk-slim-buster AS builder

RUN apt-get update -y
RUN apt-get install -y binutils

WORKDIR /app

COPY . .

RUN ./gradlew build -i --stacktrace
RUN ./gradlew jlink -i --stacktrace

# lightweight image
FROM debian:stretch-slim

COPY --from=builder /app/app/build/image /app

ENTRYPOINT /app/bin/app
```

​	then you can comment out on your `build.gradle` : `targetPlatform('linux-aarch64', '/<PATH_TO_JAVA_HOME>/jdk-aarch64-17.0.1 ')`.

{% include elements/button.html link="https://github.com/smartinrub/jre-build-example.git" text="Source Code" %}

