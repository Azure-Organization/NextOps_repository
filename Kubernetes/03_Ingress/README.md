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

# Ingress with SSL certification
### SSL Certificate Testing with NGINX Ingress and cert-manager

To test SSL certificates in Kubernetes, first deploy the required application and ingress resources:

```bash
kubectl apply -f nginx_deploy.yaml
kubectl apply -f httpd_deploy.yaml
kubectl apply -f ingress_controller.yaml
```

After deploying the applications and the NGINX Ingress Controller, install **cert-manager** in the cluster.

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

* 


