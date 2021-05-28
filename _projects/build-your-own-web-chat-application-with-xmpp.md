---
name: Build your Own Web Chat Application with XMPP
image: https://lh3.googleusercontent.com/etHW-kunMUhV9XRiwEez7g_Nhw5WLS_IMIzj4hFGI5A0eHtzFIXHhBBsLRZFqHoKTkv5-N_HC7xZIbdGCpgpaQVmYUzgczOi3Z3TKHJBeA2h__8WCbd4QKViwdOuI6ISg0KlafaqpvWFsfDeAdox-adU1DVKgyQGK81sVPGbVjRvtxpBsinBm6zTZt8tJN5QcEzVrGHPV8RjbwpQGnHo2dJ5f2YgElcdIwXFMx9RIH9VDN_yRa9m7RAqxSotS6hvuGoQRYTd6cZ4iJCjNK7C1zXwFnEi6dNIWgkgLHp2Hz1w8u-u9bjf8pc5pdPz-sON_OkpSrGDhO-UP0g0Q4RmOgsiCz4q1J7c0qo_6rDSbBqKKOh2_cun1y_AVac_bQyBMRpSp-6vHUiLw2k4Z5Ht7RSvmUoz2flLnAkYttQtaTPzzjFj6FJ0Ns-jOGcN7dHwBGFexmKC3zD6m050ax9eac7eX_q5f9Dj69r8ODI5ruRMuY7e7ZUCNwydpZNGQ5GKy5EQ-L16xGf_-vsNVMwnLXdlLKJ9maa0mdOclfzkYjTBilfBmIgEVtZ84gXSVV9RC8IfzLc4QVeVWplxb2Kzq_uh4EhoqFq7Grzzn7QQLTvtGM8wbQkyzjo6N62EiuIeeBwILhf88H9yvI0DtYesJyt0EgraIBV5tThkyiNt07EluO7NvKCHVHQhNr3APhZ8YZwsa2bBsC5v_KoLoeDrLV0=w853-h711-no?authuser=1
company: Side Project
date:  2021-05-27
layout: post
---

## XMPP Smack Chat

In this project I want to show you how to build a web chat with **XMPP** and **Smack**. XMPP is instant messaging protocol used by companies like **WhatsApp** or **Telegram** to orchestrate the message delivery system. XMPP, which is also refered as **Jabber** (the original name), is open source and extensable and  uses XML to exchange data between client and server.

### XMPP highlights

- It's robust and powerful.
- You can use it for text, pictures, videos or audios.
- Clients available for many device types.
- It's decentrilized and anyone can run their own XMPP server.

### XMPP Drawbacks

- It uses XML and this makes messages complex and verbose.
- It doesn't provide a default way to know if a message was delivered.

## Getting started

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

- **Websocket layer**: exposes the websocker endpoint and it will contains the methods for opening a session, handling incomming messages, closing a session and handling errors. We will also create a helper class for returning responses to the client given a websocket session. We also need decoders and encoders for parsing incoming message to a pojo class.

  ```java
  @ServerEndpoint(value = "/chat/{username}/{password}", decoders = MessageDecoder.class, encoders = MessageEncoder.class)
  public class ChatWebSocket {
  
      private final XMPPService xmppService;
  
      public ChatWebSocket() {
          this.xmppService = (XMPPService) SpringContext.getApplicationContext().getBean("XMPPService");
      }
  
      @OnOpen
      public void open(Session session, @PathParam("username") String username, @PathParam("password") String password) {
          xmppService.startSession(session, username, password);
      }
  
      @OnMessage
      public void handleMessage(TextMessage message, Session session) {
          xmppService.sendMessage(message.getContent(), message.getTo(), session);
      }
  
      @OnClose
      public void close(Session session) {
          xmppService.disconnect(session);
      }
  
      @OnError
      public void onError(Throwable e) {
          throw new WebSocketException(e);
      }
  }
  ```

  The `open` method expects an *username* and *password* that will be use to authenticate the user.

  Given a websocket `Session` we can send back a message like this:

  ```java
  session.getBasicRemote().sendObject(textMessage);
  ```

- **XMPP facade layer**: will contain most of the business logic of the application and will be responsible for orchestrating the creation of XMPP connections, sending messages and ending XMPP connections. Also we will store the websocket sessions associated to XMPP connections on this layer.

  - **Start session**: We will start a session by checking if the credentials are correct, and then we will create an XMPP connection for the given user. In case the user does not exist we will create on on the fly and we will use it to log in to XMPP. Then we will store the websocker session for the XMPP connection we have just created and add an incoming XMPP message listener for the connection. Finally we return with a succesful message.

    ```java
        private static final Map<Session, XMPPTCPConnection> CONNECTIONS = new HashMap<>();
    
        private final AccountService accountService;
        private final WebSocketTextMessageHelper webSocketTextMessageHelper;
        private final XMPPClient xmppClient;
    
        public void startSession(Session session, String username, String password) {
            Optional<Account> account = accountService.getAccount(username);
    
            if (account.isPresent() && !BCryptUtils.isMatch(password, account.get().getPassword())) {
                log.warn("Invalid password for user {}.", username);
                webSocketTextMessageHelper.send(session, TextMessage.builder().messageType(FORBIDDEN).build());
                return;
            }
    
            Optional<XMPPTCPConnection> connection = xmppClient.connect(username, password);
    
            if (connection.isEmpty()) {
                webSocketTextMessageHelper.send(session, TextMessage.builder().messageType(ERROR).build());
                return;
            }
    
            try {
                if (account.isEmpty()) {
                    xmppClient.createAccount(connection.get(), username, password);
                }
                xmppClient.login(connection.get());
            } catch (XMPPGenericException e) {
                log.error("XMPP error. Disconnecting and removing session...", e);
                xmppClient.disconnect(connection.get());
                webSocketTextMessageHelper.send(session, TextMessage.builder().messageType(ERROR).build());
                CONNECTIONS.remove(session);
                return;
            }
    
            CONNECTIONS.put(session, connection.get());
            log.info("Session was stored.");
    
            xmppClient.addIncomingMessageListener(connection.get(), session);
    
            webSocketTextMessageHelper.send(session, TextMessage.builder().messageType(JOIN_SUCCESS).build());
        }
    ```

  - **Send message**: given a message, a recepient and a websocket session it will talk to the XMPP client layer to send a message to the XMPP server. If something goes wrong we will disconnect the user and remove the websocket session.
  -  **Disconnecting user**: disconnecting a user will involve sending the user status *unvailable*, disconnecting from the XMPP server and removing the websocker session.

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
              throw new XMPPGenericException(connection.getUser().toString(), e);
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

2. Run `docker-compose` with the following file:

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

TODO



<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/spring-xmpp-websocket-reactjs.git" text="Download Source Code from this repository" %}
</p>
