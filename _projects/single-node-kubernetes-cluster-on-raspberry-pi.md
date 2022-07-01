---
name: Single Node Kubernetes Cluster on Raspberry Pi
image: https://lh3.googleusercontent.com/CjZf0nniOuxT3FxIB2vgFlbFvhKgz41ogZkTIaqcH77NUO-CC5aHdISlOVpC0cmlpyHXKXm-u6ZjDdyND2VzlSZbMf6RqTDDfYUquyZb0sZV_DhmakY1V4DIQ8jRZznuqFAdcKp8Cw=w700
company: Side Project
date:  2020-07-02
layout: post
---

## Single Node Kubernetes Cluster on Raspberry Pi

**Kubernetes** is the most popular container orchestrator and is now a mature project created originally by _Google_ and finally announced as an open source project in 2014.

We've been thinking about improving my _Kubernetes_ skills and we thought that setting up my own _Kubernetes_ cluster would be a good opportunity to learn more about this amazing technology. There are a few options out there to run a _Kubernetes_ cluster. You can install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) on _macOS_, _Linux_ or _Windows_, which is a one node _Kubernetes_ cluster running in a VM on top of a host; use [Google Cloud GKE](https://cloud.google.com/kubernetes-engine/) or [Amazon EKS](https://aws.amazon.com/eks/); or self host your _Kubernetes_ cluster. We already tried _Minikube_ in the past and we wanted to get a full fledged experience without worrying about account trials, so we decided to acquire the most powerful **Raspberry Pi** at the moment.

### Requirements

1. [Raspberry Pi (4 Model B with 4GB RAM is recommended)](https://thepihut.com/products/raspberry-pi-4-model-b?variant=20064052740158&src=raspberrypi)
2. _MicroSD_ card (128GB or greater is recommended)
3. Keyboard
4. HDMI cable (micro HDMI to HDMI if you by the Raspberry Pi 4 Model B)
5. MicroSD adapter

### Getting Started

#### Ubuntu Server

Once you have all the required components you can start setting up you Raspberry Pi with the [latest Ubuntu Server image](https://ubuntu.com/download/raspberry-pi) compatible with your Raspberry Pi version.

> 25/03/2020: We chose _Ubuntu 20.04.2_ after trying _Ubuntu 18.04.4 LTS_ and having some issues.

Now you can copy the Ubuntu image into the micro SD card (**on MacOS**):

1. Find the SD card `mountpoint` (e.g. `/dev/disk2`).
2. Unmount the SD card.
3. Copy the image into the SD card.

```shell
diskutil list
diskutil unmountDisk /dev/disk2
sudo sh -c 'gunzip -c ~/Downloads/ubuntu-19.10.1-preinstalled-server-arm64+raspi3.img.xz | sudo dd of=/dev/disk2 bs=32m'
```

​	The next step is to plug the Raspberry Pi and change the default password.

> Default username/password for _Ubuntu 20.04.2_ is ubuntu/ubuntu.

4. Configure the WIFI connection:

   ```shell
   ip link show # Find out the wifi adapter name
   cd /etc/netplan
   sudo cp 50-cloud-init.yaml 50-cloud-init.yaml.backup # backup
   sudo nano 50-cloud-init.yaml 
   ```

   now add the following configuration:

   ```yaml
   network:
       version: 2
       ethernets:
           eth0:
               optional: true
               dhcp4: true
       # add wifi setup information here ...
       wifis:
           wlan0:
               optional: true
               access-points:
                   "YOUR-SSID-NAME":
                       password: "<YOUR-NETWORK-PASSWORD>"
               # for static IP
               dhcp4: no
               addresses: [192.168.1.107/24]
               gateway4: 192.168.1.1
               nameservers:
                   addresses: [8.8.8.8,8.8.4.4]
   ```

   You can run `sudo netplan --debug try` to apply the changes. If an error is shown you can run `sudo netplan --debug generate` to get more info about the error.

   Now it should have access to the internet.

   Run `ip a` to find out the IP of your Rasbperry Pi.

   From now on you can access the Raspberry Pi through SSH.

   ```shell
   ssh ubuntu@<RASPBERRY_PI_IP>
   ```

To take the security a step further you can use SSH public key authentication:

1. Generate keys

   ```shell
   ssh-keygen
   ```

2. Copy the public key to Ubuntu Server

   ```shell
   ssh-copy-id ubuntu@<raspberrypi_ip>
   ```
3. (Optional) Create a ssh config file

   ```shell
   touch ~/.ssh/config
   ```

   and copy

   ```shell
   Host raspi
     HostName sergiomartin.dynu.com
     port 2280
     User ubuntu
   ```

#### MicroK8s

[MicroK8s](https://microk8s.io) is a lightweight Kubernetes which is great for hardware with limited resources like Raspberry Pi. They recommend you to have at least 20G of disk space and 4G of memory are recommended.

Installation & configuration:

1. Install it through a snap command:

   ```shell
   sudo snap install microk8s --classic
   ```

2. (Optional) Add your user to the microk8s group:

   ```shell
   sudo usermod -a -G microk8s $USER
   ```

   > A restart is required for the changes to be applied.

3. (Optional) Create an alias for `kubectl`. Add to `~/.bash_aliases` or `~/.zshrc` an alias.

   ```shell
   alias kubectl="microk8s.kubectl"
   ```

   > In my case we are also using ZSH shell so we had to uncomment `export PATH=$HOME/bin:/usr/local/bin:$PATH` and add `export PATH=$PATH:/snap/bin` in the `~/.zshrc` file.

4. Run `microk8s inspect` and it will show `The memory cgroup is not enabled`. You can enable this by editing the boot parameters:

   ```shell
   sudo nano /boot/firmware/cmdline.txt
   ```

   > For Ubuntu 20.04.2. This is not required for Ubuntu Server 22.04

   and prepend the following:

   ```shell
   cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1
   ```

   finally reboot the system:

   ```shell
   sudo reboot
   ```

   Now if you show the content of `/proc/cgroups` it should show the memory row with 1 instead of 0.

5. (Optional) Change default host name:

   ```shell
   sudo hostnamectl --static set-hostname pi401
   ```

6. Now you can check if everything is up and running.

   ```shell
   microk8s.status --wait-ready
   ```

   this should show `microk8s is running` and a list of add-ons.

   if you run `kubectl get nodes` you should also be able to see something like this:

   ```shell
   NAME    STATUS   ROLES    AGE     VERSION
   pi401   Ready    <none>   7d23h   v1.17.3
   ```

7. Install and configure `kubectl` on your local machine (this is for *MacOS*):

   ```shell
   brew install kubectl
   ```

   create `.kube/config` if not present and add the following:

   ```yaml
   apiVersion: v1
   clusters:
   - cluster:
       insecure-skip-tls-verify: true
       server: https://192.168.1.107:16443
     name: microk8s-cluster
   contexts:
   - context:
       cluster: microk8s-cluster
       user: admin
     name: microk8s
   current-context: microk8s
   kind: Config
   preferences: {}
   users:
   - name: admin
     user:
       token: <token>
   ```
   
   You can find the Raspberry Pi IP and cluster port with:
   
   ```shell
   kubectl cluster-info
   ```
   
   Get the token:
   
   ```shell
   kubectl config view --raw
   ```

   now the current context should be `microk8s`

   ```shell
   kubectl config current-context
   ```
   
   You can run `kubectl get nodes` from your remote machine to check that everything works.

#### Add-ons

MicroK8s does not come with extra features out-of-the-box, however it provides a [list of add-ons](https://microk8s.io/docs/addons) that can be installed.

Let's start installing the following add-ons:

```shell
microk8s.enable dns dashboard storage ingress registry
```

check add-ons deployments:

```shell
kubectl get deployments --all-namespaces
```

- `dns` will allow services to communicate with each other.
- `dashboard` enables the Kubernetes Web UI to manage the resources. To start using the Kubernetes dashboard:

  - Use an HTTP Proxy to access the Kubernetes API
    ```
    kubectl proxy
    ```
  - Open your browser and go to `http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`
  - Get the token:
    ```
    token=$(microk8s.kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
    kubectl -n kube-system describe secret $token
    ```

- `ingress`: Configure an ingress controller to expose your services to outside the cluster.
- `storage`: Create a default storage class which allocates storage from a host directory.
- `registry`: Deploy a private image registry and expose it on `localhost:32000`.
- Other available add-ons are [cilium](http://docs.cilium.io/en/stable/intro/), [fluentd](https://microk8s.io/docs/addon-fluentd), [gpu](https://microk8s.io/docs/addon-gpu), [helm](https://helm.sh), [istio](https://istio.io/docs/), [jaeger](https://github.com/jaegertracing/jaeger-operator), [juju](https://juju.is/docs/what-is-juju), [knative](https://knative.dev), [kubeflow](https://www.kubeflow.org)

#### Deploying an App

Deploying an app on a **Kubernetes** cluster running on a **Raspberry Pi** is the same as doing it on any other machine, except for one point. The CPU architecture is [ARM](https://www.arm.com/) rather than _x86/x64_ by _Intel_ or _AMD_. Thus, Docker based images you use have to be packaged specifically for ARM architecture, otherwise you will get an error like this:

```
standard_init_linux.go:207: exec user process caused "exec format error"
```

If the image contains _RPI_ or _ARM_ in the name or description, it can usually be used for the Raspberry Pi. As a first app we are going to deploy a Java service with the latest and greatest [Spring Boot](https://start.spring.io/) version.

1. The app will expose an HTTP endpoint to convert a string to uppercase.

   ```java
   @Slf4j
   @RestController
   @SpringBootApplication
   public class Application {

      public static void main(String[] args) {
         SpringApplication.run(Application.class, args);
      }

      @GetMapping("/uppercase/{input}")
      public String uppercase(@PathVariable("input") String input) {
         log.info("Converting string to uppercase...");
         return input.toUpperCase();
      }
   }
   ```

   <p class="text-center">
   {% include elements/button.html link="https://github.com/smartinrub/raspberrypi-microk8s-java" text="Spring Boot App Repository" %}
   </p>

2. Build the app

   ```shell
   mvn clean install
   ```

3. Build Docker image

   ```shell
   docker build -t smartinrub/raspberrypimicrok8sjava .
   ```

   > As we previously mentioned only ARM images can run on Raspberry Pi. [Java Docker image for Raspberry Pi](https://hub.docker.com/r/arm64v8/openjdk/).

   ```Dockerfile
   FROM arm64v8/openjdk:11.0.6-jdk-buster AS builder
   WORKDIR target/dependency
   ARG APPJAR=target/*.jar
   COPY ${APPJAR} app.jar
   RUN jar -xf ./app.jar

   FROM arm64v8/openjdk:11.0.6-jre-slim-buster
   VOLUME /tmp
   ARG DEPENDENCY=target/dependency
   COPY --from=builder ${DEPENDENCY}/BOOT-INF/lib /app/lib
   COPY --from=builder ${DEPENDENCY}/META-INF /app/META-INF
   COPY --from=builder ${DEPENDENCY}/BOOT-INF/classes /app
   ENTRYPOINT ["java","-cp","app:app/lib/*","com.sergiomartinrubio.raspberrypimicrok8sjava.Application"]
   ```

4. Push image to Docker Hub

   ```shell
   docker push
   ```

   > You need to run `docker login` first.

5. Create replica sets of the application on Kubernetes

   ```shell
   kubectl apply -f deployment.yaml
   ```

   _deployment.yaml_

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: spring-boot-demo-deployment
     labels:
       app: spring-boot-demo
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: spring-boot-demo
     template:
       metadata:
         labels:
           app: spring-boot-demo
       spec:
         containers:
         - name: raspberrypimicrok8sjava
           image: smartinrub/raspberrypimicrok8sjava
           ports:
           - containerPort: 8080
   ```

6. Create service resource

   ```shell
   kubectl apply -f service.yaml
   ```

   _service.yaml_

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: spring-boot-demo-service
   spec:
     selector:
       app: spring-boot-demo
     ports:
       - port: 8080
         targetPort: 8080
   ```

7. Create ingress resource

   ```shell
   kubectl apply -f ingress.yaml
   ```

   _ingress.yaml_

   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
   name: spring-boot-demo-ingress
   annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
   ingressClassName: spring-boot-demo-ingress
   rules:
      - http:
         paths:
            - path: /
               pathType: Prefix
               backend:
               service:
                  name: spring-boot-demo-service
                  port:
                     number: 8080
   ```

Now you should be able to hit the service at `https://<raspberry_pi_ip>/uppercase/hello` from your home local network.

#### Firewall Configuration

It's important to keep your Raspberry Pi secured so we are going to enable the firewall and set a few rules to allow incoming traffic to our app.

Allow pod traffic:

```shell
sudo ufw allow in on cni0 && sudo ufw allow out on cni0
```

Update the `cbr0` bridge interface `ufw` rules:

```shell
sudo ufw allow in on cbr0 && sudo ufw allow out on cbr0
```

Allow routing:

```shell
sudo ufw default allow routed
```

Allow ssh, http and https, Kubernetes management port:

```shell
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 16443
```

Enable firewall

```shell
sudo ufw enable
```

#### Expose the Cluster to the Public Network

1. Buy or use a free _DNS_ address (e.g. [dynu.com](https://www.dynu.com/))
2. Install and configure **DDClient** for your dns provider:

   ```shell
   sudo apt install ddclient
   ```

   >When installing Ubuntu, the locales may not be completely set and we might get something like `perl: warning: Setting locale failed`. This can be fixed by generating the missing locale e.g. `sudo locale-gen en_GB.UTF-8`.

   DDClient configuration for Dynu:

   ```conf
   # ddclient configuration for Dynu
   #
   # /etc/ddclient.conf
   daemon=60                                                # Check every 60 seconds.
   syslog=yes                                               # Log update msgs to syslog.
   mail=root                                                # Mail all msgs to root.
   mail-failure=root                                        # Mail failed update msgs to root.
   pid=/var/run/ddclient.pid                                # Record PID in file.
   use=web, web=checkip.dynu.com/, web-skip='IP Address'    # Get ip from server.
   server=api.dynu.com                                      # IP update server.
   protocol=dyndns2
   login=<myusername>                                       # Your username.
   password=<YOURPASSWORD>                                  # Password or MD5/SHA256 of password.
   <MYDOMAIN.DYNU.COM>                                      # List one or more hostnames one on each line.                              
   ```

3. Run a daemon to keep our dynamic ip address updated on Dynu:

   ```shell
   sudo /usr/sbin/ddclient -daemon 300 -syslog
   ```

4. Now you can hit your app with the chosen domain name `https://<domain_name>/uppercase/hello`. However Kubernetes generated a self signed SSL/TLS certificate and browsers will warn you about this. Therefore, next is to generate a trusted SSL/TSL certificate and attach it to the ingress.

> Remember that you have to upate your NAT Forwarding configuration on your router to point to your raspberry pi on a specific port.

Check out [Part Two](https://sergiomartinrubio.com/projects/single-node-kubernetes-cluster-on-raspberry-pi-part-two/) for issuing a trusted SSL/TLS certificate!