---
title: Speed Up Your Microservices Development with Telepresence
image: /assets/images/shubham-beeharry-223969.jpg
author: Sergio Martin Rubio
categories:
    - DevOps
    - Testing
mermaid: false
layout: post
---

**Microservices architecture** is the way to go when you want to release new features multiple times per day, however most of the time microservices need to connect to each other. This is when _Kubernetes_ comes in to orchestrate the life and networking of _Dockerized_ microservices, and on top of that, it is reliable and performs very well. However, when developers want to debug or test a small change in a service that depends on other services located in a remote _Kubernetes_ cluster, dev cycles can become a lot slower. A developer needs to do the following to see the changes:

1. Write code locally.
2. Create a _container image_ and then push it to the _container register_.
3. Apply the _Kubernetes_ configuration with the updated _container image_.

Fortunately, there is an open-source tool called [Telepresence](https://www.telepresence.io){:target="_blank"} which helps developers in this matter. _Telepresence_ creates a fake deployment, starts a proxy inside your _Kubernetes_ cluster, and forwards all your requests from the cluster to your local process. The following example shows how to use this excellent tool.

## Getting Started

Two applications will be running, the _Golang_ one inside the _Kubernetes_ cluster and the _Spring Boot_ one on your local machine using _Telepresence_. There will be communication in both directions, so the remote service will be able to call the _Java_ application and vice versa.

### Requirements
- _Kubernetes_ cluster (_GCP_, _Minikube_, _AWS_...)
- _Kubectl_
- _Docker_
- Install _Telepresence_

To install _Telepresence_ on _Ubuntu 18_, run:

```shell
curl -s https://packagecloud.io/install/repositories/datawireio/telepresence/script.deb.sh | sudo bash
sudo apt install --no-install-recommends telepresence
```

For other _OS_, go to the official _Telepresence_ documentation.

### Remote Service

**main.go**

```go
package main
import (
  "io/ioutil"
  "log"
  "net/http"
  "github.com/gin-gonic/gin"
)
func main() {
  router := SetupRouter()
  router.Run(":8081")
}
func SetupRouter() *gin.Engine {
  r := gin.Default()
  r.GET("/golang", func(c *gin.Context) {
  c.String(200, "Hello from Golang Service")
  })
  r.GET("/java", func(c *gin.Context) {
    resp, err := http.Get("http://java-telepresence-service:8080/java")
    if err !=nil {
    log.Fatal(err)
    }
    defer resp.Body.Close()
    contents, err := ioutil.ReadAll(resp.Body)
    c.String(200, string(contents))
  })
  return r
}
```

**Dockerfile**

```Dockerfile
FROM golang:latest 
WORKDIR /go/src/golang-telepresence-service
RUN go get -d -v github.com/gin-gonic/gin
COPY main.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -o main .
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/golang-telepresence-service/main .
CMD ["./main"]
```

**deployment.yaml**

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: golang-telepresence-service
spec:
  replicas: 1
  template:
    metadata: {labels: {app: golang-telepresence-service}}
    spec:
      containers:
      - name: golang-telepresence-service
        image: smartinrub/golang-telepresence-service
        imagePullPolicy: Always
        command:
        ports:
          - containerPort: 8081
```

**service.yaml**

```yaml
kind: Service
apiVersion: v1
metadata:
  name: golang-telepresence-service
spec:
  type: LoadBalancer
  selector:
    app: golang-telepresence-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8081
```

Now you are ready to build and push the image to your repository and apply the _Kubernetes_ specifications:

```shell
docker build -t smartinrub/golang-telepresence-service .
docker push smartinrub/golang-telepresence-service
kubectl apply --filename deployment.yaml
kubectl apply --filename service.yaml
The source code can be found here.
```

### Local Service

The _Spring Boot_ application will be like this:

```java
@RestController
publicclassMainController{
  private RestTemplate restTemplate = new RestTemplate();
  @GetMapping("/java")
  public String hello(){
  return"Hello from Java Service";
  }
  @GetMapping("/golang")
  public String callGoService(){
  return restTemplate.getForObject("http://golang-telepresence-service/golang", String.class);
  }
}
```

<p class="text-center">
{% include elements/button.html link="https://github.com/smartinrub/java-telepresence-service" text="Source Code" %}
</p>


### Telepresence

```
telepresence --new-deployment java-telepresence-service --expose 8080
```

The previous command will create a deployment in your _Kubernetes_ cluster and will forward all the calls to your local port `8080`. Make sure to use the correct name for the service, since it will be used by the _Kubernetes DNS server_.

The _Go_ application will be listening on _port 80_ from outside the _pod_, so the call to the service will be:

```shell
curl -X GET http://golang-telepresence-service/java
```

The _Java_ application will be listening on _port 8080_, and will be able to be reached by hitting:

```shell
curl -X http://java-telepresence-service:8080/golang
```

As you can see, from your local machine you can call the _Java_ service which will hit the remote service in   _Kubernetes_ service and vice versa.

## Conclusion

With **Telepresence**, testing or debugging features of an application, which depends on other services, is not a problem anymore. You can make as many changes as you want on your local application and test them out against a remote service without having to do a whole deployment process on _Kubernetes_.
