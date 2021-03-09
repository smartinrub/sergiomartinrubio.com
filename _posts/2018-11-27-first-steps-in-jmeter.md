---
title: First Steps in JMeter
image: /assets/images/shubham-beeharry-223969.jpg
author: Sergio Martin Rubio
categories:
    - Testing
    - Performance
mermaid: false
layout: post
---

In this post we are going to go through the main **JMeter** features, but first of all, what is [JMeter](https://jmeter.apache.org/){:target="_blank"}? basically, it is a tool to perform load testing against:

- Web – _HTTP, HTTPS APIs_
- Webservices (_SOAP_ or _REST_)
- _FTP_
- Database via _JDBC_
- _LDAP_
- _JMS_ services
- Mail
- Native commands or shell scripts
- _TCP_
- _Java Objects_

However, we will only cover the HTTP functionalities of **JMeter**.

Performance tests are required to make sure your application will not crash under heavy traffic, and survive dates such as _Black Friday_ or _Christmas_. Therefore, it is a very important phase of the web development process.

**JMeter** is an open source and _Java_ based that provides an _GUI_ to create your test plans, and simulates browser behavior by sending customized request to your website.

## JMeter

### Installation and Execution

#### Ubuntu 18

1. Download binaries from **Apache JMeter**.
2. Unzip file:

    ```shell
    tar -xf apache-jmeter-5.0.tgz
    ```

3. Run executable file:

    ```shell
    cd apache-jmeter-5.0/bin
    ./jmeter
    ```

#### MAC OS

1. Install it with **brew**

    ```shell
    brew install jmeter
    ```

2. jmeter should be on your `PATH`

    ```shell
    jmeter
    ```

## Elements of a Test Plan

This parent object will contain all the necessary components to build your test plan.

### Thread Group

**Thread Group** is the first element of your test plan and all samples, controllers and listeners are meant to be under this object. However, listeners are allowed to be under the _Test Plan_ object, and in this case they are applied to all _Thread Groups_. What can you do from a _Thread Group_?

- Set number of threads or users.
- Set the time for _JMeter_ to run all threads, or in other words, it is the time needed by _JMeter_ to add all users (threads) to the test execution. If you set this value to 100 and the number of users is 10, it will add a user every 10 seconds.
- Set how many times a _Thread Group_ will execute.
- Other options: scheduler, delays, actions after error…

### Controllers

**_Sampler_**

This type of controller let you send different types of requests (e.g. _HTTP, FTP, JDBC_…).

The _HTTP Request_ controller is used to send requests to the web server.

If you are planning to send multiple request to the same server it is recommended to use _Config Element_ components for your _Thread Group_, so you can set default request values:

-  **Defaults** allows you to specify default values for your _HTTP_ calls (e.g. domain, port, parameters, body…) inside a _Thread Group_ or for the whole _Test Plan_.
- **HTTP Header Manager**: allows you add or override _HTTP_ request headers.
- **HTTP Cookie Manager**: stores and sends cookies like a web browser. It is very useful to keep the same session in a _Thread Group_.
- **HTTP Cache Manager**: To simulate browser cache features and with the default _JVM_ value, the limit is 5000 items.
- **HTTP Authorization Manager**: for web pages with restricted access.

Another interesting option to generate test samplers automatically is [HTTP(S) Test Script Recorder](https://jmeter.apache.org/usermanual/component_reference.html#HTTP(S)_Test_Script_Recorder).

**_Logic Controller_**

We can use these controller to specified how and when to run the samplers. The most relevant types are:

- **Simple Controller**: for organization purposes only.
- **Loop Controller**: To loop through an object a particular number of times. If you set the value to -1 it will loop forever.
- **Once Only Controller**: runs what is inside once per thread.
- **Interleave Controller**: alternate among each controller. If you use it with a _Loop Controller_ it will run a different controller for each iteration.
- **Random Controller**: it is like the interleave controller but without an order.
- **Random Order Controller**: it is like _Simple controller_ but without an order.
- **Throughput Controller**: it controls how often a controller is executed.
- **Runtime Controller**: it sets for how long the children controllers will run.
- **Recording Controller**: it is a placeholder to use in combination with _HTTP(S) Test Script Recorder_.

### Listeners

Listeners are essential to retrieve the results from your tests and analyze the _Sampler_ requests and responses by looking at tables, graphs, trees, statistics… Moreover, they allow you to save the results in csv or xml files, so you can have a look at them later.

Some of the most relevant listeners are:

- **View Results Tree**: this listener shows requests and responses to see if something went wrong during the calls. Since it consumes a lot of resources, it is only recommended to use for debugging purposes.

{% include elements/figure.html image="https://lh3.googleusercontent.com/u_v6Qrm-B4udK88lY1kZmbmPqH5b7VDJFf_lK8qWQqqKEHP3MEnRPE6FowK22DeqGwOKafTql-8lxN82i_7cP8APG1Lm27Od9ctlOaKgW2SgRoY_u24p4f6_ZLmq-a5xUZ5gFLcO=w2400" caption="JMeter - Results Tree" %}

- **Aggregate Report**: gives you some statistics parameters (average, median, min, max, 90th percentile…) in form of a table.

{% include elements/figure.html image="https://lh3.googleusercontent.com/T5PuPN_vbh1_8fAEerkeJ9U9vqqY6sSnV2ymdD9xhj6yt73ZZmrzvdGt6ouxMqlb4WxoaCLCYjQD0-RKjKZJwRZTLoHoPxaPOKTu64sECW0O1RF3hNK85S8iL9dGvNiJ19wFrBP-=w2400" caption="JMeter - Aggregate Report" %}

- **View Results in Table**: creates a row for every sample result, so you can see in depth what is happening during each result. Bear in mind that this listener also consumes a lot of memory.

{% include elements/figure.html image="https://lh3.googleusercontent.com/YQoPGodxuxrR2oASmwutuYi3PPyShOLvTQ4oeWFbM1h9_RNvEIomH5RI6rH9fixCHKHY1AF1d5ZFHjvxmsh7swrB3B1TFbYMLlGjD378z0nwx7moG-WJMBFEc9RRxdbRFcmivLxM=w2400" caption="JMeter - View Results" %}

- **Simple Data Writer**: this is the most performance listener, since the only thing it does it is write the metrics in a external file.

{% include elements/figure.html image="https://lh3.googleusercontent.com/M4lMFtgCsby8bUbn13ajQP21_e337vMV5U6tWR8O8lMMfzbcu9yMeqi1A-pg3Rg44w8TFRvWrBCgsoxEws0AzuGsP_pAb6QKYCCBSNV5D6tOyfMlGb6BW2uxyWrV-_4ewbrs7m3L=w2400" caption="JMeter - Data Writer" %}

- **Backend Listener**: you can even push data to your _Graphite_ server. You can try this listener by running a _dockerized_ version of _Graphite_.

{% include elements/figure.html image="https://lh3.googleusercontent.com/YcEchTIoon604za-_lyi46qa_PL5QgqpDlWD4x1K6w9YZgeO-ygEPv90u_F3NxY62_RyXnzrr_C19e2xnLpxOS7fIcLaxCsfcZed_CkPBWNqY9KLApb4j7BtP5CxhEQJ_Sp7Sj8P=w2400" caption="JMeter - Listener" %}

{% include elements/figure.html image="https://lh3.googleusercontent.com/aJCQJxCmuZHXJ752dD1ShhmQ1O6susVSbZL7lRvC2qQpDDw9d8K6ANTPj7vcEGs2OokC5ez1ruxTiRId9buFdyNzlpr45r4zv-eSSojB3Hg1tJfKf8SYSwJxrbg7XsWWadEZnG-f=w2400" caption="Graphite Dashboard" %}

In order to save these reports into **CSV** files you only need give a path a filename. For **XML** output _Save as XML_ option needs to be selected in _Configure_.

### Timers

**Timers** add delays to every requests of your **Thread Group** to avoid overwhelming the server. If you want to apply a timer to only one element, add the timer as a child of it. Some of the common timers are:

- **Constant Timer**: it delays each call for the same amount of time.
- **Uniform Random Timer**: it delays each request for a random amount of time.
- **Constant Throughput Timer**: this timer uses variable delay for each request base on the server performance, so what you set on this timer is the number of samples per minute.

### Assertions

**Assertions** are very useful to make sure for instance you are getting the expected response.

The most relevant assertion for _HTTP_ requests is _Response Assertion_. This assertion allows you to set patterns for _Request and Response Headers_, _Request Data_ or _Response Message_.

{% include elements/figure.html image="https://lh3.googleusercontent.com/p1Vv4dj72ETMp9WgjB-1p_bkZesmDZRhY0GXIoKcplZkEy1h-qDbUB6sCzR2s7Wk2FoL5-AOP9ykvASy1E3L0HOUafPfq7r978AFOOiAtDWwEuE3DQdJX8fVUYQF1O29Kzl2ynVB=w2400" caption="JMeter - Response Assertion" %}

### Templates

**JMeter** includes some templates to get started. Building an **Advanced Web Test Plan** is one of them and includes many of the elements described before (config elements, controllers, assertions, listeners, timers…).

{% include elements/figure.html image="https://lh3.googleusercontent.com/e-xiY2-yPEOh1jWAKSJQmudPzZflCFbbFrNOrjm8uWcyA2eEuG2G7BUXMAJPIOYIfVG9FIYR23e-RpFLGYclYdJ9EgD8GGluyiX8X1HUZK6VawlevnVqzfsdDQXA0u_vKlDGY-u2=w2400" caption="JMeter - Left Pane" %}

### Running Test Plan

There are two options to run your test plan. You can use the _GUI_ provided by _JMeter_, however this alternative is not recommended, since JMeter consumes much computer memory, and it should only be used for creating the test script. On the other hand, non _GUI_ is much more lightweight and should be used for running the test plan.

```shell
jmeter -n -t test_plan.jmx -l results.csv -j jmeter_logs.log
```

where `-n` is _non-gui mode_, `-t` is _Test Plan_ file, `-l` file to save results (_csv_, _jtl_) and `-j`  _JMeter_ log file.

## Conclusion

**JMeter** is a powerful and customizable tool for load testing, and provides a friendly user interface to create your scripts. However, it is a heavyweight tool compare to other load testing utilities like _Gatling_ or _Locust_.
