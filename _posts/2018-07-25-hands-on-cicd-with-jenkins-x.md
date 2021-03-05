---
title: Hands-On CI/CD for Microservices with Jenkins X
image: /assets/images/shubham-beeharry-223969.jpg
categories:
    - DevOps
    - CI/CD
mermaid: false
layout: post
---

## Introduction

[Kubernetes](https://kubernetes.io){:target="_blank"} is growing in popularity and many companies use it to orchestrate their containerized microservices. Moreover, microservices are increasing the release cycle, because you do not have to build and deploy a huge monolith application every time a small change is made, therefore _DevOps_ teams should be able to deploy multiple times per day. However, the continuous integration and delivery process is becoming a bottleneck, because at the end of the development process, code changes will trigger a pipeline in a **CI tool**, and this tool is usually shared by all your microservices. So, what can we do? The answer is **Jenkins X**:

**Jenkins X** allows you to create a distributed and decoupled _CI/CD system_.

## Setting Up a Cluster

There are a few alternatives to create a _Kubernetes_ cluster with _Jenkins X_. You can use _Minikube_, _AWS_, _Azure_, _Oracle_, _OpenShift_ or _Google Cloud_, but for simplicity, **GCP** will be our choice in this example.

### Prerequisites

- Install [jx tool](https://jenkins-x.io/es/docs/getting-started/setup/install/){:target="_blank"}
- Create [Google Cloud Account](https://console.cloud.google.com/freetrial?pli=1){:target="_blank"}
- Create a **project on Google Cloud**
- Install **Git**, [gcloud](https://cloud.google.com/sdk/docs/downloads-apt-get){:target="_blank"} and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/){:target="_blank"}

### Getting Started

Login into your _Google Cloud Account_:

```shell
gcloud auth login
```

**Create cluster:**

```shell
jx create cluster gke --skip-login --default-admin-password=mypassword -n jenkinsx-demo
```

`--default-admin-password` sets the password to access **Jenkins** with the _admin user_; `-n` sets the _Kubernetes cluster name_.

Follow the prompts on the console: select your _Google Cloud project_ (`my-jenkinsx-demo`), _Google Cloud zone_ (in this case, `europe-west2-a`), machine type (`n1-standard-2`),  minimum number of nodes (3 by default), maximum number of nodes (3), install ingress controller, set domain (choose default), set _GitHub_ username, and recreate _Jenkins X_ cloud environment.

After the process is done, two new repositories will be created on your _GitHub_ account, and the _Kubernetes_ context will be switched to the Jenkins X one.

```shell
kubectl config get-contexts
CURRENT   NAME                                                CLUSTER                                             
*         gke_my-jenkinsx-demo_europe-west2-a_jenkinsx-demo   gke_my-jenkinsx-demo_europe-west2-a_jenkinsx-demo  
AUTHINFO                                            NAMESPACE
gke_my-jenkinsx-demo_europe-west2-a_jenkinsx-demo   jx

```

Now you are ready to create your microservices and deploy them on your new _Kubernetes_ cluster!

## Deploying Microservices

There are three alternatives to create your microservice and deploy it:

- Create your microservice manually and then import it using [jx import](https://jenkins-x.io/commands/jx_import/){:target="_blank"}.
- Use the [jx create](https://jenkins-x.io/commands/jx_create_spring/){:target="_blank"} _Spring_ command which provides an easy way to create a microservice with _Spring Boot_ from your terminal. Dependencies can be specified by including the `-d` argument followed by the dependency you want to add, e.g.  `jx create spring -d web -d actuator`.
- Use [jx create quickstart](https://jenkins-x.io/commands/jx_create_quickstart/) which allows you to select one of the [templates provided by Jenkins X](https://github.com/jenkins-x-quickstarts){:target="_blank"}.

**What happens when you use jx tool?**

- Creates an application in a directory.
- Initializes the repository and pushes code to _GitHub_.
- Adds _Dockerfile_ (to create container) and _Jenkinsfile_ (to create pipeline).
- Creates a _Helm chart_ to deploy on Kubernetes
- Registers _webhook_ on your git repository for your _teams jenkins_
- Triggers the pipeline for the first time

For this example, Golang will be used with the quickstart tool provided by Jenkins X. 

```shell
jx create quickstart -l Go
```

The `-l` argument allows us to filter by language.

>Note: The only available template for _Golang_ at the moment is _golang-http_.

After following all the steps and the pipeline runs, the _Go_ application will be deployed on the staging namespace (`jx-staging`).

_Kubernetes_ specifications include an ingress for the microservice, so you will be able to hit it from outside the cluster.

The _URL_ to hit the service can be found by running the following command:

```shell
jx get app
APPLICATION               STAGING PODS URL                                                               
golang-http               0.0.1   1/1  http://golang-http.jx-staging.your-google-cloud-ip.nip.io

PRODUCTION PODS URL
```

The application was only deployed on the staging environment and now it is time to promote it to production.

```shell
jx promote golang-http --version 0.0.1 --env production
```

>Note: Make sure that the version match with the one in staging, otherwise the promotion pipeline will fail.

The previous command will open a pull request on the production repository, and only when the changes are merged, the production pipeline on _Jenkins_ will be triggered. If everything goes well, the application should be deployed on your `jx-production` namespace, and the production URL will be available.

```shell
APPLICATION    STAGING PODS URL           
golang-http    0.0.1   1/1  http://golang-http.jx-staging.your-google-cloud-ip.nip.io               

PRODUCTION PODS URL
0.0.1      1/1  http://golang-http.jx-production.your-google-cloud-ip.nip.io
```

Congratulations! You have just deployed your first application on **Kubernetes** with **Jenkins X**!

## Troubleshooting

Depending on your **jx version** you might come across a **Helm error** like this:

```shell
'Error: UPGRADE FAILED: incompatible versions client[v2.10.0-rc.1] server[v2.9.1]': exit status 1
```

This happens because _Helm_ is not back compatible and _Jenkins X_ might have installed an old version. This can be solved by installing the latest _Helm_ version on your local machine. 

```shell
helm init --upgrade
```

## Conclusion

As you can see, _Jenkins X_ will take care of your microservices deployment pipeline and will save you the time of creating pipelines for each microservice.
