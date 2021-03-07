---
title: Introduction to Message-Driven Beans
image: https://lh3.googleusercontent.com/8flZCr8BsqV3J3frpifp5W9cjU1V4u3umDq7uHC7T5nkNP4B14zTHHQ2_hsTllyOeHBzzmr16K4L3_cA859cuWTt-aUwxZyNBt4yzr1eFJr0-WZUylk4nqEhsyES7_KdJE5pKup-Balqxrcq8eu3qz4EMzsTzCoOnlmqNhNBreKcIGDY3L-WYNWJPPUGhVAVGR5aSTen1WuR-xIlJI5UvmkQLymhXSyT2pCPy82IgXHcyXtURYDbzOJMj2X27Ad6bVB6Pi7TO0OykwIOxo6Fq2dlTpJfqjv2w9Fg-yXsoovahEnfCvVbnKRSNypwoEfPT8acXnU0RRaChcRjEt15w8WTjFDzPDz8ugCB0BrxxeT57eE46SK8-vXfU9YYOfJYP86DEFLDsFglVhBdAChv9WLk9qHnACgz7uXqTxVAsmjrx8CXeyax8GyfEHiZPMFyWzIFVknFKzLp1lYnKKhlNPoo_afcdAFVg5FYjofUG-H2cnowgRoIjbFUI9kd7XFQPVe-n8iqIdRyB4dUSMOtfUOKLCBbn4RNHIoOnaLdNa8EnAG1qikglwbNqmsszrOPbF4tqGWmuND7Mx-8AlxOqi5VAVsnMQfiovTCQkiOFBelXPBMK4-Id67KqVcTG17HjYOLs7qUij4R8p2askuZiqhtINzGKmroRcdeBdmXnrtkQbXMppiDIzAK5D0m=w459-h309-no?authuser=0
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

Image by <a href="https://pixabay.com/users/atlantios-4957810/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3437294">Antonios Ntoumas</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=3437294">Pixabay</a>
