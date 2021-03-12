---
title: Understanding Messaging Pattern with JMS
image: https://lh3.googleusercontent.com/pw/ACtC-3cah-UPJeR4OS9_0dNeZ2dbXOc8acWHWODWLCD8smM43Oy0wghvA62RwPqNKqbCBzQV4w1uysqHyPe_Yc2hcsdMI41Wg5blzpsmeT8ZPB5pOqzvnt7TkY0Vjg83UxniRkGfkeWJ9jUrbqc92-Bhhf6H=w640-h404-no?authuser=1
author: Sergio Martin Rubio
categories:
    - Java Framework
    - Messaging
mermaid: false
layout: post
---

The message design pattern is very common nowadays in distributed systems to decouple applications into smaller components, and it brings better performance, increased reliability and asynchronous communication.

## Types Of Messaging Techniques

There are two types of messaging techniques:

- **Point-to-Point** model.
- **Publisher/Subscriber** model

The differences are:


| Feature | Point-to-Point | Publishes/Subscriber |
| - | - | - |
| **Middleware** | Queue | Topic |
| **Timing** | No timing dependency | Timing dependency |
| **Consumers** | Single consumer | Multiple consumers |
| **Mechanism** | Pull | Push |
| **Persistence** | Yes | No |
| **Ordered** | Messages are consumed in order | Messages are NOT consumed in order |
| **Destination** | Known | Unknown |

{% include elements/figure.html image="https://lh3.googleusercontent.com/6aMxwceKlc4JYyfF99FVS0Z9rUiPe8MIkc1N4cXP1onbvYuw5Qd2KoQdXl4-IPpemVrPRVx6rVKHCc48n_a9lQLJmFbjU9HTSd5glhEUOHBQ1h_c2ks0P52_zsoZOA4jnrm4ymb6gA=w800" caption="Point-to-Point Diagram" %}

In a **point-to-point** model messages are usually stored in a staging area where they are waiting to be consumed by a single listener. Consumers use a pull mechanism to get the messages from the queue in order, so the listener does not need to be online all the time to receive or read messages from the queue. This model is usually used to decouple two applications and allow asynchronous processing.

{% include elements/figure.html image="https://lh3.googleusercontent.com/tZcegl5e9NhUZQClD25OTwSF2G3dcwK-3iyrRvxEAoxYoL3e1IMfUFTT4MOdU93GfI2IG8S81D0df1NFDAJBTtOEITBPkJvm3qipzeZz2-6yl8pHENNwBtQ4Kphiipbm5JNr4QufRw=w800" caption="Publisher/Subscriber Diagram" %}

On the other hand, a **publisher/subscriber** model does not store messages, and as a result if no consumers are available, the published message is lost, so it requires that the consumer is present at the time the message is ready to be delivered, unless it has durable subscription for inactive consumers. This model allows multiple clients to subscribe to a topic, but there is no guarantee that the messages are delivered in order. This technique is usually used in a fan-out strategy when we want to send a message to multiple applications.

## JMS (Java Message Service)

_JMS_ allows _Java_ applications to communicate with messaging systems through a set of interfaces. _JMS_ supports both messaging model, _point-to-point_ and _publisher/subscriber_.

### JMS Model

The JMS model consists of:

- **Provider**: _JMS_ system that implements the _JMS_ specification.
- **Clients**: _Java_ applications that send and receive messages.
- **Administered objects**: Preconfigured _JMS_ objects that are created by an administrator for the use of _JMS_ clients.
- **Messages**: objects that are sent are received and contain _header_, _properties_ and _body_:
  - **Header**: it's mandatory and contains some metadata like priority, correlation ID, expiration or destination.
  - **Properties**: they are optional and you can set attributes by adding the prefix `.jmx`.
  - **Body**: the body is also optional and contains the data we want to exchange. JMS defines six different message types: `Message`, `StreamMessage`, `MapMessage`, `TextMessage`, `ObjectMessage`, `BytesMessage`.

### JMS Architecture

The steps for producing and consuming messages are:

1. Create a connection through the `ConnectionFactory`
2. Create a session
3. Create Message
4. Use a producer or consumer to send message to destination or to receive message from destination

### Java Implementation with ActiveMQ

Before getting started you need to spin up a standalone ActiveMQ server:

1. [Download ActiveMQ](http://activemq.apache.org/components/classic/download/){:target="_blank"}
2. Unzip the file and go to `apache-activemq-<version>/bin`
3. Execute `./activemq start`

Now you can access the ActiveMQ dashboard: http://localhost:8161/admin/

> You can stop the ActiveMQ Server at any time by executing `./activemq stop`.

Also, assuming you have a Maven project you will need to add the following dependencies.

```xml
<dependency>
    <groupId>javax.jms</groupId>
    <artifactId>javax.jms-api</artifactId>
    <version>2.0.1</version>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.13</version>
</dependency>
```

`javax.jms-api` is required to start a new context and perform naming operations. `activemq-all` includes all the dependencies required to talk to **ActiveMQ**.

#### Point to Point Implementation

You can create a producer as follows:

```java


public class Producer {

    private static final Logger LOGGER = Logger.getLogger(Producer.class.getName());

    public static void main(String[] args) throws JMSException, NamingException {
        // log4j configuration
        BasicConfigurator.configure();

        // Obtain a JNDI connection
        Properties properties = new Properties();
        properties.setProperty(INITIAL_CONTEXT_FACTORY, "org.apache.activemq.jndi.ActiveMQInitialContextFactory");
        properties.setProperty(PROVIDER_URL, DEFAULT_BROKER_URL);
        properties.setProperty("queue.MyQueue", "example.MyQueue");
        InitialContext jndi = new InitialContext(properties);

        // Look up a JMS connection factory
        ConnectionFactory connectionFactory = (ConnectionFactory) jndi.lookup("ConnectionFactory");

        try (Connection connection = connectionFactory.createConnection()) {
            connection.start();

            // Create session to send a receive messages. Set the first parameter to true
            // if you want to allow transactions
            Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

            Destination destination = (Destination) jndi.lookup("MyQueue");

            // To send messages
            MessageProducer producer = session.createProducer(destination);
            TextMessage message = session.createTextMessage("Hello World!");
            producer.send(message);
            LOGGER.info("Message " + message.getText() + " was sent!");
        }
    }
}
```

> The default ActiveMQ broker URL is `tcp://localhost:61616`.

The context properties can be also define in `resources/jndi.properties`.

```
java.naming.factory.initial = org.apache.activemq.jndi.ActiveMQInitialContextFactory
java.naming.provider.url = tcp://localhost:61616
queue.MyQueue = example.MyQueue
topic.MyTopic = example.MyTopic
```

and the consumer will be:

```java
public class Consumer {

    private static final Logger LOGGER = Logger.getLogger(Producer.class.getName());

    public static void main(String[] args) throws JMSException {
        // log4j configuration
        BasicConfigurator.configure();

        // Getting JMS connection from the server
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(DEFAULT_BROKER_URL);

        try (Connection connection = connectionFactory.createConnection()) {
            connection.start();

            // Create session for receiving messages
            Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

            // Getting the queue
            Destination destination = session.createQueue("example.MyQueue");
            MessageConsumer consumer = session.createConsumer(destination);
            // This blocks indefinitely until a message is produced
            Message message = consumer.receive();

            if (message instanceof TextMessage) {
                TextMessage textMessage = (TextMessage) message;
                LOGGER.info("Receive message: " + textMessage.getText());
            }
        }

    }
}
```

Alternatively, you can use `ActiveMQConnectionFactory` instead of manually creating the context as you can see on the consumer.

#### Publisher Subscriber Implementation

You can create a publisher as follows:

```java
public class Publisher {

    private static final Logger LOGGER = Logger.getLogger(Producer.class.getName());

    public static void main(String[] args) throws NamingException, JMSException {
        // log4j configuration
        BasicConfigurator.configure();

        // Obtain a JNDI connection
        InitialContext jndi = new InitialContext();

        // Look up a JMS connection factory
        TopicConnectionFactory connectionFactory = (TopicConnectionFactory) jndi.lookup("TopicConnectionFactory");

        // Create a JMS connection and start the JMS connection; allows messages to be received
        TopicConnection connection = connectionFactory.createTopicConnection();
        connection.start();

        // Create JMS session publisher
        TopicSession publisherSession = connection.createTopicSession(false, Session.AUTO_ACKNOWLEDGE);

        // Look up a JMS topic
        Topic topic = (Topic) jndi.lookup("MyTopic");

        // Create JMS publisher
        TopicPublisher publisher = publisherSession.createPublisher(topic);

        // Create and send message using topic publisher
        TextMessage message = publisherSession.createTextMessage("How are you my friend?");
        publisher.publish(message);
        LOGGER.info("Message " + message.getText() + " was published to topic " + topic.getTopicName());
    }
}
```

and a subscriber like this:

```java
public class Subscriber {

    public static void main(String[] args) throws NamingException, JMSException {
        // log4j configuration
        BasicConfigurator.configure();

        // Obtain a JNDI connection
        InitialContext jndi = new InitialContext();

        // Look up a JMS connection factory
        TopicConnectionFactory connectionFactory = (TopicConnectionFactory) jndi.lookup("TopicConnectionFactory");

        // Create a JMS connection and start the JMS connection; allows messages to be delivered
        TopicConnection connection = connectionFactory.createTopicConnection();
        connection.start();

        // Create JMS session publisher
        TopicSession subscriberSession = connection.createTopicSession(false, Session.AUTO_ACKNOWLEDGE);

        // Look up a JMS topic
        Topic topic = (Topic) jndi.lookup("MyTopic");

        // Create a JMS subscriber
        TopicSubscriber subscriber = subscriberSession.createSubscriber(topic);

        // Set a JMS message listener
        subscriber.setMessageListener(new MyListener());
    }

}
```

You also need to define a listener implementing `MessageListener` interface and provide an override `onMessage` method, where we can process the message received. Every time a new message is published to the topic the subscriber is listening to, the `onMessage` method is called.

```java
public class MyListener implements MessageListener {

    private static final Logger LOGGER = Logger.getLogger(Producer.class.getName());

    @Override
    public void onMessage(Message message) {
        try {
            TextMessage textMessage = (TextMessage) message;
            LOGGER.info("Receive message: " + textMessage.getText());
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/jms-example.git" text="Examples" %}
</p>

Image by <a href="https://pixabay.com/users/epicantus-168198/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=595854">Daria Nepriakhina</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=595854">Pixabay</a>
