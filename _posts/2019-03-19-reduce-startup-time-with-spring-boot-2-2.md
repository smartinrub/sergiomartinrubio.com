---
title: Reduce Startup Time with Spring Boot 2.2
image: https://lh3.googleusercontent.com/x8p-5iuYF74sA8PuStSt-eAy7CG4Glst7oOJ0p_J8Y9PDWdpkWB7dLAU4GEmC6_pJGbeIvVze7PVzbpF9FxMQ4DJvAhaS4aiwpt87zV4IKWFE2goR7G3oJUUgnKSAQ7o7GSYpMYyaVYu46HuIq9LcClV1gVryVzj49jQcUAYukWbGgwX1cWgx70CAtlI29QEOs4XgNhLQrN0r1IVkrHyByXcbyMlZTlPz1pu9NLP8rJ2lVX_E9EkJW912G-mkY4PXR07oUiQx2ah8AdpNri61D0KpmMbkOaJA2Cj3fMSJkfdk7dsY0WlSSyGuzF7ubym_fT7bep2GtvGgTwAbo2ZAMagcTNGRFiRVD-Ue0kjJaEc_PDrien2zYrUJ7RFZk0C92qTXAnDLiicr4DFPyLZoopPaDeNoD_wd9AfumBLsR36P9xTzgYEB8T0bU7Jg_jFK6B_ujcwvfy8FD7Scr8q-y6MCjWj1rjr-G72sFDEQHqABH3ONBTnh5x6jh9qdWeBt0eie6S6Hu6Bl2Hciej1HQkob_7mSolTZb-o6txTtoNal8seMJ96Kgs3sldOrfAMg8ZykXjZz9D783WTohGngj0lcS27K_NicTKzZ8dq2h9qfaCSD7xu1N6VqexYqeQh6iv1tg2EMbxcEAOCRy0rokdhu0CgqlBWZB_CDnx23FyspL-R6FNKhJ4w4Wyh=w400-h266-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Java Framework
    - Spring
mermaid: false
layout: post
---

**Spring Boot 2.2** will be released soon (currently 2.2.0 M1) and it’s going to bring some [improvement and new features](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2.0-M1-Release-Notes) like lazy initialization of beans.

Until now [`spring-boot-devtools`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html){:target="_blank"} was the only way to see changes without having to restart the whole application. Whenever there are changes in your classpath folder(s) DevTools will take the appropriate action to apply those changes.  However, for IDEs like **IntelliJ**, adding this dependency is not enough, and you also have to:

1. Enable the “*Make project automatically*” option. You can find it in *Settings – Build, Execution, Deployment – Compiler*

{% include elements/figure.html image="https://lh3.googleusercontent.com/g3cSR3ZRkkjsCEhuV0-n9RfR9XLHe_qPxTc_e4Osiq_uM-GUoPb3wQl7IiGrBtLV0UNEotWdteI3EfZFoUWEpznI3lxBJw5mRYiaVa5l9dbPx8Nr7mToxe-3msc0mlAgF6dxCTSv=w2400" caption="IntelliJ Settings" %}

2. Open the registry, just press *SHIFT+CTRL+ALT+/* (**Ubuntu 18.04**). In the registry window, enable the “`compiler.automake.allow.when.app.running`” check-box.


{% include elements/figure.html image="https://lh3.googleusercontent.com/N1opxIOgiarGIzMUA1RnI7P7HpNaasTqWMFpUIUZQYB3McAmqwMqC3p7cM41PxlrfGYorBRHWvD8zjU3p0bVWNVQBH1t5QgkoSR-H_Ig5guR_nxEwkJtBeeA8-oIE-w-uA8-ZxZ6=w2400" caption="IntelliJ Registry" %}

Another alternative offered by **DevTools** is to explicitly tell **IntelliJ IDEA** to run “*Build Project*” (*CRLT+9* in **Ubuntu 18.04**) to build the target classpath. This feature is also available since **Spring Boot 2** when running the application in *Debug Mode*.

On the other hand, the latest _Spring Boot_ version offers an additional alternative. With the version 2.2 your beans can be initialized lazily by setting a *System Property*.

```properties
spring.main.lazy-initialization=true
```

If this property is set to `true` your beans will be initialized only when they are needed. For instance, if you have a dependency in your pom file to setup a DB connection, this will not happen until the connection is required. As you can imagine this can cause unexpected behaviors, so make sure you only use this during development. Moreover, this could **slow down some functionalities** of your application when they run for the first time, since it will need to initialize the required beans, and **will not show failures** until later, instead of during start up.

{% include elements/figure.html image="https://lh3.googleusercontent.com/8Gz9vQV_5NdUPe9_vCgX2T_0tHzxNVm2Qlc3VzHvQ0upzcwUwD38ADvmwqFGMAc4eyK2282Q5TS2pZwGKcEwfgza97u6tprYSzM4nXK9RsmO2-ep6W6wdzZuoawamK8OSURjdnBd=w2400" caption="spring.main.lazy-initialization=false" %}

{% include elements/figure.html image="https://lh3.googleusercontent.com/YDnU73O9s3RQl6m_J9CSUybPQK_YI_12iBTtKRxX3gptS3DdytAZIQtZo9MZ5Wo-6rpPBa8DYTmf0LbM8jUav-oWEgWLxeFNP_KW13biZVtLWcrWZCKfdZy5yrMyw4wYmCqGTQKh=w2400" caption="spring.main.lazy-initialization=true" %}

As you can see in the [previous examples](https://github.com/smartinrub/spring-boot-lazy-initialization), when the property is set to `false`, some connection errors are thrown, and the start up time is around **1.5 seconds**. whereas, when it is set to `true`, no failures are displayed and start up time is around **1 second**.
