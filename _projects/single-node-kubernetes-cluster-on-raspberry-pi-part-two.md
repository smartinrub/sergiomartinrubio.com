---
name: Single Node Kubernetes Cluster on Raspberry Pi - Part Two
image: https://lh3.googleusercontent.com/pw/AM-JKLUJVWu6q6QIJc48pts7PM7yMfSMhZzdri7r-JatwQqccSuFt7alnW2ubdB17zr2gUNlMh0OybbFXnWitfhMU31gHwFCs4hFAehV5P_aQFdclk24ojGLorvXmfTXFpHDsHVzOKjK0ihgQaXGJ_DTzfON=w640-h427-no?authuser=0
company: Side Project
date:  2022-07-01
layout: post
---

## Single Node Kubernetes Cluster on Raspberry Pi - Part Two

In the [first part](https://sergiomartinrubio.com/projects/single-node-kubernetes-cluster-on-raspberry-pi/) we configured a Kubernetes cluster, deployed a Spring Boot application, configured a firewall and exposed the cluster to the world. However, there was something else I would like to cover. Kubernetes generates a self signed SSL/TSL certificate for HTTPS requests and this is not very nice, specially because you will see an ugly warning on your browser saying that the certificate is not trusted 😢.

On this part we will conver how to issue a trusted SSL/TSL certificate with cert-manager and Let's Encrypt! 🚀

### Issue trusted SSL/TLS certificate 

We are going to use [cert-manager](https://cert-manager.io) for issuing trusted SSL/TLS certificates for our Kubernetes cluster. cert-manager is able to generate certificates from some of the most popular certificate authorities like [Let's Encrypt](https://letsencrypt.org), [Vault](https://www.vaultproject.io) or [Venafi](https://www.venafi.com). 

cert-manager will also make sure that our certificates are valid and up to date.

### cert-manager instalation

There are [multiple ways of installing cert-manager](https://cert-manager.io/docs/installation/) in our Kubernetes cluster and for this guide we will use [Helm](https://helm.sh).

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

We are going to create to issuers, one for stating and one for production, so we can test the one for staging before issuing production certification, since there is a rate limiting in the number of certificates that Let's Encrypt issues per day.

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
	
	```
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

	```
	kubectl apply -f staging-issuer.yaml
	kubectl apply -f prod-issuer.yaml
	```

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
	
	If everything went well you can now use the production certificate by updating the ingress. Otherwise I recommend to use [this troubleshooting page from cert-manager](https://cert-manager.io/docs/faq/troubleshooting/).
	
	```yaml
	apiVersion: networking.k8s.io/v1
	kind: Ingress
	metadata:
	  name: spring-boot-demo-ingress
	  annotations:
	    nginx.ingress.kubernetes.io/rewrite-target: /
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
