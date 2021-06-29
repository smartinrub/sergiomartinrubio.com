---
name: Build your Own Web Chat Application with XMPP
image: https://lh3.googleusercontent.com/pw/AM-JKLUy-M8aXv7j5uoW706ppG678IwT1YOYK1qVTgOMeeKQ_ngKnXGgk5x9y0exkDXSNfcln_ZeERUCui4eZZedLU0U5MajzSi1frZKLDp-wil1F7LqnbdbIi5Ik4WwEg1Qbc2jTMCcQ-nPm30wVsmbtMXu=w2798-h1562-no?authuser=1
company: Side Project
date:  2021-05-27
layout: post
---

## XMPP Smack Chat

In this project I want to show you how to build an **Instant Messaging** (IM) system with **XMPP** and **Smack**. XMPP is instant messaging protocol used by companies like **WhatsApp** or **Telegram** to orchestrate the message delivery system. XMPP, which is also referred as **Jabber** (the original name), is open source and extensible and uses XML to exchange data between client and server.

XMPP follows a client/server architecture and XMPP clients can only communicate other clients on the same domain and most of the processing and IM logic is happening on the server.

### XMPP highlights

- It's robust and powerful.
- You can use it for text, pictures, videos or audios.
- Clients available for many device types.
- It's decentralized and anyone can run their own XMPP server.

### XMPP Drawbacks

- It uses XML and this makes messages complex and verbose.
- It doesn't provide a default way to know if a message was delivered.
- It lacks standard enterprise features like transactions and quality of service support, so you cannot build mission-control applications on top of XMPP.

### XMPP core concepts

- **XMPP domain**: XMPP domains provide local control over parts of the XMPP network as well as communicating with users outside the XMPP domain. A domain consist of an internet address name.
- **Users and resources**: a XMPP user is a logical messaging endpoint which represents a user account. XMPP users are addressed by their username. The username consist of a *name + @ + domain*, like the email guidelines. XMPP supports multiple client access by using the concept of *XMPP resources*. If a single user access the XMPP server from different clients the packets are sent to distinct messaging endpoints for the same user and the XMPP server is responsible for properly routing packets sent to a user to the best resource available for that user. i.e. if a messages is sent to "foo" user, the user checks what clients for that users are connected, if any. If none are, the message is stored for later delivery. If two clients are connected, the server detects this and determines which one is the preferred resource and sends the message to that client.
- **Jabber IDs**: also referred as JID. It has the following structure: *user@domain/resource*. The resource is usually omitted. The most common usage of server addresses is to send messages to XMPP servers outside of your own XMPP domain.
- **Presence**: This gives users visibility to indicate if a user is available/unavailable. Presence provide users a more instant communication since they indicate if they are away or they are online. Presence also provides a permission mechanism to approve or disapprove presence subscription requests from other users.
- **Roster**: rosters are similar to a list of friends and they allow you to maintain a list of users and their current presence status.

## Getting started

{% include elements/video.html id="Wp8gXDY6cfk" %}

{% include elements/figure.html image="https://lh3.googleusercontent.com/pw/AM-JKLUWlfZQqIw1eoFmyT9r97djSjoypIlh94qRU9k4XAdIHAPlg_p_H3EXQ5ns5NJWFg7HC-xpe-XCFdq867y55MCtviKktqsHcQZqJ4WQTtojucg3NksWk5H7cVJmKLDTWeBMiq91VXMf9MWjMlFLaTch=w1201-h558-no?authuser=1" caption="Fitzing Diagram" %}

### XMPP Server

You can find [a list of XMPP servers at the official XMPP site](https://xmpp.org/software/servers.html). For this project we are going to use [Openfire](https://hub.docker.com/r/quantumobject/docker-openfire)

### Backend Application

We are going to build a Java application that is going to be the middleware between the front-end and the XMPP server and it will be responsible for handling the XMPP sessions, request types and responses.

The technologies that will be used on the backend web application will be:

- **Spring Boot** to speed up the process of building the web application.
- [Smack](https://www.igniterealtime.org/projects/smack/): XMPP client library to handle the interactions with the XMPP server.
- **Websocket**: this technology will be use to keep a channel of communication between the backend and front-end.
- **[MySQL](https://sergiomartinrubio.com/articles/mysql-guide/)** as a relational database to store users.
- **[Liquibase](https://sergiomartinrubio.com/articles/getting-started-with-liquibase-and-spring-boot/)** for creating the SQL schema and keeping track of the changes.
- **[BCrypt](https://sergiomartinrubio.com/articles/storing-passwords-securely-with-bcrypt-and-java/)** for hashing passwords.

We will structure our Spring Boot application in multiple layers to separate the different concerns:

- **Websocket layer**: exposes the websocker endpoint and it will contains the methods for opening a session, handling incoming messages, closing a session and handling errors. We will also create a helper class for returning responses to the client given a websocket session. We also need decoders and encoders for parsing incoming message to a pojo class.

  ```java
  @Slf4j
  @ServerEndpoint(value = "/chat/{username}/{password}", decoders = MessageDecoder.class, encoders = MessageEncoder.class)
  public class ChatWebSocket {
  
      private final XMPPFacade xmppFacade;
  
      public ChatWebSocket() {
          this.xmppFacade = (XMPPFacade) SpringContext.getApplicationContext().getBean("XMPPFacade");
      }
  
      @OnOpen
      public void open(Session session, @PathParam("username") String username, @PathParam("password") String password) {
          xmppFacade.startSession(session, username, password);
      }
  
      @OnMessage
      public void handleMessage(WebsocketMessage message, Session session) {
          xmppFacade.sendMessage(message, session);
      }
  
      @OnClose
      public void close(Session session) {
          xmppFacade.disconnect(session);
      }
  
      @OnError
      public void onError(Throwable e, Session session) {
          log.debug(e.getMessage());
          xmppFacade.disconnect(session);
      }
  }
  ```

  The `open` method expects an *username* and *password* that will be use to authenticate the user.

  Given a websocket `Session` we can send back a message like this:

  ```java
  session.getBasicRemote().sendObject(textMessage);
  ```

- **XMPP facade layer**: will contain most of the business logic of the application and will be responsible for orchestrating the creation of XMPP connections, sending messages and ending XMPP connections. Also we will store the websocket sessions associated to XMPP connections on this layer.

  - **Start session**: We will start a session by checking if the credentials are correct, and then we will create an XMPP connection for the given user. In case the user does not exist we will create on on the fly and we will use it to log in to XMPP. Then we will store the websocket session for the XMPP connection we have just created and add an incoming XMPP message listener for the connection. Finally we return with a successful message.

    ```java
    public void startSession(Session session, String username, String password) {
      Optional<Account> account = accountService.getAccount(username);
    
      if (account.isPresent() && !BCryptUtils.isMatch(password, account.get().getPassword())) {
        log.warn("Invalid password for user {}.", username);
        webSocketTextMessageHelper.send(session, WebsocketMessage.builder().messageType(FORBIDDEN).build());
        return;
      }
    
      Optional<XMPPTCPConnection> connection = xmppClient.connect(username, password);
    
      if (connection.isEmpty()) {
        webSocketTextMessageHelper.send(session, WebsocketMessage.builder().messageType(ERROR).build());
        return;
      }
    
      try {
        if (account.isEmpty()) {
          xmppClient.createAccount(connection.get(), username, password);
        }
        xmppClient.login(connection.get());
      } catch (XMPPGenericException e) {
        handleXMPPGenericException(session, connection.get(), e);
        return;
      }
    
      CONNECTIONS.put(session, connection.get());
      log.info("Session was stored.");
    
      xmppClient.addIncomingMessageListener(connection.get(), session);
    
      webSocketTextMessageHelper.send(session, WebsocketMessage.builder().to(username).messageType(JOIN_SUCCESS).build());
    }
    ```
    
  - **Send message**: given a message, a recipient and a websocket session it will talk to the XMPP client layer to send a message to the XMPP server. If something goes wrong we will disconnect the user and remove the websocket session. Apart from a message sent to another user other type of messages are supported like, adding a user to a *Roster* or getting all the users from a *Roster*.

    ```java
    public void sendMessage(WebsocketMessage message, Session session) {
      XMPPTCPConnection connection = CONNECTIONS.get(session);
    
      if (connection == null) {
        return;
      }
    
      switch (message.getMessageType()) {
        case NEW_MESSAGE -> {
          try {
            xmppClient.sendMessage(connection, message.getContent(), message.getTo());
          } catch (XMPPGenericException e) {
            handleXMPPGenericException(session, connection, e);
          }
        }
        case ADD_CONTACT -> {
          try {
            xmppClient.addContact(connection, message.getTo());
          } catch (XMPPGenericException e) {
            handleXMPPGenericException(session, connection, e);
          }
        }
        case GET_CONTACTS -> {
          Set<RosterEntry> contacts = Set.of();
          try {
            contacts = xmppClient.getContacts(connection);
          } catch (XMPPGenericException e) {
            handleXMPPGenericException(session, connection, e);
          }
    
          JSONArray jsonArray = new JSONArray();
          for (RosterEntry entry : contacts) {
            jsonArray.put(entry.getName());
          }
          WebsocketMessage responseMessage = WebsocketMessage.builder()
            .content(jsonArray.toString())
            .messageType(GET_CONTACTS)
            .build();
          log.info("Returning list of contacts {} for user {}.", jsonArray, connection.getUser());
          webSocketTextMessageHelper.send(session, responseMessage);
        }
        default -> log.warn("Message type not implemented.");
      }
    }
    ```

  - **Disconnecting user**: disconnecting a user will involve sending the user status *unavailable*, disconnecting from the XMPP server and removing the websocket session.

    ```java
    public void disconnect(Session session) {
      XMPPTCPConnection connection = CONNECTIONS.get(session);
    
      if (connection == null) {
        return;
      }
    
      try {
        xmppClient.sendStanza(connection, Presence.Type.unavailable);
      } catch (XMPPGenericException e) {
        log.error("XMPP error.", e);
        webSocketTextMessageHelper.send(session, WebsocketMessage.builder().messageType(ERROR).build());
      }
    
      xmppClient.disconnect(connection);
      CONNECTIONS.remove(session);
    }
    ```

- **XMPP client layer**: on this layer we will handle all the communication with the XMPP server, from creating the connection to sending user statuses (*stanza*).

  ```java
  @Slf4j
  @Component
  @RequiredArgsConstructor
  @EnableConfigurationProperties(XMPPProperties.class)
  public class XMPPClient {
  
      private final XMPPProperties xmppProperties;
      private final AccountService accountService;
      private final XMPPMessageTransmitter xmppMessageTransmitter;
  
      public Optional<XMPPTCPConnection> connect(String username, String plainTextPassword) {
          XMPPTCPConnection connection;
          try {
              EntityBareJid entityBareJid;
              entityBareJid = JidCreate.entityBareFrom(username + "@" + xmppProperties.getDomain());
              XMPPTCPConnectionConfiguration config = XMPPTCPConnectionConfiguration.builder()
                      .setHost(xmppProperties.getDomain())
                      .setPort(xmppProperties.getPort())
                      .setXmppDomain(xmppProperties.getDomain())
                      .setUsernameAndPassword(entityBareJid.getLocalpart(), plainTextPassword)
                      .setSecurityMode(ConnectionConfiguration.SecurityMode.disabled)
                      .setResource(entityBareJid.getResourceOrEmpty())
                      .setSendPresence(true)
                      .build();
  
              connection = new XMPPTCPConnection(config);
              connection.connect();
              log.info("User '{}' connected.", username);
          } catch (SmackException | IOException | XMPPException | InterruptedException e) {
              return Optional.empty();
          }
          return Optional.of(connection);
      }
  
      public void createAccount(XMPPTCPConnection connection, String username, String plainTextPassword) {
          AccountManager accountManager = AccountManager.getInstance(connection);
          accountManager.sensitiveOperationOverInsecureConnection(true);
          try {
              accountManager.createAccount(Localpart.from(username), plainTextPassword);
          } catch (SmackException.NoResponseException |
                  XMPPException.XMPPErrorException |
                  SmackException.NotConnectedException |
                  InterruptedException |
                  XmppStringprepException e) {
              throw new XMPPGenericException(connection.getUser().toString(), e);
          }
  
          accountService.saveAccount(new Account(username, BCryptUtils.hash(plainTextPassword)));
          log.info("Account for user '{}' created.", username);
      }
  
      public void login(XMPPTCPConnection connection) {
          try {
              connection.login();
          } catch (XMPPException | SmackException | IOException | InterruptedException e) {
              log.error("Login to XMPP server with user {} failed.", connection.getUser(), e);
  
              EntityFullJid user = connection.getUser();
              throw new XMPPGenericException(user == null ? "unknown" : user.toString(), e);
          }
          log.info("User '{}' logged in.", connection.getUser());
      }
  
      public void addIncomingMessageListener(XMPPTCPConnection connection, Session webSocketSession) {
          ChatManager chatManager = ChatManager.getInstanceFor(connection);
          chatManager.addIncomingListener((from, message, chat) -> xmppMessageTransmitter
                  .sendResponse(message, webSocketSession));
          log.info("Incoming message listener for user '{}' added.", connection.getUser());
      }
  
      public void sendMessage(XMPPTCPConnection connection, String message, String to) {
          ChatManager chatManager = ChatManager.getInstanceFor(connection);
          try {
              Chat chat = chatManager.chatWith(JidCreate.entityBareFrom(to + "@" + xmppProperties.getDomain()));
              chat.send(message);
              log.info("Message sent to user '{}' from user '{}'.", to, connection.getUser());
          } catch (XmppStringprepException | SmackException.NotConnectedException | InterruptedException e) {
              throw new XMPPGenericException(connection.getUser().toString(), e);
          }
      }
    
      public void addContact(XMPPTCPConnection connection, String to) {
          Roster roster = Roster.getInstanceFor(connection);
  
          if (!roster.isLoaded()) {
              try {
                  roster.reloadAndWait();
              } catch (SmackException.NotLoggedInException | SmackException.NotConnectedException | InterruptedException e) {
                  log.error("XMPP error. Disconnecting and removing session...", e);
                  throw new XMPPGenericException(connection.getUser().toString(), e);
              }
          }
  
          try {
              BareJid contact = JidCreate.bareFrom(to + "@" + xmppProperties.getDomain());
              roster.createItemAndRequestSubscription(contact, to, null);
              log.info("Contact '{}' added to user '{}'.", to, connection.getUser());
          } catch (XmppStringprepException | XMPPException.XMPPErrorException
                  | SmackException.NotConnectedException | SmackException.NoResponseException
                  | SmackException.NotLoggedInException | InterruptedException e) {
              log.error("XMPP error. Disconnecting and removing session...", e);
              throw new XMPPGenericException(connection.getUser().toString(), e);
          }
      }
    
      public Set<RosterEntry> getContacts(XMPPTCPConnection connection) {
          Roster roster = Roster.getInstanceFor(connection);
  
          if (!roster.isLoaded()) {
              try {
                  roster.reloadAndWait();
              } catch (SmackException.NotLoggedInException | SmackException.NotConnectedException
                      | InterruptedException e) {
                  log.error("XMPP error. Disconnecting and removing session...", e);
                  throw new XMPPGenericException(connection.getUser().toString(), e);
              }
          }
  
          return roster.getEntries();
      }
  
      public void disconnect(XMPPTCPConnection connection) {
          Presence presence = PresenceBuilder.buildPresence()
                  .ofType(Presence.Type.unavailable)
                  .build();
          try {
              connection.sendStanza(presence);
          } catch (SmackException.NotConnectedException | InterruptedException e) {
              log.error("XMPP error.", e);
  
          }
          connection.disconnect();
          log.info("Connection closed for user '{}'.", connection.getUser());
      }
  
      public void sendStanza(XMPPTCPConnection connection, Presence.Type type) {
          Presence presence = PresenceBuilder.buildPresence()
                  .ofType(type)
                  .build();
          try {
              connection.sendStanza(presence);
              log.info("Status {} sent for user '{}'.", type, connection.getUser());
          } catch (SmackException.NotConnectedException | InterruptedException e) {
              log.error("XMPP error.", e);
              throw new XMPPGenericException(connection.getUser().toString(), e);
          }
      }
  }
  ```

### Starting Up Backend Services

We are going to use docker to run all the backend services:

1. Build your Spring Boot application: `mvn clean install`

2. Run `docker-compose up` with the following file:

   ```yml
   services:
     xmpp-server:
       container_name: spring-xmpp-websocket-server
       image: smartinrub/spring-xmpp-websocket-server
       ports:
         - "8080:8080"
       depends_on:
         - spring-postgres
         - openfire
     spring-postgres:
       container_name: spring-postgres
       image: "postgres:latest"
       ports:
         - "5432:5432"
       environment:
         POSTGRES_USER: xmpp
         POSTGRES_PASSWORD: password
         POSTGRES_DB: chat
     openfire-mysql:
       container_name: openfire-mysql
       image: mysql/mysql-server:latest
       ports:
         - "3306:3306"
       environment:
         MYSQL_DATABASE: openfire
         MYSQL_USER: openfireuser
         MYSQL_PASSWORD: openfirepasswd
         MYSQL_RANDOM_ROOT_PASSWORD: "yes"
     openfire:
       container_name: openfire
       image: quantumobject/docker-openfire
       ports:
         - "9090:9090"
         - "5222:5222"
         - "5269:5269"
         - "5223:5223"
         - "7443:7443"
         - "7777:7777"
         - "7070:7070"
         - "5229:5229"
         - "5275:5275"
       depends_on:
         - openfire-mysql
   ```

3. Go to `http://localhost:9090` and setup openfire XMPP server:
   - Server settings:
     -  Set "XMPP Domain Name" to `localhost`
     - Set "Server Host Name (FQDN)" to `localhost`
     - Leave the rest as it is.
   - Database Settings:
     - Select "Standard Database Connection"
     - Select "MySQL"
     - Replace on the "Database URL" `HOSTNAME` with `openfire-mysql` and `DATABASENAME` with `openfire`, then fill in the username and password.
   -  Continue and ignore the rest of the steps.

4. Now you can use a websocket client to try out the backend application.
   - Endpoint: ws://localhost:8080/chat/sergio/pass
   - Connect will return `{"messageType":"JOIN_SUCCESS"}`
   - Send new message with body: 

```json
{
  "from": "sergio",
  "to": "jose",
  "content": "hello world",
  "messageType": "NEW_MESSAGE"
}
```

  will return `{"from":"sergio","to":"jose","content":"hello world","messageType":"NEW_MESSAGE"}`

### Front-end

For the frontend we have chosen [ReactJS](https://reactjs.org) with [Redux](https://redux.js.org). React allows you to create a frontend application by components that manage their state and to get a little help with managing the state of all the components we decided to use Redux, which basically centralize the application's state.

In order to run the fronent application we need NPM. NPM comes with Node, so on a MacOs we can simply run `brew install node`. Then to start the application we have to run `npm start`.



<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/spring-xmpp-websocket-reactjs.git" text="Download Source Code from this repository" %}
</p>
