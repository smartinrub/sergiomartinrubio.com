---
title: Java Memory Model Overview
image: https://lh3.googleusercontent.com/pw/ACtC-3eDuWIMt0pm7BOj4jw375yNoKYJKLa5DDeUnpdz67gh-8UmRlpKbFYFKpcn9gmK8bJExOCQc0CPYIa4QUij7oBhNzoJURM6TGo75WpmVFtJi56M9uH88-7n2ZRmsHUEGLcgNiNX-cMvcNgfjxeFnqCp=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java
mermaid: false
layout: post
---

The _JVM_ uses its own memory model and as soon as it is launch the system allocates memory for the process, and this memory is shared by:

- **Heap Memory**: _Java_ objects are stored in one of the [heap](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.3){:target="_blank"} areas.
- [Metaspace](https://www.baeldung.com/java-permgen-metaspace){:target="_blank"}: since _Java 8_ it has replaced the older PermGen memory space to provide a more flexible and reliable memory usage.
- **Stack Memory**: a [JVM stack](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.2){:target="_blank"} is created for each thread and used to store local variables and partial results, perform dynamic linking, return values from methods and dispatch exceptions.
- [Codecache](https://docs.oracle.com/javase/8/embedded/develop-apps-platforms/codecache.htm){:target="_blank"}: it is mostly used by the _just-in-time_ (_JIT_) to store bytecode compiled into native code.

{% include elements/figure.html image="https://lh3.googleusercontent.com/wfKu7pzoAPH-H9VS0w6K2MGn4vVVTU3Q2nM0P-bbl8SlJS6AfFicTDprqv0YGLqA4LpV4elWl7GCBjgNOVkIojcmqmPibGUMzu9Fi2rQ5uAWz1B4GJTALu8gTINW0PtgrH_7AB9sWQ=w800" caption="JVM Memory Model" %}

## Heap Memory

The **heap memory** is divided into two areas or generations, young (also called nursery) and old generation. The heap is where your object data is stored and it is managed by the [Garbage Collector](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html){:target="_blank"}.

>Garbage collection: is the process of looking at the heap memory to clean up the unused objects. A Used object is a referenced object, which means that  somewhere in your program there is something pointing to that object. In other languages like _C_ this is a manual process whereas in _Java_ is handled automatically.

### Young Generation

If the _young generation_ fills up, this causes a minor garbage collection and it triggers a _Stop the World_. Surviving objects are moved to the old generation.

>_Stop the World_: All the application threads are stoped until a particular operation finishes.

Inside the _Young Generation_ there are three areas: Eden, Survivor Space 0, and Survivor Space 1.

### Old Generation

The _Old Generation_ is used to store long surviving objects. To classify an object as a candidate to be moved from the _young generation_ to the old generation a threshold is set. Eventually the _old generation_ needs to be collected and this event is called a major garbage collection.

A mayor garbage collection is also a _Stop the World_ event. However, this type of Stop the World is much slower since it involves live objects, so this kind of events should be minimized.

### Generational Garbage Collection Process

The _Garbage Collector_ is an automatic process responsible for identifying and delete unused objects. Steps:

1. New objects are put in the eden and survivor spaces start empty.
2. When the eden space is full a minor garbage collection is triggered. Referenced objects are moved to the first survivor space (_S0_) and unreferenced objects are removed.
3. Next time the eden fills up it will happen the same, but this time referenced objects are moved to the second survivor space (_S1_), and referenced objects from _S0_ are also moved to 
 and age is incremented. Unreferenced object in _S0_ are removed.
4. During the next minor _GC_, the same process occurs, but this time survivor spaces switch. Referenced objects are moved to _S0_, survivors are aged and unreferenced objects from eden and _S1_ are removed.
5. When objects reach a particular age they are promoted from the young generation to the old generation.
6. The previous 5 steps repeat until a major _GC_ is performed on the old generation which cleans up and compact the space.

{% include elements/figure.html image="https://lh3.googleusercontent.com/5li9Ai49SfxisvdcKlUS3s3ryvNSn7yDeGq4CsLC3dORyuumzV3ih41jXfNr0Q-gH_tLIc58as8CLybPTng8D7Uar1cCRN4r4lD8jDTfmP2YBIVRcsbmkzhcriQ0tnA1pJO1-xqEnQ=w800" caption="Generational Garbage Collection Process Diagram" %}

### Garbage Collector Selection

When no _Garbage Collector_ is selected by CLI (e.g. `-XX:+UseSerialGC`), the _JVM_ selects a _GC_ based on the platform is running on, heap size and runtime compiler. The process of selecting a particular _GC_ is called **[Ergonomics](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/ergonomics.html#ergonomics)**.

[HotSpot Garbage Collection Types](https://javapapers.com/java/types-of-java-garbage-collectors/){:target="_blank"}:

1. **Young Generation Collection**
-  **Serial** (_S GC_) is a stop-the-world. It uses a single thread for garbage collection and freezes all the threads while doing the garbage collection.
-  **Parallel Scavenge** is a stop-the-world, copying collector that uses multiple GC threads 
-  **ParNew** (_P GC_) is a stop-the-world. It uses multiple threads and also freezes all the threads during garbage collection. It differs from _Parallel Scavenge_ in that it has enhancements that make it usable with _CMS_. For example, _ParNew_ does the synchronization needed so that it can run during the concurrent phases of _CMS_.

2. **Old Generation Collection**
-  **Serial Old** is a stop-the-world, mark-sweep-compact collector that uses a single GC thread
-  **CMS** (_Concurrent Mark Sweep_, _CMS GC_) is a mostly concurrent, low-pause collector. It uses multiple threads and only freezes the threads while marking the referenced objects in the tenured generation space, and if there is a change in heap memory in parallel while doing the garbage collection. It uses more CPU to improve throughput.
-  **Parallel Old** is a compacting collector that uses multiple GC threads

3. [G1](https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html){:target="_blank"} (_G1 GC_) is the Garbage First collector for large heaps and provides reliable short GC pauses
- Has generations but uses different memory layout
- Default collector in _JDK 9_
- It supposed to replace CMS in the long term and the main differences with _CMS_ are:
    - It uses regions to simplify the collection and avoid fragmentation issues.
    - G1 offers more predictable garbage collection pauses and allows users to specify desired pause targets.

{% include elements/figure.html image="https://lh3.googleusercontent.com/wZSqu7r16KpwA2-7pbQFzorYharzcuhnLi5R21xcYpzQ2zf0F_D3HOE7aUP_sRo3TNWT6nf3XzZx5JWT0-nTnsLBubzp21BnJfJDno5fj1lzkS12CaJ9a4CKtp4l7XMxlPRPqQ9JFg=w800" caption="Generational Garbage Collection Process Diagram" %}

5. _[The Z Garbage Collector](https://www.opsian.com/blog/javas-new-zgc-is-very-exciting/){:target="_blank"}_ - _ZGC_: Is designed to offer very low pause times on large heaps and it does this through the use of colored pointers and load barriers.

The GC selected by default in your machine can be checked with the following _JVM_ option:

```shell
java -XX:+PrintCommandLineFlags -version
```

>All options are set or retrieve with `-XX`. Options that have true/false values are specified using + for true and - for false. e.g. `-XX:+PrintCommandLineFlags` sets this option to true. To see all you default JVM options you can run `XX:+PrintFlagsFinal`.

If a parallel GC is selected you can tune a couple of parameters:

- _Maximum Pause Time Goal_ (`-XX:MaxGCPauseMillis=<milliseconds>`): The JVM will adjust the heap size and other GC parameters to reduce the pause time during garbage collector stops. This might cause more frequent GC and reduce the application throughput.
- _Throughput Goal_ (`-XX:GCTimeRatio=<milliseconds>`): The ratio is `1 / (1 + <milliseconds>)`. For instance, if it is set to 19, it will be 1/20th or 5% of the total time for the garbage collection. This ratio affects both the young and old generation time. The JVM might increase the size of the generations, so the application will run for longer without having to execute a garbage collection.

### Tuning Heap Memory

System performance is greatly influenced by the size of the Java heap available to the JVM.

[Rules](https://docs.oracle.com/cd/E15523_01/web.1111/e13814/jvm_tuning.htm#PERFM161){:target="_blank"}:

- Heap Size < Available Physical RAM.
- Young Generation Size < Total Heap Size/2
- It's recommended to use same values for minimum and maximum heap size to prevent wasting VM resources used to constantly grow and shrink the heap.

**Options**:

Description | Option
---------|----------
 Initial java heap size | `-Xms`
 Maximum Heap Size | `-Xmx`
 Initial and maximum size of the young generation | `-Xmn`

> _Oracle_ docummentation advices not to choose a maximum value for the heap unless you know that you need a heap greater than the default maximum heap size. Instead choose a throughput goal that is sufficient for your application.

## Metaspace

**Metaspace** is the memory allocated for metadata. In _JDK 8_, the permanent generation was replace with metaspace, and now class metadata is stored in native memory.

The memory is allocated in the same as C does (Metadata uses space allocated by mmap, not by malloc).The amount of native memory that can be used by **Metaspace** area is by default unlimited, however you can use the option `MaxMetaspaceSize` to set an upper limit. The GC will also run on metaspace when is getting full.

### Java Class Metadata
**Java Class Metadata** is JVM's **internal model of everything in the byte-code. The _JVM_ gets all the byte-code, creates the model and throughs away the byte-code (byte-code is not used because of complexity reasons).

It also contains:

- _Resolution state_ (linkage to other classes/interfaces, fields, methods and constants).
- _Interpretation state_ (quick access resources for resolved references and basic profile counters) 
- _Compilation state_ (code entry addresses, linking stubs and sophisticated profile counters).

**Why Java Class Metadata?** It's needed to model the code base at runtime; for the interpreter and JIT, so they know about the class organization; for reflection; and for the _JVM_ Tool Interface to query or update classes at runtime.

## Stack Memory

**Java Stack memory** is used for static memory allocation and during the execution of threads, it stores method specific values that are short-lived and references to other objects in the heap that are getting referred from the method. 

Whenever a method is invoked, a new block is created in the stack memory for the method to hold local primitive values and reference to other objects in the method. As soon as method ends, the block becomes unused and become available for next method.

Features:

- Access to this memory is in _LIFO_ (last-in-first-out).
- Size depends on the OS and is relatively small compared to Heap memory.
- This memory exists as long as the method is running.
- When it's full it throws a `java.langStackOverFlowException`.
- Access to this memory is faster than heap memory.

## Code Cache

**Code Cache** is the memory area where the _JVM_ stores the compiled code generated by the _Just-In-Time_ (_JIT_) compiler. *Code Cache* uses native memory and is managed by the *Code Cache Sweeper*.

### Code Cache Tuning

You can use _JVM_ option to tweak the *codecache* consumption by the _JIT_.

Some common *Codecache* **options**:

Option | Default | Description 
---------|----------|--------- 
`InitialCodeCacheSize` | 160K (varies) | Initial code cache size (in bytes) 
`ReservedCodeCacheSize` | 32M/48M | Reserved code cache size (in bytes) - maximum code cache size 
`CodeCacheExpansionSize` | 32K/64K | Code cache expansion size (in bytes) 
`UseCodeCacheFlushing` | false | Attempt to sweep the codecache before shutting off compiler 
`CodeCacheMinimumFreeSpace` | 500K | When less than the specified amount of space remains, stop compiling. This space is reserved for code that is not compiled methods, for example, native adapters 
`PrintCodeCache` | false | Print the code cache memory usage when exiting 

You can **benefit** from codecache restriction when a lot of code is stored during startup, but very little of this compiled code is needed afterward. By doing this codecache flushing will be likely triggered and make room for the code needed during runtime.
