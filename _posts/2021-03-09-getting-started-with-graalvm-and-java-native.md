---
title: Getting Started with GraalVM and Java Native
image: https://lh3.googleusercontent.com/pw/ACtC-3fm450j3RMNL8j8gbPO4tr5uyD1lCD3c11td_W4_g0rtOl51ANDICTXnx16AqdJCqdRTSQjfIgfjD47L_GhC8tf5BgMgcMb1O8OFe5k6UaoCfbHzQ2IsSYz5QGizGAVmFUlRVY9skU3CSFMV7tbfzhI=w640-h426-no?authuser=1
author: Sergio Martin Rubio
categories:
    - DevOps
mermaid: false
layout: post
---

[GraalVM](https://www.graalvm.org){:target="_blank"} is one of the virtual machines to run applications written in multiple languages like JavaScript, Python or Java. One of the main features of GraalVM is [Native Images](https://www.graalvm.org/reference-manual/native-image/){:target="_blank"}. GraalVM allows you to generate a native image of your Java code, so it does not need to run on the JVM, and includes all the necessary componentes.

## Build a Native Java Application

### Prerequisites

- **Install GraalVM**. For Mac OS users [SdkMan](https://sdkman.io){:target="_blank"} provides GraalVM images.

  ```shell
  sdk install java 21.0.0.2.r11-grl
  ```

- **Install the GraalVM native tool**.

  ```shell
  gu install native-image
  ```

- For Mac OS users **install Xcode tools** if you don't have them yet.

  ```shell
  xcode-select --install
  ```

### Hello World Application

Now you can write your first GraalVM application.

```java
public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

and run:

```shell
javac HelloWorld.java
native-image HelloWorld
```

During the compilation you will see something like this:

```shell
[helloworld:65139]    classlist:   3,096.34 ms,  0.96 GB
[helloworld:65139]        (cap):   3,308.79 ms,  0.96 GB
[helloworld:65139]        setup:   6,654.19 ms,  0.96 GB
[helloworld:65139]     (clinit):     233.40 ms,  1.20 GB
[helloworld:65139]   (typeflow):  14,796.92 ms,  1.20 GB
[helloworld:65139]    (objects):   6,133.03 ms,  1.20 GB
[helloworld:65139]   (features):     308.55 ms,  1.20 GB
[helloworld:65139]     analysis:  21,670.32 ms,  1.20 GB
[helloworld:65139]     universe:     782.71 ms,  1.20 GB
[helloworld:65139]      (parse):   4,390.33 ms,  1.20 GB
[helloworld:65139]     (inline):   1,680.80 ms,  1.44 GB
[helloworld:65139]    (compile):  17,090.49 ms,  1.89 GB
[helloworld:65139]      compile:  23,661.70 ms,  1.89 GB
[helloworld:65139]        image:   1,185.79 ms,  1.89 GB
[helloworld:65139]        write:     437.82 ms,  1.89 GB
[helloworld:65139]      [total]:  57,892.54 ms,  1.89 GB
```

The generation of the executable file will took around 1 minute on my machine (*MacBook Air (M1, 2020) - 16 GB*) in the first execution. Once it is finished it  generates a  `helloworld`  file that you can run from your terminal or by simply double clicking on it.

The generated file is 7.7MB, which is quite impressive for a Java application since this executable does need a JVM.

In the real world you might want to use something like [Kubernetes](https://kubernetes.io){:target="_blank"} or [EC2](https://aws.amazon.com/ec2/){:target="_blank"} to deploy your native Java application and you will see how to do it in the next section.

## Dockerized Java Native Application

### Prerequisites

- You need to have *[Docker](https://docs.docker.com/get-docker/){:target="_blank"}* installed on your machine to create *Docker* images.
- Bytecode class file `HelloWorld.class`: `javac HelloWorld.java`

### Getting Started

1. **Create a Java native executable in a Linux machine**. If you built the native image on a system other than Linux and try to use that image in a docker container with a *Linux* base image, it will not work! and you will get something like `standard_init_linux.go:219: exec user process caused: exec format error`, since the binaries generated by *GraalVM* are only compatible with the system where they are generated, this means if you are running *macOS*, binaries built on *macOS* won't work on a *Linux* distribution. The workourd for this is to **generate the binaries from a Linux host with Docker.** Steps:

   - Create a `Dockerfile` with GraalVM as the base image with `native-image` installed.

     ```dockerfile
     FROM ghcr.io/graalvm/graalvm-ce:latest
     WORKDIR /opt/graalvm
     RUN gu install native-image
     ENTRYPOINT ["native-image"]
     ```

   - Run the *GraalVM* image to generate a static native Java executable with Linux binaries.

     ```shell
     docker run -it -v <PATH_TO_JAVA_BYTECODE_CLASS_FILE>:/opt/cp -v <PATH_TO_OUTPUT_FOLDER>:/opt/graalvm graalvm-native-image HelloWorld
     ```

     where we are mounting two directories, one for the path where the Java bytecode class file is (`HelloWorld.class` in this case) and another one for the output executable file; and `HelloWorld` is the name of the entry class.

     After running this command you will get a `helloworld` file in the specified output folder.

     > You can also specify the output file name with `-H:Name=<name>`

2. **Build a Docker image for your Java executable file**.

   `Dockerfile`:

   ```dockerfile
   FROM debian:buster-slim
   COPY helloworld /opt/helloworld
   CMD ["/opt/helloworld"]
   ```

   and run:

   ```shell
   docker build . -t graalvm-hello-world
   ```

   This will create a **Docker image with a size of ~72MB**, **but we can do better than this!** We can compile a static native image that contains all the necessary resources, so we can use a smaller Docker base image.

   If we run the Docker command from the previous step with a `--static` flag it will bundle all the required binaries in the native image so the Docker container can run without any additional dependencies.

   ```shell
   docker run -it -v <PATH_TO_JAVA_BYTECODE_CLASS_FILE>:/opt/cp -v <PATH_TO_OUTPUT_FOLDER>:/opt/graalvm graalvm-native-image --static HelloWorld
   ```

   As a result the output native image will went up from 8.6MB to 9.8MB. 

   Now we can use a smaller base Docker image:

   ```dockerfile
   FROM scratch
   COPY helloworld /opt/helloworld
   CMD ["/opt/helloworld"]
   ```

   and after running again:

   ```shell
   docker build . -t graalvm-hello-world
   ```

   **you will get an image of size ~10MB**

3. Finally you can **run the Docker image**!

   ```shell
   time docker run --rm graalvm-hello-world
   ```

   Output:

   ```shell
   Hello, World!
   docker run graalvm-hello-world  0.17s user 0.08s system 14% cpu 1.758 total
   ```

   As you can see **it takes only 0.17s to run the native image**, and 1.7s in total (this is including the Docker overhead).

## Native Image Execution vs JVM Execution

Native images provide some benefits like a reduced executable size compared to a traditional Java application running in the JVM.

<canvas id="dockerImageSize" width="400" height="200"></canvas>

<script>
var ctx = document.getElementById("dockerImageSize");
var myChart = new Chart(ctx, {
    type: 'bar',
    data: {
        labels: ["Native Image", "JVM (8u131-jdk-alpine)"],
        datasets: [{
            label: 'Docker Image Size in MB',
            data: [10.2, 108],
            backgroundColor: [
                'rgba(255, 99, 132, 0.2)',
                'rgba(54, 162, 235, 0.2)',
            ],
            borderColor: [
                'rgba(255,99,132,1)',
                'rgba(54, 162, 235, 1)',
            ],
            borderWidth: 1
        }]
    },
    options: {
        scales: {
            yAxes: [{
                ticks: {
                    beginAtZero:true
                }
            }]
        }
    }
});
</script>

However, there is no difference in terms of execution time:

<canvas id="executionTime" width="400" height="200"></canvas>

<script>
var ctx = document.getElementById("executionTime");
var myChart = new Chart(ctx, {
    type: 'bar',
    data: {
        labels: ["Native Image", "JVM (8u131-jdk-alpine)"],
        datasets: [{
            label: 'Total Execution Time in Seconds',
            data: [1.8, 1.8],
            backgroundColor: [
                'rgba(255, 99, 132, 0.2)',
                'rgba(54, 162, 235, 0.2)',
            ],
            borderColor: [
                'rgba(255,99,132,1)',
                'rgba(54, 162, 235, 1)',
            ],
            borderWidth: 1
        }]
    },
    options: {
        scales: {
            yAxes: [{
                ticks: {
                    beginAtZero:true
                }
            }]
        }
    }
});
</script>

## Conclusion

Native images can provide benefits in terms of executable size, however realtime bytecode manipulation or reflection is problematic and it is a tradeoff that you need to consider before using GraalVM with Java.

{% include elements/button.html link="https://github.com/smartinrub/graalvm-hello-world.git" text="Examples" %}
