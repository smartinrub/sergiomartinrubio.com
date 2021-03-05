---
title: Introduction to Message-Driven Beans
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java Framework
    - Messaging
mermaid: false
layout: post
---

## Introduction

A **MDB** (*Message-Driven Bean*) is a bean used by [JMS](https://sergiomartinrubio.com/articles/understanding-messaging-pattern-with-jms) to listen to new asynchronous messages. In the same way as any other Java EE enterprise bean, the [EJB](https://sergiomartinrubio.com/articles/ejb-what-it-is-why-it-exists-and-how-it-works) container in which the MDB runs takes of the bean lifecycle so you do not have to configure a listener by yourself and add all the required boilerplate.

The main difference between regular session beans and MDBs is that clients don't access message-driven beans through interfaces, instead MDBs are listening to messages published by a JMS application.

## MDB Class

MDBs are regular Java classes that implement `javax.jms.MessageListener`  and are annotated with `@MessageDrive`. The `MessageListener` requires that you implement the `onMessage()` method that is invoked when a message arrives, so we can process the message. You will have to cast the message to one of the five JMS message types (`BytesMessage`, `MapMessage`,  `ObjectMessage`, `StreamMessage`, `TextMessage`).

Messages are delivered in a transaction context, in other words, if for some reason an exception is thrown, the message will be redelivered.

Message-driven bean classes allows you to set configuration properties to specify things like destination name and destination type. You can define a `@ActivationConfigProperty ` as a value of the parameter `activationConfig` that is accepted by `@MessageDrive` to set `destination` and `destinationType` . i.e.

```java
@MessageDriven(activationConfig = {
        @ActivationConfigProperty(propertyName = "destination", propertyValue = "myQueue"),
        @ActivationConfigProperty(propertyName = "destinationType", propertyValue = "javax.jms.Queue")
})
public class MyMessageDrivenBean implements MessageListener {

    private static final Logger LOGGER = Logger.getLogger(MyMessageDrivenBean.class.getName());

    @Override
    public void onMessage(Message message) {
        TextMessage textMessage = (TextMessage) message;

        try {
            LOGGER.info("Message received: " + textMessage.getText());
        } catch (JMSException e) {
            LOGGER.severe("Something went wrong when consuming message: " + e.getMessage());
        }
    }
}
```

> The destination type can also be a topi if you specify as the property value `javax.jms.Topic`.

## Other Configuration Properties

| Property Name              | Property Value               | Description                                                  |
| -------------------------- | ---------------------------- | ------------------------------------------------------------ |
| `acknowledgeMode`          | `AUTO_ACKNOWLEDGE` <br />(default) | An acknowledge message <br />is sent automatically                 |
| `acknowledgeMode`          | `DUPS_OK_ACKNOWLEDGE`        | An acknowledgment message <br />is not sent to the client right away. <br />This is good for performance. |
| `acknowledgeMode`          | `CLIENT_ACKNOWLEDGE`         | The client is responsible <br />for manually acknowledge the message |
| `subscriptionDurability`   | `NonDurable` <br />(default)       | If nobody is listening to <br />the topic the message is missed    |
| ``subscriptionDurability`` | `Durable`                    | Ensures that messages sent to <br />a topic are not missed         |

## Lifecycle Callbacks and Interceptors

MDBs are compatible with enterprise bean [lifecycle methods](https://sergiomartinrubio.com/articles/ejb-what-it-is-why-it-exists-and-how-it-works#session-bean-lifecycle-methods):

- `@PostConstruct`: the method is invoked before the first message is received by the MDB.
- `@PreDestroy`:  the method is invoked when the MDB is removed from the pool or destroyed.

[Interceptors](https://sergiomartinrubio.com/articles/ejb-what-it-is-why-it-exists-and-how-it-works#session-bean-interceptor) are also allowed. We can define our interceptor  a method annotated with `@AroundInvoke` . These methods will be invoke before and after the execution of the `onMessage()` method.

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/message-driven-bean-example.git" text="Examples" %}
</p>

