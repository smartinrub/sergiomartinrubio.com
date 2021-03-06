---
title: Improve Code Quality with SpotBugs
image: https://lh3.googleusercontent.com/CACxH2p9BsBeEr4xVKTYWnlb_n59ppFmxxjL8F0XAbrO-R26FABMHdkOHrmE_w8nsAY__5KFBXVRAzbzYmhdsxRcLpRyT6hhpxXUjYaSrDZ6QvE97gOYSwhvrwCKcGZiAcTk-UWB9kI5wysFLLMGJ2Lxa2mUufVdbtVjSxyHvmxRfqPFChOcyBfKKBBV-0I2fwfKVaPzNgH3cpXDCT2qiovGJNS51osyB-0ZxbWgnaPStU7RuiSTHi5vFsjZuaHwW9retbfthtk3H_hzXqSC9HIg9-clnUdTvG0TZXQ6g0DAgfQDnyYQoDOA9dgDrLUsG1kApQLUoNTAoq52RRxrBl2yfkcCrDcD_y1Vvk4b5wa1V1CevAdAM_YN3PLKFULvIdHelovAG0OUMQGY9p_7JEyslNwLHxC-MpKR0UOgz5p_f5mPUZnC3h9YFrGDY-d9Hb6puHzEJUM1pHQY5Dh3H2vjVN6WWyfrsvJbI-3Z7slMZtmx-32SPGIMx7AIcgdvFjRbgPT0ISSiOjkRDCUrsj9z36GOWwL3FLhBRWRHclA8Hk2anEZi27OB179IuN_r07nffkIynmXKgSdcd1e95xDCgXbLV1NB2gOaZsEzwvXsQq__iWX5pLSv36ovWX9zPEhw8rSsxMyI5AFvkAZAXN9-Al2Hug2nvSTyWAISlfvRQGunO0QuxfAVEl85=w460-h308-no?authuser=1
categories:
    - DevOps
    - Analysis
mermaid: false
layout: post
---

## Introduction

Every developer should ensure code quality and follow language conventions, otherwise [Technical Debt](https://medium.com/existek/what-is-technical-debt-and-how-to-calculate-it-80193e4e746d){:target="_blank"} is created, and at some point in the future you will have to revisit that smelly piece of code.

**Code Debt** can be created without even realizing when: 

- development has to be done before a deadline; you do not have enough experience; 
- or simply you are having a bad day. 

Because of this, it is very important that before merging changes into master we double check that you are not adding performance or security issues, or any other kind of code smell.

## SpotBugs

- [SpotBugs](https://spotbugs.github.io/){:target="_blank"} helps you analize your **Java** code to find bugs.
- Free open source tool which was previously named [FindBugs](http://findbugs.sourceforge.net/){:target="_blank"}.
- It is a plugin available for _Maven_, _Gradle_, _Eclipse_, _Ant_...
- It looks at your source code and runs a static analysis.

### How To Use It

There are a few ways of using **SpotBugs**, however, we are going to focus on the SpotBugs plugin for Maven. Alternatively, you can execute _SpotBugs_ on _Windows_, _macOS_ or _Linux_ to run the _SpotBugs GUI_; install a plugin for _Eclipse_; integrate it with **Ant** or add a **Gradle Plugin**.

1. Add the _SpotBugs_ plugin into your pom.xml file:

```xml
<plugin>
  <groupId>com.github.spotbugs</groupId>
  <artifactId>spotbugs-maven-plugin</artifactId>
  <version>3.1.11</version>
  <dependencies>
    <dependency>
      <groupId>com.github.spotbugs</groupId>
      <artifactId>spotbugs</artifactId>
      <version>4.0.0-beta1</version>
    </dependency>
  </dependencies>
</plugin>
```

2. Run one of the goals provided by the plugin:

- **spotbugs** goal: runs analysis
- **check** goal: runs analisys and fails build if bugs are found
- **gui** goal: launches GUI to see results
- **help** goal: shows how to use SpotBugs plugin

```shell
mvn clean install spotbugs:spotbugs
```

### Advance Configuration

Additionally, you can know more about one particular goal if you run:

```shell
mvn spotbugs:help -Ddetail=true -Dgoal=check
```

and set configuration properties like:

```shell
mvn clean install spotbugs:check -Dspotbugs.failOnError=false
```

or in a configuration block inside the **SpotBugs Maven plugin** block like below:

```xml
<configuration>
    <threshold>Low</threshold>
    <effort>Max</effort>
    <debug>true</debug>
</configuration>
```

_SpotBugs_ also allows advance [filter configuration](https://spotbugs.readthedocs.io/en/stable/filter.html#){:target="_blank"} in order to include or exclude specific classes or methods from the report. First, we need to create a _xml_ file where we are going to define our _Match_ elements:

```shell
cd your-project-root-folder
touch spotbugs.xml
```

and add it in your _SpotBugs_ plugin configuration:

```xml
<includeFilterFile>spotbugs.xml</includeFilterFile>
```

A filter file is a _XML_ file with a parent `FindBugsFilter` tag which contains as many `Match` tags as you want, and each `Match` element accepts many [types of Match clauses](https://spotbugs.readthedocs.io/en/stable/filter.html#types-of-match-clauses){:target="_blank"}. Moreover, you can use [Java regular expressions](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html){:target="_blank"} to include/exclude classes, methods, fields or sources.

An example with some of the _SpotBugs_ clauses can be found below.

```xml
<FindBugsFilter>
    <Match>
        <Class name="com.sergiomartinrubio.spotbugsexample.PerformanceBugs" />
        <Bug category="PERFORMANCE" />
    </Match>
    <Match>
        <Class name="com.sergiomartinrubio.spotbugsexample.CorrectnessBugs" />
        <Not>
            <Method name="test" />
        </Not>
    </Match>
    <Match>
        <Class name="com.sergiomartinrubio.spotbugsexample.BadPracticeBugs" />
        <Or>
            <Method name="removeAllFromCollection" returns="void" />
            <Method name="NamingConvention" returns="void" />
        </Or>
        <Bug category="BAD_PRACTICE" />
    </Match>

    <Match>
        <Package name="~.*\.spotbugsexample" />
        <Bug pattern="UCF_USELESS_CONTROL_FLOW" />
    </Match>
</FindBugsFilter>
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/spotbugs-example" text="Code Examples" style="dark" %}
</p>

### GUI

_SpotBugs_ has a GUI out-of-the-box by simply running:

```shell
mvn spotbugs:gui
```

> A build and _SpotBugs_ analysis is required.

A new window will pop up with a bug tree which gives us the following information:

- Bug Category
- Bug Kind
- Bug Pattern
- Bug Rank

{% include elements/figure.html image="https://lh3.googleusercontent.com/ANLKz0dfuvd1m0fKUhuUkBCGg5mBbObv0li-eiqbHiXeJpdy5a9N8LfJE5KZY0t0KtKjcb4_FntTYwYCsVUiKFQBVUOdGIPIppwLGokGYtoyhIcsda3J8BURIlDj3XCNao7L8uZm=w2400" caption="SpotBugs GUI - Hierarchy" %}

When a particular bug is selected we will see a pane with the class and highlighted line code, and at the bottom a description and possible solution.

{% include elements/figure.html image="https://lh3.googleusercontent.com/_Tr_pt9VWpFMnJFYqwTPncFcaXofAyao5qpfG1veW6pEC0h9pzSDwneraWbY1hF3DaiE2Hb6_7xvnswG6lfoPXNHkp2GjJqtTXfcOg_dhaDOjANqkuYqBcmJGbAT1vIpt3ChiyIY=w2400" caption="SpotBugs GUI - Source Code and Description Panel" %}

You can also save the report and import or export filters.

> The GUI can also be run by itself and load the jar file that we want to analyze.

## Alternatives

Probably you have already heard about [**SonarQube**](https://www.sonarqube.org/){:target="_blank"}, which basically provides _SpotBugs_ features and a few extra more. In fact, _SonarQube_ used to use _FindBugs_ plugin to generate bug reports, until they decided to used their own analyzer and stop using [Checkstyle](http://checkstyle.sourceforge.net/){:target="_blank"}, [PMD](https://pmd.github.io/){:target="_blank"} and **FindBugs**. 

**Why would you choose SpotBugs?** Because it is easier to integrate into your _Maven_ build, rather than relying on a separate Sonar server, and having to learn an additional _API_. The greatest benefit of **SonarQube** is the GUI, which lets you configure anything easily. Nevertheless, you could run something like [Jenkins Warnings Next Generation Plugin](https://github.com/jenkinsci/warnings-ng-plugin){:target="_blank"} as part of your **Jenkins CI build** and have nice graphs.

## Conclusion

Whether you are a junior or senior developer, code analysis tools like _Spotbugs_ can be very useful to improve code quality and avoid bugs, so you should consider adding this tool to your build.
