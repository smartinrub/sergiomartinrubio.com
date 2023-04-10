---
name: Single Node Kubernetes Cluster on Raspberry Pi - Part Two
image: https://lh3.googleusercontent.com/pw/AM-JKLUJVWu6q6QIJc48pts7PM7yMfSMhZzdri7r-JatwQqccSuFt7alnW2ubdB17zr2gUNlMh0OybbFXnWitfhMU31gHwFCs4hFAehV5P_aQFdclk24ojGLorvXmfTXFpHDsHVzOKjK0ihgQaXGJ_DTzfON=w640-h427-no?authuser=0
company: Side Project
date:  2022-07-01
layout: post
---

## Single Node Kubernetes Cluster on Raspberry Pi - Part Two

In the [first part](https://sergiomartinrubio.com/projects/single-node-kubernetes-cluster-on-raspberry-pi/) we configured a Kubernetes cluster, deployed a Spring Boot application, configured a firewall and exposed the cluster to the world. However, there was something else I would like to cover. Kubernetes generates a self signed SSL/TSL certificate for HTTPS requests and this is not very nice, specially because you will see an ugly warning on your browser saying that the certificate is not trusted 😢.

On this part we will cover how to issue a trusted SSL/TSL certificate with cert-manager and Let's Encrypt! 🚀

### Quick intro

We are going to use [cert-manager](https://cert-manager.io){:target="_blank"} for issuing trusted SSL/TLS certificates for our Kubernetes cluster. cert-manager is able to generate certificates from some of the most popular certificate authorities like [Let's Encrypt](https://letsencrypt.org){:target="_blank"}, [Vault](https://www.vaultproject.io){:target="_blank"} or [Venafi](https://www.venafi.com){:target="_blank"}. 

cert-manager will also make sure that our certificates are valid and up to date.

Additionally, we are going to install [Fail2Ban](https://www.fail2ban.org){:target="_blank"} and NGINX rate limiting rules to prevent attacks.

### cert-manager installation

There are [multiple ways of installing cert-manager](https://cert-manager.io/docs/installation/){:target="_blank"} in our Kubernetes cluster and for this guide we will use [Helm](https://helm.sh){:target="_blank"}.

1. First of all make sure you have `Helm` cli installed. On mac you can install it via Homebrew: `brew install helm`
2. Install Jetstack and update repository cache:

	```shell
	helm repo add jetstack https://charts.jetstack.io
	helm repo update
	```
3. A few new Kubernetes CRD (CustomResourceDefinition) resources are required:

	```shell
	kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.2/cert-manager.crds.yaml
	```
	> this is installing version `v1.8.0` but a newer version might be available.

4. Install cert-manager in the Kubernetes cluster:

	```shell
	helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.8.0
	```

### Issue an ACME certificate using HTTP validation

Now that cert-manager is running in our Kubernetes cluster we can start issues our certificates.

We are going to create to issuers, one for staging and one for production, so we can test the one for staging before issuing the production certification one because there is a rate limiting in the number of certificates that Let's Encrypt issues per day.

1. Create `ClusterIssuer` resource definition.

	staging-issuer.yaml
	
	```yaml
	apiVersion: cert-manager.io/v1
	kind: ClusterIssuer
	metadata:
	  name: letsencrypt-staging
	spec:
	  acme:
	    server: https://acme-staging-v02.api.letsencrypt.org/directory
	    email: <your_email> # it should be a valid one
	    privateKeySecretRef:
	       name: letsencrypt-staging
	    solvers:
	     - selector: {}
	       http01:
	         ingress:
	           class: public # the built-in ingress class is called public in MicroK8s, not nginx
	```
	
	production-issuer.yaml
	
	```yaml
	apiVersion: cert-manager.io/v1
	kind: ClusterIssuer
	metadata:
	name: letsencrypt-prod
	spec:
	acme:
		server: https://acme-v02.api.letsencrypt.org/directory
		email: <your_email> # it should be a valid one
		privateKeySecretRef:
		name: letsencrypt-prod
		solvers:
		- selector: {}
		http01:
			ingress:
			class: public # the built-in ingress class is called public in MicroK8s, not nginx
	```
	
> IMPORTANT: The selected ingress class is called `public` instead of `nginx`. This is specific for a MicroK8s cluster.
	
2. Apply the issuers:

	- For staging:

	```
	kubectl apply -f staging-issuer.yaml
	```

	Check the issuer is ready:

	```
	kubectl get clusterIssuer -n cert-manager
	```

	- For production:

	```
	kubectl apply -f prod-issuer.yaml
	```

	Check the issuer is ready:

	```
	kubectl get clusterIssuer -n cert-manager
	```

	It should print out something like:

	```shell
	NAME                  READY   AGE
	letsencrypt-staging   True    8m14s
	letsencrypt-prod      True    7s
	```

	If you see False on the `READY` field you can find more details on the official cert-manager site for [Troubleshooting Problems with ACME / Let's Encrypt Certificates ](https://cert-manager.io/docs/troubleshooting/acme/){:target="_blank"}.

3. Update the `ingress.yaml`:

	```yaml
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: spring-boot-demo-ingress
	  annotations:
	    nginx.ingress.kubernetes.io/rewrite-target: /
	    cert-manager.io/cluster-issuer: "letsencrypt-staging"
	spec:
	  tls:
	  - hosts:
	    - <your_dns>
	    secretName: tls-secret
	  rules:
	  - host: <your_dns> # this is required to make the certificate work
	    http:
	      paths:
	        - path: /
	          pathType: Prefix
	          backend:
	            service:
	              name: spring-boot-demo-service
	              port:
	                number: 8080
	```
	
	After a few seconds or minutes when running `kubectl get certificate` you should see:
	
	```
	NAME         READY   SECRET       AGE
	tls-secret   True    tls-secret   <some_time>
	```
	
	> For the ACME challenge validation to succeed you might need to open the port 80.
	
	If everything went well you can now use the production certificate by updating the ingress. Otherwise I recommend to use [this troubleshooting page from cert-manager](https://cert-manager.io/docs/faq/troubleshooting/){:target="_blank"}.
	
	```yaml
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: spring-boot-demo-ingress
	  annotations:
	    nginx.ingress.kubernetes.io/rewrite-target: /$1 # this is required when having a pathType=Prefix
	    cert-manager.io/cluster-issuer: "letsencrypt-prod"
	spec:
	  tls:
	  - hosts:
	    - <your_dns>
	    secretName: tls-secret
	  rules:
	  - host: <your_dns> # this is required to make the certificate work
	    http:
	      paths:
	        - path: /
	          pathType: Prefix
	          backend:
	            service:
	              name: spring-boot-demo-service
	              port:
	                number: 8080
	```
	
Now browsers shouldn't show any warnings about self signed certificates! 🙌

### Fail2Ban

You can install Fail2Ban with:

```bash
sudo apt install fail2ban -y
```

then check that it's up and running:

```
sudo systemctl status fail2ban
```

If it showing as `inactive (dead)` try to restart Fail2Ban with `sudo systemctl restart fail2ban`.

Fail2Ban comes with a default configuration but you can create your own configuration for services like SSH. The convention is to create separate configuration files for each service. For example, for SSH we would create a file named `sshd.conf` under `/etc/fail2ban/fail.d/`.

```shell
sudo nano /etc/fail2ban/jail.d/sshd.conf
```

with the following content:

```conf
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 120
ignoreip = whitelist-IP
```

This configuration will ban IPs hitting the SSH service after 3 failures and for 120 seconds.

To apply the configuration you just need to restart Fail2Ban:

```
sudo systemctl restart fail2ban
```

You can also check all the active bans on IPs with:

```
sudo fail2ban-client status
```

To ban a particular IP address for the SSH service run: `sudo fail2ban-client set sshd banip <IP_ADDRESS>`. Similarly, you can unban an IP with: `sudo fail2ban-client set sshd unbanip <IP_ADDRESS>`.

Under the hood Fail2Ban is simply adding rules to `iptables` and you can see those rules with `sudo iptables -nL`.

### Kubernetes Rate Limiting

You can configure the [Kubernetes NGINX Ingress Controller with annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#rate-limiting){:target="_blank"} to mitigate DDoS Attacks.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
	name: spring-boot-demo-ingress
	annotations:
		nginx.ingress.kubernetes.io/rewrite-target: /$1 # this is required when having a pathType=Prefix
		nginx.ingress.kubernetes.io/limit-rps: "3"
    	nginx.ingress.kubernetes.io/limit-rpm: "60"
    	nginx.ingress.kubernetes.io/limit-connections: "5"
		cert-manager.io/cluster-issuer: "letsencrypt-staging"
spec:
	tls:
	- hosts:
		- <your_dns>
		secretName: tls-secret
	rules:
	- host: <your_dns> # this is required to make the certificate work
		http:
		paths:
			- path: /
			pathType: Prefix
			backend:
				service:
				name: spring-boot-demo-service
				port:
					number: 8080
```
