---
title: Get Started with Java Servlets
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - Java Framework
mermaid: false
layout: post
---

## What Is a Servlet?

A **Java Servlet** is simply a class that extends from one of the classes in `javax.servlet.http` or `javax.servlet` packages and is used in applications to handle network communication like _HTTP request-response_ model.

A **Container** is an interface between a component (_Servlet_, data persistance...) and the low-level functionalities that supports the component. Web containers receive network requests and redirect those requests to a servlet object by mapping the URL path contained in the request to the servlet. A URL path contains the context root and, optionally, a URL pattern:

http://host:port/context-root[/url-pattern] 

_Servlet_ containers handle many tedious tasks like opening sockets; transformations to convert HTTP into _Java API_ calls; or provide a set of interfaces to write your own servlet implementation.

>Note: Servlets and _JSPs_ require _Servlet Containers_ like [Apache Tomcat](http://tomcat.apache.org/){:target="_blank"}, [WildFly](https://wildfly.org/){:target="_blank"}, etc.. 

## How a Client Request is processed by the Web Server

1. The web server receives the HTTP request.
2. The request is forwarded to the _Servlet Container_.
3. The _Servlet Container_ creates two objects the `HttpServletRequest` and `HttpServletResponse`.
4. The container creates a thread for the request, passes the request and response objects as arguments and delegates the request to a particular _Servlet_ chosen between the _Servlets_ it contains.
5. The `service()` method is executed and evaluates which Servlet method will be called (`doGet()`, `doPost()`, `doPut()`, `doDelete()`, `doHead()`, ...) based on the request sent by the client (`GET`, `POST`, `PUT`, `DELETE`, `HEAD`, ...).
6. The _Servlet_ handles the client request and generates content to be returned to the client.
7. The container sends the _Servlet_ response to the client.

>Note: `ServletRequest` contains parameters such as name, values, attributes, and an input stream.

{% include elements/figure.html image="https://lh3.googleusercontent.com/Ig7kjtbUiYNt86tx49lpvMsZY-Cc-4teAFboao1-szXV450v4J11BEfUFX1qhAnlM5g7t2gp3flPbitnaSqHv285gttOEPkNFCZf8IIk6gkpatXSba3XtbkfnNnmUlRn96SsuGVr9g=w2400" caption="Web Server and Servlet Container" %}

## Servlet Lifecycle

The servlet container is responsible for controlling the _Servlet_ lifecycle. What happen when a request comes in?

1. If an instance of the _Servlet_ does not exist, the web container:
    - Loads the servlet class.
    - Creates an instance of the class.
    - Calls `init()` method to initialize the servlet.
2. If an instance already exists the container calls the `service()` and passes the request and response objects.

_Servlets_ stay in memory waiting for other requests, and will not be unloaded unless the servlet container sees a shortage of memory. Before the servlet is ready to be garbage collected the `destroy()` method is called. This will happen when all servlet methods are completed or after a server-specific grace period.

>Note: when `destroy()` method is called you release resources created by the `init()` method like database connections.

## Servlet Component

Use the `@WebServlet(name = "ConvertServlet")` annotation or the deployment descriptor `web.xml`.

```xml
<servlet>
    <servlet-name>ConvertServlet</servlet-name>
    <servlet-class>com.sergiomartinrubio.javaservletfilters.servlet.IpAddressConverterServlet</servlet-class>
</servlet>
```

Both the annotated servlet or the _XML_ servlet declaration must specify at least one URL pattern. In case of annotation use the `urlPatterns` attribute when other attributes are also used. For XML declaration:

```xml
<servlet-mapping>
    <servlet-name>ConvertServlet</servlet-name>
    <url-pattern>/convert</url-pattern>
</servlet-mapping>
```

```java
@WebServlet(urlPatterns = "/convert", name = "ConvertServlet")
public class IpAddressConverterServlet extends HttpServlet { 
    // servlet methods (doGet(), doPost(), doPut())
}
```

>Note: Annotations require _Servlet API 3.0_ or higher and _tomcat7_ or any later version of _Tomcat_.

>From now on most of the servlet definitions will be shown with annotations.

The servlet initialization process can be customize if you override the `init()` method of the `Servlet` interface, or if you use the `initParams` anotation attribute in combination with `@WebInitParam` annotation.

```java
@WebServlet(urlPatterns = "/convert", name = "ConvertServlet", initParams = {
        @WebInitParam(name = "param2", value = "hello"),
        @WebInitParam(name = "param2", value = "goodbye")
})
```

Initialization paramters are used to provide data that a _Servlet_ needs. The values can be retrieved with the  `getInitParameter()` method.

### Servlet Request

Clients send data to the Servlet in the `HttpServletRequest`, which contains the request URL, _HTTP_ headers, query string, and so on. Query strings contain a set of parameters and values, that can be retrieved by using the `getParameter()` method.

e.g.

```java
String myParameter = request.getParameter("myParameter");
```

### Servlet Response

Servlets return responses to clients in the `HttpServletResponse`. To send character data, use the `PrintWriter` returned by the response's `getWriter()` method. You can also send binary data with the `ServletOutputStream` returned by `getOutputStream()` method. Additionally, the response object allows you to set things like content type, status codes, cookies.

```java
PrintWriter out = response.getWriter();
response.setStatus(HttpServletResponse.SC_OK);
response.setContentType("application/json");
out.println("{\"value\":\"Hello World}\"");
```

## Filters

A **filter** is used to transform headers or content of a request or response. Filters are attached to web components, but they should be independent from web resources, so they can reused with multiple web components.

Common use cases:

- Retrieve request and add some business logic around it.
- Block the request or response. e.g. check for authentication tokens in the headers.
- Modify request headers and data. e.g. set new attributes.
- Modify response headers and data. e.g. add tracing id.
- Interact with external resources.

Use the `@WebFilter` annotation with at least one URL pattern to define a filter in a web application. Classes annotated with the `@WebFilter` annotation must implement the `javax.servlet.Filter` interface.

```java
@WebFilter(urlPatterns = "/convert")
public class FormatFilter implements Filter { 
    // filter methods (doFilter(), init(), destroy())
}
```

`doFilter()` is the main method in a `Filter` and is used to access and/or modify the request and response headers, invoke the next filter in the filter chain. If the current filter is the last filter in the chain that ends with the target web component or static resource. However, the filter can block the request and handle the response. In addition to doFilter, you must implement the init and destroy methods.

Filters can modify, add or remove data in the request and response by calling methods like `setAttribute()`. You can also  override HTTP request methods wrapping the request in an object that extends either `HttpServletRequestWrapper`. To override HTTP response methods, you wrap the response in an object that extends either `HttpServletResponseWrapper`.

```java
@WebFilter(urlPatterns = "/convert")
public class FormatFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("The filter " + FormatFilter.class.getName() + " has been created!");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        // FormatRequestWrapper extends HttpServletRequestWrapper
        request = new FormatRequestWrapper((HttpServletRequest) request);
        String format = request.getParameter("format");

        if (format != null) {
            chain.doFilter(request, response);
        } else {
            throw new InputParameterException("Missing format parameter!");
        }
    }

    @Override
    public void destroy() {
        log.info("Destroy method is invoked for the servlet " + FormatFilter.class.getName());
    }
}
```

>Note: A web container uses filter mappings to decide how to apply filters to web resources. A filter mapping matches a filter to a web component by name or to web resources by URL pattern. The filters are invoked in the order in which filter mappings appear in the filter mapping list of a WAR.

Filter constrains:

- `REQUEST`: Request comes directly from the client.
- `ASYNC`: Asynchronous request from the client.
- `FORWARD`: Request has been forwarded to a component.
- `INCLUDE`: Request is being processed by a component that has been included.
- `ERROR`: Request is being processed with the error page mechanism.

The default dispatcher type is `REQUEST` and multiple types can be selected as follows:

```java
@WebFilter(urlPatterns = "/convert", dispatcherTypes = {DispatcherType.REQUEST, DispatcherType.FORWARD})
public class FormatFilter implements Filter {
    // filter methods (doFilter(), init(), destroy())
}
```

### Event Listeners

**Event Listeners** are used to track events in your Web application. There are two types of servlet events:

1. Servlet context-level catches events about `ServletContext` lifecycle changes.
2. Session-level events are about requests coming into and going out of scope of a web application.

Both types can catch lifecycle and attribute changes.

You can implement `ServletRequestListener`, `ServletRequestAttributeListener`, `ServletContextAttributeListener`, `ServletContxtListener`, `HttpSessionAttributeListener`... and use the `@WebListener` annotation to define an event listener.

```java
@WebListener("Checks for new attributes during request")
public class NewAttributeListener implements ServletRequestAttributeListener {

    @Override
    public void attributeAdded(ServletRequestAttributeEvent servletRequestAttributeEvent) {
        log.info("The attribute \"" + servletRequestAttributeEvent.getName() + "\" with value \""
                + servletRequestAttributeEvent.getValue() + "\" was added.");
    }

    @Override
    public void attributeReplaced(ServletRequestAttributeEvent servletRequestAttributeEvent) {

    }

    @Override
    public void attributeRemoved(ServletRequestAttributeEvent servletRequestAttributeEvent) {

    }
}
```

## Exception Handling

The Servlet API provides support for custom servlet error handling. This can be configured in the `web.xml` file in your project. 

Exceptions can be mapped to _Servlets_ or _JSP_.

Mapping error to a servlet:

```xml
<error-page>
    <exception-type>com.sergiomartinrubio.javaservletfilters.exception.InputParameterException</exception-type>
    <location>/ExceptionHandler</location>
</error-page>
```

Mapping error to JPS:
```xml
<error-page>
    <exception-type>com.sergiomartinrubio.javaservletfilters.exception.InputParameterException</exception-type>
    <location>/WEB-INF/jsp/400.jsp</location>
</error-page>
```

In either case you can access the error attributes in the request, so the exception response can be customized. 

Request Attribute | Type
---------|----------
`javax.servlet.error.status_code` |	`java.lang.Integer`
`javax.servlet.error.exception_type` |	`java.lang.Class`
`javax.servlet.error.message` |	`java.lang.String`
`javax.servlet.error.exception` |	`java.lang.Throwable`
`javax.servlet.error.request_uri` |	`java.lang.String`
`javax.servlet.error.servlet_name` |	`java.lang.String`

Error codes can also be mapped as follows:

```xml
<error-page>
    <error-code>500</error-code>
    <location>/WEB-INF/jsp/400.jsp</location>
</error-page>
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-servlets.git" text="Source Code" %}
</p>
