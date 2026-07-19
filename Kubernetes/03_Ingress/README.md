# Kubernetes Ingress and Load Balancers

## 1. Understanding Layer 4 and Layer 7 Load Balancers

### Layer 4 Load Balancer (Transport Layer)

A Layer 4 Load Balancer operates at the **Transport Layer (TCP/UDP)** of the OSI model.

* A Layer 4 Load Balancer is a basic load balancer that operates at the Transport Layer (TCP/UDP). It does not have application-level intelligence and simply forwards incoming requests to the backend servers based on predefined rules.
* This type of load balancer can handle any type of network traffic and can be used with various backend resources, such as Linux virtual machines, Windows virtual machines, web servers, database servers, FTP servers, and many other applications. Since it works at the transport layer, it is not application-specific and can be used for virtually any service.
* The Layer 4 Load Balancer simply receives a request and forwards it to one of the connected backend servers. It does not inspect the content of the request or make routing decisions based on URLs, hostnames, or other application-level information. Its primary purpose is to distribute traffic and improve the availability and reliability of applications.

#### Traffic Flow

```text
Client
   |
Layer 4 Load Balancer
   |
-------------------------
|           |           |
Server 1   Server 2   Server 3
```

The load balancer simply forwards the request to one of the backend servers without understanding the request content.

***

### Layer 7 Load Balancer (Application Layer)

* A Layer 7 Load Balancer is an advanced and intelligent load balancer that operates at the Application Layer (HTTP/HTTPS). It is specifically designed for web applications and websites. Unlike a Layer 4 Load Balancer, it understands the content of web requests and can make routing decisions based on various application-level attributes.
* Layer 7 Load Balancers support only HTTP and HTTPS traffic, making them suitable for web-based applications. Other services, such as mail servers, FTP servers, or database servers, typically cannot take advantage of Layer 7 routing capabilities.
* The intelligence of a Layer 7 Load Balancer comes from the routing rules that administrators configure. These rules determine how incoming requests should be processed and where they should be routed.

#### Advanced Features

* Host-based routing
* Path-based routing
* SSL termination
* Authentication
* URL rewriting
* Rate limiting
* Session affinity

***

### Host-Based Routing Example

```text
jio.com              --> Jio Backend
reliancefresh.com    --> Fresh Backend
```

Traffic is routed based on the requested hostname.

***

### Path-Based Routing Example

```text
jio.com/orders      --> Order Service
jio.com/payments    --> Payment Service
```

Traffic is routed based on the URL path.

***

### Why Layer 7 is Cost Effective
* One of the major advantages of a Layer 7 Load Balancer is its ability to host multiple web applications behind a single load balancer. For example, a company such as Reliance may have multiple websites, such as:
* Instead of deploying a separate load balancer for each website, a single Layer 7 Load Balancer can handle traffic for all these applications. This approach helps reduce infrastructure costs and simplifies management.
* This functionality is possible because Layer 7 Load Balancers support advanced routing features such as

```text
jio.com
jio.com/orders
jio.com/payments
reliancefresh.com
```

A single Layer 7 Load Balancer can handle all these applications using routing rules.

***

## Layer 4 vs Layer 7 Comparison

| Feature              | Layer 4 Load Balancer | Layer 7 Load Balancer         |
| -------------------- | --------------------- | ----------------------------- |
| OSI Layer            | Transport Layer       | Application Layer             |
| Protocols            | TCP / UDP             | HTTP / HTTPS                  |
| Understands Requests | No                    | Yes                           |
| Host-Based Routing   | No                    | Yes                           |
| Path-Based Routing   | No                    | Yes                           |
| URL Rewriting        | No                    | Yes                           |
| SSL Termination      | No                    | Yes                           |
| Use Cases            | VMs, Databases, FTP   | Websites, APIs, Microservices |

### Interview Summary

> Layer 4 Load Balancers distribute traffic based on IP addresses and ports without inspecting the request content. Layer 7 Load Balancers understand HTTP/HTTPS traffic and can make intelligent routing decisions based on hostnames, URLs, headers, and other application-level information.

***

# 2. What is Ingress in Kubernetes?

An **Ingress** is a Kubernetes API resource that defines how external HTTP/HTTPS traffic should reach services inside the cluster.

Ingress itself only contains routing rules.

### What Ingress Defines

* Host-based routing
* Path-based routing
* Backend services

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: orders-ingress
spec:
  ingressClassName: nginx
```

### Important

Ingress does not process traffic by itself.

It only stores routing rules in Kubernetes.

***

# 3. What is an Ingress Controller?

An **Ingress Controller** is the component that reads Ingress resources and implements the routing rules.

It acts as a **Layer 7 Load Balancer** inside Kubernetes.

Without an Ingress Controller:

```text
Ingress Resource
      |
      X
 Nothing happens
```

With an Ingress Controller:

```text
Internet
    |
Load Balancer
    |
Ingress Controller
    |
+----------------------+
|     AKS Cluster      |
+----------------------+
    |             |
 Service-A     Service-B
    |             |
  Pods          Pods
```

### How It Works

1. User sends a request.
2. Request reaches the Load Balancer.
3. Traffic reaches the Ingress Controller.
4. The Ingress Controller checks Ingress rules.
5. Request is sent to the appropriate service.
6. Service forwards traffic to the application pods.

***

# 4. Common Ingress Controllers

Many Ingress Controllers are available in Kubernetes:

* NGINX Ingress Controller
* Azure Application Gateway Ingress Controller (AGIC)
* Traefik
* HAProxy Ingress Controller

Reference:
[Kubernetes Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)

***

# 5. NGINX Ingress Controller in AKS

By default, AKS provides the Ingress API resource, but it does not install an Ingress Controller.

To use Ingress functionality, we typically deploy an Ingress Controller such as NGINX.

### Why NGINX is Popular

* Open source
* Community driven
* Easy to deploy
* Rich feature set
* Widely used in production

Most organizations use the **Open Source NGINX Ingress Controller**.

***

## Deploy NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/cloud/deploy.yaml
```

This deployment creates:

* Namespace
* Service Accounts
* Roles and Role Bindings
* Cluster Roles
* ConfigMaps
* Services
* Deployment
* Admission Webhooks
* IngressClass

```
pathan [ ~ ]$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/cloud/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
```
To verify:

```bash
kubectl get pods -n ingress-nginx
```

***

# 6. Why Do We Use Annotations in NGINX Ingress?

Annotations are used to **customize the behavior of the NGINX Ingress Controller for a specific Ingress resource** without changing the global NGINX configuration. The NGINX Ingress Controller reads these annotations and generates the required NGINX configuration automatically. [\[Configure...soft Learn \| Learn.Microsoft.com\]](https://learn.microsoft.com/en-us/azure/aks/app-routing-nginx-configuration), [\[bing.com\]](https://bing.com/search?q=nginx+ingress+annotations+purpose)

### Simple Rule

```text
Ingress = Where traffic should go

Annotations = How traffic should be handled
```

***

## Common Features Configured Using Annotations

* HTTP to HTTPS redirection
* URL rewriting
* Authentication
* Rate limiting
* CORS
* Sticky sessions
* Backend timeouts
* Upload size limits

***

## Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: orders-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"

spec:
  ingressClassName: nginx
```

***

## Common Annotations

| Annotation                                       | Purpose                  |
| ------------------------------------------------ | ------------------------ |
| `nginx.ingress.kubernetes.io/ssl-redirect`       | Redirect HTTP to HTTPS   |
| `nginx.ingress.kubernetes.io/rewrite-target`     | Rewrite URLs             |
| `nginx.ingress.kubernetes.io/proxy-body-size`    | Increase upload size     |
| `nginx.ingress.kubernetes.io/proxy-read-timeout` | Increase backend timeout |
| `nginx.ingress.kubernetes.io/limit-rps`          | Rate limiting            |
| `nginx.ingress.kubernetes.io/enable-cors`        | Enable CORS              |
| `nginx.ingress.kubernetes.io/affinity: cookie`   | Sticky sessions          |

***

# AKS Interview Answer (Short Version)

### What is Ingress?

> Ingress is a Kubernetes API resource that defines Layer 7 routing rules such as host-based and path-based routing for services running inside the cluster.

### What is an Ingress Controller?

> An Ingress Controller is the actual Layer 7 Load Balancer that reads Ingress resources and enforces the routing rules. Examples include NGINX Ingress Controller and AGIC.

### Why Use Annotations?

> Annotations are used to configure advanced NGINX behaviors such as SSL redirection, URL rewriting, authentication, rate limiting, CORS, sticky sessions, and timeout settings without changing the controller's global configuration.

### One-Line Memory Trick

```text
Ingress            = Routing Rules

Ingress Controller = Layer 7 Load Balancer

Annotations        = Traffic Handling Rules
```
***

### Ingress with SSL Certification

### What is cert-manager?

**cert-manager** is a Kubernetes add-on that runs as one or more pods inside the cluster. Its primary purpose is to automate the issuance, management, and renewal of TLS/SSL certificates for Kubernetes applications.

In production environments, cert-manager is commonly deployed to manage certificates automatically and eliminate the need for manual certificate renewal.

### How cert-manager Works

1. An application requires an SSL certificate.
2. cert-manager detects the certificate request.
3. cert-manager communicates with a Certificate Authority (CA), such as **Let's Encrypt**.
4. Let's Encrypt validates domain ownership and issues the certificate.
5. cert-manager stores the certificate in a Kubernetes Secret.
6. The Ingress Controller uses the certificate to serve HTTPS traffic.
7. cert-manager continuously monitors certificate expiry dates and automatically renews certificates before they expire.

### Why Do We Need cert-manager?

Without cert-manager:

* Certificates must be requested manually.
* Renewals must be performed manually.
* There is a risk of certificate expiration causing application downtime.

With cert-manager:

* Certificate issuance is automated.
* Certificate renewal is automated.
* Certificate management becomes simpler and more reliable.

### Let's Encrypt Certificate Validity

Let's Encrypt certificates are typically valid for **90 days**.

Without automation, administrators must manually renew certificates every three months. cert-manager automates this process by renewing certificates before they expire, ensuring uninterrupted HTTPS access.

### Important Requirement: Valid Domain Name

To obtain a certificate from Let's Encrypt, you must have a **valid public domain name**. During the certificate issuance process, Let's Encrypt verifies that you own or control the domain.

For example:

```text
app.contoso.com
orders.contoso.com
api.contoso.com
```

These domains must resolve to the public IP address of your Ingress Controller so that Let's Encrypt can complete the validation process.

#### Install cert-manager

After deploying the applications and the NGINX Ingress Controller, install **cert-manager** in the cluster.

```bash
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.yaml
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
namespace/cert-manager created
serviceaccount/cert-manager-cainjector created
serviceaccount/cert-manager created
serviceaccount/cert-manager-webhook created
configmap/cert-manager-webhook created
clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrole.rbac.authorization.k8s.io/cert-manager-view created
clusterrole.rbac.authorization.k8s.io/cert-manager-edit created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrole.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
role.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
role.rbac.authorization.k8s.io/cert-manager:leaderelection created
role.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
rolebinding.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
service/cert-manager created
service/cert-manager-webhook created
deployment.apps/cert-manager-cainjector created
deployment.apps/cert-manager created
deployment.apps/cert-manager-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
```
cert-manager is a third-party tool, so it extends the Kubernetes API by creating several Custom Resource Definitions (CRDs). The main resources used by cert-manager are:

```text
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io
```

These are internal cert-manager resources. When cert-manager is installed, a namespace called **cert-manager** is also created, and all cert-manager deployments and resources are deployed in that namespace.

```bash
kubectl get deploy -n cert-manager

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
cert-manager              1/1     1            1           9m15s
cert-manager-cainjector   1/1     1            1           9m15s
cert-manager-webhook      1/1     1            1           9m15s
```

```bash
kubectl get pods -n cert-manager

NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-b65f7b9fb-xnwhs              1/1     Running   0          10m
cert-manager-cainjector-849cbd4d5-j4h6t   1/1     Running   0          10m
cert-manager-webhook-799f4b488-xfwv6      1/1     Running   0          10m
```

```bash
kubectl get svc -n cert-manager

NAME                   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
cert-manager           ClusterIP   10.0.5.16    <none>        9402/TCP   11m
cert-manager-webhook   ClusterIP   10.0.52.70   <none>        443/TCP    11m
```

### Configure cert-manager

Installing cert-manager alone is not enough. After installation, we must configure the required resources for certificate management, such as:

* Issuers
* ClusterIssuers
* Certificates

Only after configuring these resources can cert-manager request, issue, and renew certificates.

### Configure ClusterIssuer

Next, we will configure the resources required to obtain SSL certificates. The first resource we need is a **ClusterIssuer**.

A ClusterIssuer is a cluster-wide resource used by cert-manager to request and manage certificates from a Certificate Authority (CA), such as Let's Encrypt.

Similar to how we create an Ingress resource for the Ingress Controller, we must create a ClusterIssuer resource for cert-manager. The ClusterIssuer is responsible for:

* Requesting certificates from the Certificate Authority
* Validating domain ownership
* Issuing certificates
* Renewing certificates automatically before they expire

cert-manager typically supports two environments:

* **Staging Environment** – Used for testing and validation purposes.
* **Production Environment** – Used for real production workloads and trusted certificates.

For testing purposes, we will first deploy the **staging ClusterIssuer** by applying the `staging-issuer.yaml` file.

The ClusterIssuer will then handle certificate requests and renewal operations on behalf of the applications running in the cluster.

**Note:** Technically, the certificate and private key are usually stored together in a Kubernetes TLS Secret. However, keeping your original explanation and only fixing the sentence formation:

When requesting a certificate, it is divided into two parts:

* **Public Certificate**
* **Private Certificate (Private Key)**

The **public certificate** is associated with the **Ingress resource** because the application is exposed through the Ingress and serves HTTPS traffic to users.

The **private certificate** is stored securely in **Kubernetes Secrets**. We can create a Secret and keep the secret name as **letsencrypt**.

cert-manager requests the certificate from Let's Encrypt and stores the generated certificate and private key in the configured Kubernetes Secret. The Ingress resource then references this Secret to enable HTTPS for the application.

For example:

```yaml
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: letsencrypt
```

In the above configuration:

* The certificate and private key are stored in the Kubernetes Secret named **letsencrypt**.
* The Ingress resource uses this Secret to serve HTTPS traffic.
* cert-manager automatically renews the certificate before it expires and updates the Secret without manual intervention.

### Architecture

```text
Internet
    |
    v
Ingress Controller (NGINX)
    |
    v
cert-manager
    |
    v
Let's Encrypt
    |
    v
TLS Certificate
    |
    v
Kubernetes Secret
    |
    v
Application
```

### Interview-Style Answer

> cert-manager is a Kubernetes add-on used to automate the issuance, management, and renewal of TLS/SSL certificates. It runs as pods inside the cluster and integrates with Certificate Authorities such as Let's Encrypt. Since Let's Encrypt certificates are valid for only 90 days, cert-manager automatically renews them before expiration and updates the corresponding Kubernetes Secrets. To obtain certificates from Let's Encrypt, a valid domain name is required for domain ownership validation.

* Actually, Let's Encrypt works with Cert-Manager to automate SSL certificate management. Cert-Manager sends validation requests through the Ingress Controller service within the cluster. As the domain name is already mapped to the Ingress Controller, the validation process completes successfully, and the certificate is issued.
* 
