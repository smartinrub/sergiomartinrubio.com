---
title: Java Socket IO and NIO
image: https://lh3.googleusercontent.com/RwUgHOsiSrdD-MxvWTTcmXBjtZkBrjzoZGSnGFDEHkhyZGnc0nbijrtf29-jbGkShDj8FhWx4waaPDP6IvWeUlJWl4z-to3UtAvKn-wG5j20_UZaGWqfCYZijH0gcKKg-bG07vqGXkMvzhqYVznIliUaGRts2OSHyjfs_AWJ6ODzgFPhOQD2H-9Ix-CVipzDHrf9DGcccwU1K56eI9PNo8Kk493teJQrj3-wrqYDj8AcwqlcUXaA6JALOMJwozGLKuX37o1gefy3t3FL6DgTZzS6LqyqmQl0mrnyMUc6jWGTmJB33ioECfjolF79-zDl26SCIeNoxCK-CRJrZhm5FzJ1fLCXIRVh09xSaDfe1rMIM5jtl88EMNuC3eEKGwa7ikFOq2uUt9Pwjhf-zxYqXa9aqVwv98Xp2Eb1PQMeGJs-jY7KBq2l-tde5PEfClgKAxa8LN2zyCPzLjfrO5B2jGIyRIuKdl7fGc6CZyic2MkJrzl6jY4DjMcSfjZLEaq4GcvaVEeMwMgr-72ib1X99sihYLC1eZwKqiVAChsr8XtbVHzAIDD8k5MVeE1UZmnpkwnNn4asg7F66uX3KQ3jM5xzAOpLRXLKm3G_NhpJ2ElNKfUmre4aSk2NErlvQwJ3YKwg5anEtQ2i0Tk02aO1J-ovBGBtWL_Pwpkf1K6V5KvegQhbIYjhpFxEI0Nd=w2462-h1640-no?authuser=0
author: Sergio Martin Rubio
categories:
    - Java
mermaid: false
layout: post
---

## Introduction

Sockets use _TCP/IP_ transport protocol and they are the last piece of a network communication between two hosts. You do not usually have to deal with them, since there are protocols built on top of sockets like _HTTP_ or _FTP_, however it is important to know how they work.

>_TCP_: It is a reliable data transfer protocol that ensures that the data sent is complete and correct and requires to establish a connection.

_Java_ offers a blocking and non blocking alternative to create sockets, and depending on your requirements you might consider the one or the other.

## Java Blocking IO

The _Java_ blocking IO API is included in **JDK** under the package `java.net` and is generally the simplest to use.

This API is based on flows of byte streams and character streams that can be read or written. There is not an index that you can use to move back and forth, like in an array, it is simply a continuous flow of data.

{% include elements/figure.html image="https://lh3.googleusercontent.com/B5e8q-Kn1kzI_apnfLbX8n2abY-uJzTzaFevpdr7ewQBarkSDut0zdpDQeqVUo6cPAqTieIa9S8U0GVgB7DMPHqPU3n386ZIM5g_KzZktCj0iCTn7tsUZxubg4ESaEIwNShIPoXiuw=w600" caption="Java Blocking IO" %}

Every time a client requests a connection to the server, it will block a thread. Therefore, you have to create a pool of threads large enough if you expect to have many simultaneous connections.

```java
ServerSocket serverSocket = new ServerSocket(PORT_NUMBER);

while (true) {
    Socket client = serverSocket.accept();
    try {
        BufferedReader in = new BufferedReader(
            new InputStreamReader(client.getInputStream())
        );
        OutputStream out = client.getOutputStream();
        in.lines().forEach(line -> {
            try {
                out.write(line.getBytes());
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
        client.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

1. A `ServerSocket` is created with a given port to listen on.
2. The server will block when `accept()` is invoked and starts listening for clients connections.
3. If a client requests a connection a `Socket` is returned by `accept()`.
4. Now you can read from the client (`InputStream`) and send data back to the client (`OutputStream`).

If you want to allow multiple connections, you have to create a **Thread Pool**:

```java
ExecutorService threadPool = Executors.newFixedThreadPool(100);

 threadPool.execute(() -> {
     // SOCKET CREATION
 });
```

As you can see, this API has some limitations. You will not be able to accept more connections than threads available in you machine. Therefore, if you are expecting to have many connections, you need an alternative.

## Java NIO

**java.nio** is a non blocking API for socket connections which means you are not tight to the number of threads available. With this library one thread can handle multiple connections at once.

{% include elements/figure.html image="https://lh3.googleusercontent.com/UPsm3Jc2Gicv6fHuIqnSjOrwvXhO73u5bDYcWMU2WtuCKM9Q6ePPEGoJPKxKA0dl9DQwrkr5B3YNcQ505xgQUtwZB-jKnSx3uetK0bkRK01g9S1lsWWAPZ-hSfVfeP0ZpvL7ap3RrA=w600" caption="Java NIO" %}

**Elements**:

- **Channel**: channels are a combination of input and output streams, so they allow you to read and write, and they use buffers to do this operations.
- **Buffer**: it is a block of memory used to read from a `Channel` and write into it. When you want to read data from a `Buffer` you need to invoke`flip()`, so that it will set `pos` to 0.

    ```java
    int read = socketChannel.read(buffer); // pos = n & lim = 1024
    while (read != -1) {
        buffer.flip(); // set buffer in read mode - pos = 0 & lim = n
        while(buffer.hasRemaining()){
            System.out.print((char) buffer.get()); // read 1 byte at a time
        }
        buffer.clear(); // make buffer ready for writing - pos = 0 & lim = 1024
        read = socketChannel.read(buffer); // set to -1
    }
    ```

    1. On line 1, pos will be equals to the number of bytes written into the `Buffer`.
    2. On line 3, `flip()` is called to set position to 0 and limit to the number of bytes previously written.
    3. On line 5, it reads from `Buffer` one byte at a time up to the limit.
    4. On line 7, finally it clears the `Buffer`.

- **Selector**: A `Selector` can register multiple Channels and will check which ones are ready for accepting new connections. Similar to `accept()` method of blocking IO, when `select()` is invoked it will block the application until a `Channel` is ready to do an operation. Because a `Selector` can register many channels, only one thread is required to handler multiple connections.
- **Selection Key**: It contains properties for a particular **Channel** (interest set, ready set, selector/channel and an optional attached object). Selection keys are mainly use to know the current interest of the channel (`isAcceptable()`, `isReadable()`, `isWritable()`), get the channel and do operations with that channel.

### Example

You will use an **Echo Socket Channel** server to show how _NIO_ works.

```java
var serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.configureBlocking(false);
serverSocketChannel.socket().bind(new InetSocketAddress(8080));

var selector = Selector.open();
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

while (true) {
    selector.select();
    var keys = selector.selectedKeys().iterator();

    while (keys.hasNext()) {
        var selectionKey = (SelectionKey) keys.next();

        if (selectionKey.isAcceptable()) {
            createChannel(serverSocketChannel, selectionKey);
        } else if (selectionKey.isReadable()) {
            doRead(selectionKey);
        } else if (selectionKey.isWritable()) {
            doWrite(selectionKey);
        }
        keys.remove();
    }
}
```

1. From lines 1 to 3 a `ServerSocketChannel` is created, and you have to set it to non-blocking mode explicitly. The socket is also configure to listen on _port 8080_.
2. On line 5 and 6, a `Selector` is created and `ServerSocketChannel` is registered on the `Selector` with a `SelectionKey` pointing to ACCEPT operations.
3. To keep the application listening all the time the blocking method `select()` is inside an infinite while loop, and `select()` will return when at least one channel is selected `wakeup()` is invoked or the thread is interrupted.
4. Then on line 10 a set of keys are returned from the `Selector` and it will iterate through them in order to execute the ready channels.

```java
private static void createChannel(ServerSocketChannel serverSocketChannel, SelectionKey selectionKey) throws IOException {
    var socketChannel = serverSocketChannel.accept();
    LOGGER.info("Accepted connection from " + socketChannel);
    socketChannel.configureBlocking(false);
    socketChannel.write(ByteBuffer.wrap(("Welcome: " + socketChannel.getRemoteAddress() +
            "\nThe thread assigned to you is: " + Thread.currentThread().getId() + "\n").getBytes()));
    dataMap.put(socketChannel, new LinkedList<>()); // store socket connection
    LOGGER.info("Total clients connected: " + dataMap.size());
    socketChannel.register(selectionKey.selector(), SelectionKey.OP_READ); // selector pointing to READ operation
}
```

5. Every time a new connection is created `isAcceptable()` will be true and a new `Channel` will be registered into the `Selector`.
6. To keep track of the data of each channel, it is put in a `Map` with the socket channel as the key and a list of `ByteBuffers`.
7. Then the selector will point to _READ_ operation.

```java
private static void doRead(SelectionKey selectionKey) throws IOException {
    LOGGER.info("Reading...");
    var socketChannel = (SocketChannel) selectionKey.channel();
    var byteBuffer = ByteBuffer.allocate(1024); // pos=0 & lim=1024
    int read = socketChannel.read(byteBuffer); // pos=numberOfBytes & lim=1024
    if (read == -1) { // if connection is closed by the client
        doClose(socketChannel);
    } else {
        byteBuffer.flip(); // put buffer in read mode by setting pos=0 and lim=numberOfBytes
        dataMap.get(socketChannel).add(byteBuffer); // find socket channel and add new byteBuffer queue
        selectionKey.interestOps(SelectionKey.OP_WRITE); // set mode to WRITE to send data
    }
}
```

8. In the read block the channel will be retrieved and the incoming data will be written into a `ByteBuffer`.
9. On line 6 it checks if the connection has been closed.
10. On line 9 and 10, the buffer is set to read mode with `flip()` and added to the `Map`.
11. Then, `interestOps()` is invoked to point to _WRITE_ operation.

```java
private static void doWrite(SelectionKey selectionKey) throws IOException {
    LOGGER.info("Writing...");
    var socketChannel = (SocketChannel) selectionKey.channel();
    var pendingData = dataMap.get(socketChannel); // find channel
    while (!pendingData.isEmpty()) { // start sending to client from queue
        var buf = pendingData.poll();
        socketChannel.write(buf);
    }
    selectionKey.interestOps(SelectionKey.OP_READ); // change the key to READ
}
```

12. Once again, the channel is retrieved in order to write the data saved in the `Map` into it.
13. Then, it sets the `Selector` to _READ_ operations.

```java
private static void doClose(SocketChannel socketChannel) throws IOException {
    dataMap.remove(socketChannel);
    var socket = socketChannel.socket();
    var remoteSocketAddress = socket.getRemoteSocketAddress();
    LOGGER.info("Connection closed by client: " + remoteSocketAddress);
    socketChannel.close(); // closes channel and cancels selection key
}
```

14. In case the connection is closed, the channel is removed from the `Map` and it closes the channel.

## Java IO vs NIO

Choosing between _IO_ and _NIO_ will depend on the use case. For fewer connections and a simple solution, _IO_ might be better fit for you. On the other hand, if you want something more efficient which can handle thousands of connections simultaneously _NIO_ is probably a better choice, but bear in mind that it will add much code complexity, however, there are frameworks like [Netty](https://netty.io/){:target="_blank"} or [Apache MINA](https://mina.apache.org/){:target="_blank"} that are built on top of _NIO_ and hide the programming complexity.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-sockets" text="Source Code" %}
</p>
