# Ingress:
**Layer 4 Load Balancer**
* A Layer 4 Load Balancer is a basic load balancer that operates at the Transport Layer (TCP/UDP). It does not have application-level intelligence and simply forwards incoming requests to the backend servers based on predefined rules.
* This type of load balancer can handle any type of network traffic and can be used with various backend resources, such as Linux virtual machines, Windows virtual machines, web servers, database servers, FTP servers, and many other applications. Since it works at the transport layer, it is not application-specific and can be used for virtually any service.
* The Layer 4 Load Balancer simply receives a request and forwards it to one of the connected backend servers. It does not inspect the content of the request or make routing decisions based on URLs, hostnames, or other application-level information. Its primary purpose is to distribute traffic and improve the availability and reliability of applications.

**Layer 7 Load Balancer**
* A Layer 7 Load Balancer is an advanced and intelligent load balancer that operates at the Application Layer (HTTP/HTTPS). It is specifically designed for web applications and websites. Unlike a Layer 4 Load Balancer, it understands the content of web requests and can make routing decisions based on various application-level attributes.
* Layer 7 Load Balancers support only HTTP and HTTPS traffic, making them suitable for web-based applications. Other services, such as mail servers, FTP servers, or database servers, typically cannot take advantage of Layer 7 routing capabilities.
* The intelligence of a Layer 7 Load Balancer comes from the routing rules that administrators configure. These rules determine how incoming requests should be processed and where they should be routed.
* One of the major advantages of a Layer 7 Load Balancer is its ability to host multiple web applications behind a single load balancer. For example, a company such as Reliance may have multiple websites, such as:

```text
https://jio.com/orders    --> Order Service
https://jio.com/payments --> Payment Service
https://jio.com          --> Backend
https://reliancefresh.com --> Backend
```

* Instead of deploying a separate load balancer for each website, a single Layer 7 Load Balancer can handle traffic for all these applications. This approach helps reduce infrastructure costs and simplifies management.
* This functionality is possible because Layer 7 Load Balancers support advanced routing features such as:

Host-based Routing: Routes traffic based on the hostname requested by the client.
Path-based Routing: Routes traffic based on the URL path requested by the client.

**For example:**
* Requests for jio.com can be routed to one backend application.
* Requests for reliancefresh.com can be routed to another backend application.
* Requests for jio.com/orders can be routed to a specific service using path-based routing.

These advanced routing capabilities are not available in a traditional Layer 4 Load Balancer.

Summary
| Feature                                        | Layer 4 Load Balancer                                 | Layer 7 Load Balancer          |
| ---------------------------------------------- | ----------------------------------------------------- | ------------------------------ |
| OSI Layer                                      | Transport Layer (TCP/UDP)                             | Application Layer (HTTP/HTTPS) |
| Intelligence                                   | No                                                    | Yes                            |
| Traffic Type                                   | Any TCP/UDP traffic                                   | HTTP/HTTPS traffic only        |
| Backend Support                                | Any server/application                                | Web applications and websites  |
| Host-based Routing                             | No                                                    | Yes                            |
| Path-based Routing                             | No                                                    | Yes                            |
| Multiple Applications Behind One Load Balancer | Limited                                               | Supported                      |
| Common Use Cases                               | VMs, databases, FTP, web servers, custom applications | Websites, APIs, microservices  |


In short: A Layer 4 Load Balancer distributes traffic without understanding the application data, while a Layer 7 Load Balancer can inspect requests and intelligently route traffic based on hostnames, URLs, and other application-level information. This makes Layer 7 Load Balancers ideal for hosting and managing multiple web applications behind a single endpoint.
 
Layer 7 load balancers are commonly used as Ingress controllers in Kubernetes. In AKS, an Ingress Controller such as NGINX Ingress Controller or Azure Application Gateway Ingress Controller (AGIC) provides Layer 7 load balancing capabilities. It can inspect HTTP/HTTPS requests and perform intelligent routing based on hostnames and URL paths.

By default in AKS will have the ingress api-resource, for this resource utilization and using this resource we need to deploy the Nginx ingress, in market we have no of ingress controller 

https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/#ingress-controllers

Most of them use the NGINX Ingress Controller. It is available in two variants: an open-source version and a licensed (commercial) version. In most cases, most of them choose the open-source NGINX Ingress Controller.

**Deploy Nginx Ingress Controller**
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
**Why are annotations used in NGINX Ingress?**

Annotations are used to **customize the behavior of the NGINX Ingress Controller for a specific Ingress resource** without changing the global NGINX configuration. The NGINX Ingress Controller reads these annotations and generates the required NGINX configuration automatically. [\[Configure...soft Learn \| Learn.Microsoft.com\]](https://learn.microsoft.com/en-us/azure/aks/app-routing-nginx-configuration), [\[bing.com\]](https://bing.com/search?q=nginx+ingress+annotations+purpose)

### Simple Explanation

The Ingress resource itself only defines:

* Host-based routing
* Path-based routing
* Backend services

If you need advanced features such as:

* SSL redirection
* URL rewriting
* Rate limiting
* Authentication
* CORS
* Custom timeouts
* Session affinity (sticky sessions)

you configure them through **annotations**. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa), [\[bing.com\]](https://bing.com/search?q=nginx+ingress+annotations+purpose)

### Example

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

### Common Annotations

| Annotation                                         | Purpose                           |
| -------------------------------------------------- | --------------------------------- |
| `nginx.ingress.kubernetes.io/ssl-redirect: "true"` | Redirect HTTP to HTTPS            |
| `nginx.ingress.kubernetes.io/rewrite-target`       | Rewrite URL paths                 |
| `nginx.ingress.kubernetes.io/proxy-body-size`      | Increase upload file size limit   |
| `nginx.ingress.kubernetes.io/proxy-read-timeout`   | Increase backend response timeout |
| `nginx.ingress.kubernetes.io/limit-rps`            | Rate limiting                     |
| `nginx.ingress.kubernetes.io/enable-cors`          | Enable CORS                       |
| `nginx.ingress.kubernetes.io/affinity: cookie`     | Sticky sessions                   |

### AKS Interview-Style Answer

> NGINX Ingress annotations are used to configure advanced Layer 7 load-balancing features on a per-application basis. They allow us to customize NGINX behavior such as SSL redirection, URL rewriting, authentication, rate limiting, CORS, session affinity, and timeout settings without modifying the NGINX controller's global configuration. The NGINX Ingress Controller reads these annotations and dynamically generates the appropriate NGINX configuration for the application.

A simple way to remember is:

**Ingress = Defines routing rules**  
**Annotations = Define how NGINX should handle the traffic**. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa), [\[bing.com\]](https://bing.com/search?q=nginx+ingress+annotations+purpose)

### What is an Ingress Controller?

An **Ingress Controller** is a Kubernetes component that watches Ingress resources and implements the routing rules defined in them. It acts as a **Layer 7 (HTTP/HTTPS) Load Balancer** for applications running inside the cluster. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa), [\[Configure...soft Learn \| Learn.Microsoft.com\]](https://learn.microsoft.com/en-us/azure/aks/app-routing-nginx-configuration)

#### Without an Ingress Controller

When you create an Ingress resource, Kubernetes only stores the routing rules. By itself, the Ingress resource does **nothing**. An Ingress Controller is required to read those rules and configure a load balancer accordingly. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa)

#### How it works

1. User sends a request to the Load Balancer.
2. The request reaches the Ingress Controller.
3. The Ingress Controller checks the Ingress rules.
4. It routes the request to the correct Kubernetes Service and Pods. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa), [\[Configure...soft Learn \| Learn.Microsoft.com\]](https://learn.microsoft.com/en-us/azure/aks/app-routing-nginx-configuration)

```text
Internet
    |
Load Balancer
    |
Ingress Controller
    |
+-------------------+
|   AKS Cluster     |
+-------------------+
    |         |
 Service-A  Service-B
    |         |
   Pods      Pods
```

#### Common Ingress Controllers

* **NGINX Ingress Controller**
* **Azure Application Gateway Ingress Controller (AGIC)**
* Traefik
* HAProxy Ingress Controller

#### Example

Suppose you have two applications:

```text
jio.com            --> Jio Service
reliancefresh.com  --> Fresh Service
```

The Ingress Controller can inspect the hostname and route traffic to the correct backend service. This is called **host-based routing**. It can also route based on URL paths, such as:

```text
jio.com/orders    --> Order Service
jio.com/payments  --> Payment Service
```

This is called **path-based routing**. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa), [\[Configure...soft Learn \| Learn.Microsoft.com\]](https://learn.microsoft.com/en-us/azure/aks/app-routing-nginx-configuration)

### Simple Interview Answer

> An Ingress Controller is a Kubernetes component that implements Ingress resources and provides Layer 7 load-balancing capabilities. It receives HTTP/HTTPS traffic, applies host-based or path-based routing rules, and forwards requests to the appropriate services inside the cluster. NGINX Ingress Controller and AGIC are common examples. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa), [\[Configure...soft Learn \| Learn.Microsoft.com\]](https://learn.microsoft.com/en-us/azure/aks/app-routing-nginx-configuration)

**In short:**  
**Ingress = Routing rules**  
**Ingress Controller = The actual Layer 7 load balancer that enforces those rules**. [\[# Ingress Nginx \| External\]](https://eng.ms/cid/d1e88b28-7f74-4736-8f95-6fcb287492ea/fid/8f436bbb960a1c72d99bf7cec446751c4fe2813ca1f7463a4bd98038785c6dfa)
