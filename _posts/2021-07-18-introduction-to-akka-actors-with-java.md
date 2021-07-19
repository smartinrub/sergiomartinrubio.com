---
title: Introduction to Akka Actors with Java
image: https://lh3.googleusercontent.com/pw/AM-JKLUd9_b3L_WRwvS_8t9Ozz1p3MP_paj_o27YGe4gD6kWN5uwLpNg5iUcsjcaKNO0SlrLzKzWezdxSqrUhm-Zo8bxvzDqbcJCYfmbFitdaWgwHhOY2coiPeop2tBnvp4I1lJu6tu2cJ4MkvsZ2Q7xAwCe=w2858-h1640-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java Framework
mermaid: false
layout: post
---

[Akka](https://akka.io){:target="_blank"} is a library for Java and Scala that allows you to develop applications that involve running asynchronous processes in distributed systems.

This open source library supports clustering and reactive streams, as well as, it's easy to maintain and performs very well.

Akka makes use of something called the *actor model*. This model solves some issues that comes with OOP (Object Oriented Programming) like concurrency, since *OOP* languages were not designed for that and it is easy to introduce race conditions. On the other hand, the actor model do not share any state between actors and it follows a "fire and forget" strategy (similar to a [message queue system](https://sergiomartinrubio.com/articles/understanding-messaging-pattern-with-jms/)), therefore it works better on distributed systems. However, the actor model has **some drawbacks**: it's harder to get "return values"; it might add more complexity.

Actors can receive and send messages, this means each actor can delegate work to other actors. When a return value is expected, the actor delivers the results in a reply message.

# Getting Started

The following examples will show you how to start creating Akka actors with Java.

First of all, we need to add one dependency to our `pom.xml` file.

```xml
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor-typed_2.13</artifactId>
    <version>${akka.version}</version>
</dependency>
```

There are two versions of the Akka actors API: **Classic** and **Typed**

## Classic Actor

To create your first *Classic* actor you just need to create a class that inherits from `AbstractActor` and override the method `createReceive()`. This method will be the entry point of the actor and is used to setup the behavior of the actor. Within the `createReceive()` we can have multiple matchers for different *types* of messages. We can match a particular Java *class* with `match()`, the exact content of the message with `matchEquals()` or fallback to a default handler in case there is not match for the message (`matchAny()`). An alternative way of handling unknown messages is to override the `unhandled()` method.

Akka actors has its own lifecycle that you can tweak as you wish by overriding the following methods:

- `preStart()`: Invoked right after the actor starts. Executed only once when the actor is first created if `postRestart()` is not overridden.
- `preRestart()`: Invoked when an exception happens while processing the message. Good for clean up. `postStop()` is called afterward.
- `postRestart()`: The new actor invokes this method after the restart. This method calls `preStart()` of the new actor.
- `postStop()`: This is called after stopping an actor. Messages sent to a stopped actor are sent to dead letters. You can use it for cleaning up resources.

Some considerations while a restart is happening are:

- The message that caused a restart is lost.
- Messages sent to the actor while it is restarting are enqueued.
- Messages already enqueued to be consumed by the actor are not lost.

```java
public class FirstClassicActor extends AbstractActor {

    private FirstClassicActor() {
    }

    public static record FooMessage(String param) {

    }

    public static record BarMessage(String param) {
    }

    static Props props() {
        return Props.create(FirstClassicActor.class, FirstClassicActor::new);
    }

    @Override
    public Receive createReceive() {
        return receiveBuilder()
                .match(FooMessage.class, message -> System.out.println("Received Foo message: " + message.param))
                .match(BarMessage.class, message -> System.out.println("Received Bar message: " + message.param))
                .matchEquals("secret-message", message -> System.out.println("This is a secret message: " + message))
                .matchAny(message -> System.out.println("Received unknown message: " + message))
                .build();
    }

    @Override
    public void unhandled(Object message) {
        System.out.println("Unknown message: " + message.toString());
        super.unhandled(message);
    }

    @Override
    public void preStart() throws Exception {
        super.preStart();
    }

    @Override
    public void preRestart(Throwable reason, Optional<Object> message) throws Exception {
        super.preRestart(reason, message);
    }

    @Override
    public void postRestart(Throwable reason) throws Exception {
        super.postRestart(reason);
    }

    @Override
    public void postStop() throws Exception {
        super.postStop();
    }
}
```

> it's a good practice to define the messages as inner classes

You can invoke an actor as follows:

1. Create actor system.
2. Register actor into actor system. We can have multiple top level actors defined for a single system.
3. Send messages to actor.
4. (Optional) Stop the actor, so no more messages will be received by the actor.

> Important: actors are stateful resources and has to be manually stopped, otherwise they will be loaded forever.

```java
ActorSystem system = ActorSystem.create("my-actor-system");
Props actor = FirstClassicActor.props();
ActorRef actorRef = system.actorOf(actor, "my-actor");

System.out.println("Start sending messages...");
actorRef.tell(new FirstClassicActor.BarMessage("message"), ActorRef.noSender());
actorRef.tell("hello world", ActorRef.noSender());
actorRef.tell(new FirstClassicActor.FooMessage("another-message"), ActorRef.noSender());
actorRef.tell("secret-message", ActorRef.noSender());
System.out.println("Done!");
system.stop(actorRef);
```

## Typed Actor

The Typed Actor API is the new API of the Akka library and the creation and execution of actors is slightly different. Also, actors are structured in a different way. A top level actor is define for a particular *actor system*, and children actors are created by the top level actor called *guardian actor*. Alternatively, actors can also be created per HTTP request.

The lifecycle of Typed Actors is handled in a different manner, instead of overriding method we can trigger different behaviors based on the signal received by the actor.

```java
public class FirstTypedActor extends AbstractBehavior<FirstTypedActor.FooMessage> {

    private final ActorRef<ChildTypedActor.BarMessage> childTypedActor;

    public static record FooMessage(String message) implements SpawnProtocol.Command {

    }

    private FirstTypedActor(ActorContext<FooMessage> context) {
        super(context);

        childTypedActor = context.spawn(ChildTypedActor.create(), "child-typed-actor");
    }

    public static Behavior<FooMessage> create() {
        return Behaviors.setup(FirstTypedActor::new);
    }

    @Override
    public Receive<FooMessage> createReceive() {
        return newReceiveBuilder()
                .onMessage(FooMessage.class, this::onFooMessage)
                .onSignal(PreRestart.class, this::onPreRestart)
                .onSignal(Terminated.class, this::onTerminated)
                .onSignal(PostStop.class, this::onPostStop)
                .build();
    }

    private FirstTypedActor onFooMessage(FooMessage message) {
        System.out.println("Received Foo message: " + message.message);
        childTypedActor.tell(new ChildTypedActor.BarMessage("hello from top level actor"));
        return this;
    }

    private FirstTypedActor onPreRestart(PreRestart preRestart) {
        System.out.println("Job is about to restart.");
        return this;
    }

    private FirstTypedActor onTerminated(Terminated terminated) {
        System.out.println("Job" + terminated.getRef().path().name() + "stopped.");
        return this;
    }

    private FirstTypedActor onPostStop(PostStop postStop) {
        System.out.println("Job is stopped.");
        return this;
    }

}
```

> Default constructor is required for Typed actors.

As you can see in the example above, children actors can be created from within the guardian actor. You would usually create new actors either in the constructor on when a message is received.

You can invoke an actor as follows:

1. Create actor system and top level actor.
2. Send message to guardian actor.
3. (Optional) Shutdown actor.

```java
ActorSystem<FirstTypedActor.FooMessage> system = ActorSystem.create(FirstTypedActor.create(), "typed-actor-system");
system.tell(new FirstTypedActor.FooMessage("hello world"));
system.terminate();
```

# Conclusion

In this article we only covered the basic features of Akka actors and this library provides many other features that can be found in the [official documentation](https://doc.akka.io/docs/akka/current/index.html){:target="_blank"}.

Akka is a powerful messaging framework to make multithreading easy to implement and is an ideal solution to scale up I/O communication in a distributed system.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/akka-java-example.git" text="Source Code" %}
</p>

Photo by [Jan Huber](https://unsplash.com/@jan_huber?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/components?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)
