---
title: How to Publish a Java Library to Maven Central
image: https://lh3.googleusercontent.com/pw/AM-JKLXIJE7ME0lqx19I-tPQCKryFhmZiyQKk-W7DU35x6gA9B72B-pJtBBdKAZU1xSSZRu3AYfV66BEwKCB5OND7kddnt9TSTH6LTFYd7LXC3yPJo7ARkcj9M_VQIrfX_oTRsbclQoxofut1_mIXUKIkVxP=w2458-h1640-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
    - Maven
mermaid: false
layout: post
---

The process of publishing a Java library to Maven Central is not as straightforward as you might think and we will go through all the required steps to share your new shiny Java library with the  of world.

## Steps

### Nexus Account Setup

Maven central requires having a unique Group ID that is usually a domain name reversed. If you own a domain, you can use it i.e. `com.sergiomartinrubio`, otherwise you can simply use your GitHub account i.e. `com.github.sergiomartinrubio`.

1. Create a [Sonatype Jira account](https://issues.sonatype.org/secure/Signup!default.jspa){:target="_blank"}. Use your email address from your domain if possible.
2. [Create Create a new Jira Issue](https://issues.sonatype.org/secure/CreateIssue.jspa?issuetype=21&pid=10134){:target="_blank"} as follows:
   - **Summary**: can be something like `Create new com.sergiomartinrubio project`.
   - **Group Id**: will be something like `com.sergiomartinrubio` or `com.github.sergiomartinrubio`.
   - **Project URL**: can be your website or where you document the library (i.e. GitHub account).
   - **SCM url**: is the host where the library is uploaded. i.e. `https://github.com/smartinrub`
   - **Username**: is the chosen username. i.e. `sergiomartinrubio`
   - **Description**: can be something like `I intend to publish open source projects.`

3. **Wait for a Reply for the domain verification**. If you own a domain you can verify the domain by [adding a `TXT` record to the resource records](https://central.sonatype.org/faq/how-to-set-txt-record/){:target="blank"}. The resource record must include the Jira ticket ID. i.e. [OSSRH-71520](https://issues.sonatype.org/browse/OSSRH-71520){:target="blank"}. In case you don't own the domain you'll have to [setup some coordinates](https://central.sonatype.org/publish/requirements/coordinates/){:target="_blank"}. The response is usually quick.

> If everything went well a comment with a guide for publishing your library is added.

Once you account is verified you should be able to access the [Nexus Repository Manager](https://oss.sonatype.org){:target="_blank"} where you will be able to see the artifacts of your uploaded libraries.

### Create GPG Key

GPG is required by Maven Central in order to be able to publish a libraries.

1. **Install GPG**. For MacOS users you can run: `brew install gpg`.

2. **Create key**: `gpg --full-generate-key`. It is recommended to use your real contact information and a secured password.

3. **Upload public key to a server** so *sonatype* can find it:

   ```sh
   gpg --list-keys # list created gpg keys
   gpg --keyserver keyserver.ubuntu.com --recv-keys <key_id> # upload key to server
   ```

> You can also export the *gpg* key with `gpg --export-secret-keys <key_id> > sonatype_upload.gpg`

You can find more detailed information on the [Sonatype GPG configuration page](https://central.sonatype.org/publish/requirements/gpg/#delete-a-sub-key){:target="_blank"}.

### Configure Maven Settings

```sh
nano ~/.m2/settings.xml
```

```xml
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <username>SONATYPE_JIRA_ACCOUNT_USERNAME</username>
      <password>SONATYPE_JIRA_ACCOUNT_PASSWORD</password>
    </server>
  </servers>
</settings>
```

### Configure your Maven Project File

Go to your `pom.xml`.

**Add project metadata**:

- Group Id, artifact Id, version, project name, description, url...

  ```xml
  <groupId>com.sergiomartinrubio</groupId>
  <artifactId>telegram-bot-client</artifactId>
  <version>0.1.0</version>
  
  <name>telegram-bot-client</name>
  <description>Client library for interacting with Telegram Bots</description>
  <url>https://www.sergiomartinrubio.com</url>
  ```

  

- **Organization information**. i.e.:

  ```xml
  <organization>
    <name>sergiomartinrubio.com</name>
    <url>https://www.sergiomartinrubio.com</url>
  </organization>
  ```

- **Licenses**. i.e.:

  ```xml
   <licenses>
     <license>
       <name>Apache License, Version 2.0</name>
       <url>https://www.apache.org/licenses/LICENSE-2.0</url>
     </license>
   </licenses>
  ```

- **Software Configuration Management** (*scm*) information. i.e.:

  ```xml
  <scm>
    <url>https://github.com/smartinrub/telegram-bot-client</url>
    <connection>scm:git:git://github.com/smartinrub/telegram-bot-client.git</connection>
    <developerConnection>scm:git:git@github.com:smartinrub/telegram-bot-client.git</developerConnection>
  </scm>
  ```

- **Developers** information. i.e.:

  ```xml
  <developers>
    <developer>
      <name>Sergio Martin Rubio</name>
      <email>me@sergiomartinrubio.com</email>
      <organization>sergiomartinrubio.com</organization>
      <organizationUrl>https://www.sergiomartinrubio.com</organizationUrl>
    </developer>
  </developers>
  ```

**Configure Maven Distribution Management**:

You might need to include the following distribution management entry if you are going to publish artifacts to the snapshots repository.

```xml
<distributionManagement>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
</distributionManagement>
```

**Include the following Maven plugins**:

- [nexus-staging-maven-plugin](https://help.sonatype.com/repomanager2/staging-releases/configuring-your-project-for-deployment){:target="_blank"}: is used to perform the deployment to the Nexus repository.

  ```xml
  <plugin>
    <groupId>org.sonatype.plugins</groupId>
    <artifactId>nexus-staging-maven-plugin</artifactId>
    <version>1.6.8</version>
    <extensions>true</extensions>
    <configuration>
      <serverId>ossrh</serverId>
      <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
      <autoReleaseAfterClose>true</autoReleaseAfterClose>
    </configuration>
  </plugin>
  ```

- [maven-source-plugin](https://maven.apache.org/plugins/maven-source-plugin/){:target="_blank"}: plugin for generating the a jar file with the source files of the project. **This is required by Maven Central.**

  ```xml
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-source-plugin</artifactId>
      <version>3.2.1</version>
      <executions>
          <execution>
              <id>attach-sources</id>
              <goals>
                  <goal>jar-no-fork</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
  ```

- [maven-javadoc-plugin](https://maven.apache.org/plugins/maven-javadoc-plugin/){:target="_blank"}: used for generating the *javadoc* files for the project. **This is required by Maven Central.**

  ```xml
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-javadoc-plugin</artifactId>
      <version>3.3.0</version>
      <executions>
          <execution>
              <id>attach-javadocs</id>
              <goals>
                  <goal>jar</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
  ```

- [maven-gpg-plugin](https://maven.apache.org/plugins/maven-gpg-plugin/){:target="_blank"}. This is used to sign the project artifact with *GPG*.

  ```xml
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-gpg-plugin</artifactId>
      <version>3.0.1</version>
      <executions>
          <execution>
              <id>sign-artifacts</id>
              <phase>verify</phase>
              <goals>
                  <goal>sign</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
  ```

*Sonatype* documentation also provides a [detailed guide to configure the different Maven plugins](https://central.sonatype.org/publish/publish-maven/){:target="_blank"}.

### Publish you Java Library

Before publishing your library you might want to update the version. You can simply run the following Maven command:

```sh
mvn versions:set -DnewVersion=1.0.0
```

You can also create a Maven profile for publishing the artifact so the release is not executed by mistake. Include plugins from the previous section here:

```xml
<profiles>
  <profile>
    <id>release</id>
    <build>
      ...
      nexus deploy, javadoc, source and gpg plugin
      ...
    </build>
  </profile>
</profiles>
```

Now you can run the following Maven command:

```sh
 mvn clean deploy -Prelease
```

If everything went well you should be able to see your library at [Nexus Repository Manager](https://oss.sonatype.org){:target="_blank"}, however, It might take a while to show up. Also, the library will also appear at https://mvnrepository.com, but I noticed this takes about half a day or even a day.

## Troubleshooting

You are using MacOS you might need to add to your `.bash_profile` or `.zshrc`:

```sh
export GPG_TTY=$(tty)
```

{% include elements/button.html link="https://github.com/smartinrub/telegram-bot-client.git" text="Source Code" %}

Photo by [Devon Divine](https://unsplash.com/@lightrisephoto?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/library?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
