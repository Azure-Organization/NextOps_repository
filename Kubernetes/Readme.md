## AKS Architecture Overview

<img width="2455" height="1735" alt="image" src="https://github.com/user-attachments/assets/5ec0ce94-daa0-441e-84f7-8a77aaf51454" />

AKS (Azure Kubernetes Service) consists of **two major parts**:

### 1. Control Plane (Managed by Microsoft)

This is the **brain of the cluster** and is fully managed by Azure. You don't need to patch, upgrade, or maintain it. [\[learn.microsoft.com\]](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks/baseline-aks), [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

**Control Plane Components:**

* **API Server (kube-apiserver)**
  * Entry point for all cluster operations.
  * `kubectl`, Azure Portal, CI/CD pipelines talk to API Server.

* **etcd**
  * Stores cluster state and configuration.
  * Contains information about pods, services, deployments, secrets, etc.

* **Scheduler**
  * Decides on which node a pod should run.

* **Controller Manager**
  * Ensures desired state equals actual state.
  * Example: If a deployment requires 3 pods and one crashes, it creates another.

* **Cloud Controller Manager**
  * Integrates Kubernetes with Azure resources such as:
    * Load Balancers
    * Public IPs
    * Managed Disks
    * Route Tables [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

***

## 2. Data Plane (Customer Managed)

This is where your application workloads run. You manage scaling, node pools, workloads, networking, and applications. [\[michaeldurkan.com\]](https://michaeldurkan.com/2026/02/08/aks-architecture-fundamentals/), [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

### Components

#### Node Pools

A Node Pool is a group of Azure VMs.

AKS normally contains:

### System Node Pool

Hosts AKS system components such as:

* CoreDNS
* Metrics Server
* CNI Components
* Ingress Controllers
* Other Kubernetes system pods

### User Node Pool

Hosts customer applications:

* Web applications
* APIs
* Microservices
* Background jobs

Best practice:

* Keep system workloads and application workloads separate. [\[medium.com\]](https://medium.com/@mbnarayn/understanding-the-key-components-of-azure-kubernetes-service-aks-c0015067bc2b), [\[michaeldurkan.com\]](https://michaeldurkan.com/2026/02/08/aks-architecture-fundamentals/)

***

## AKS High-Level Diagram

```text
                    ┌────────────────────────┐
                    │      Azure Portal      │
                    │      kubectl / CI/CD   │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │      API Server        │
                    └────────────────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            ▼                   ▼                   ▼
      ┌─────────┐         ┌─────────┐        ┌─────────┐
      │ Scheduler│         │  etcd   │        │Controller│
      └─────────┘         └─────────┘        └─────────┘
                   (Managed by Microsoft)

────────────────────────────────────────────────────

                    Data Plane (Your Subscription)

      ┌─────────────────────────────────────────┐
      │         System Node Pool                │
      │ CoreDNS, Metrics Server, CNI, etc.      │
      └─────────────────────────────────────────┘

      ┌─────────────────────────────────────────┐
      │          User Node Pool                 │
      │ API Pods                                │
      │ Frontend Pods                           │
      │ Backend Pods                            │
      └─────────────────────────────────────────┘
```

***

## Networking Flow in AKS

Suppose a customer accesses:

```text
https://app.contoso.com
```

Request flow:

```text
Internet
   │
   ▼
Azure Load Balancer
   │
   ▼
Ingress Controller
(NGINX / AGIC)
   │
   ▼
Kubernetes Service
   │
   ▼
Application Pod
```

For example:

```text
User
  ↓
Public IP
  ↓
NGINX Ingress
  ↓
Service
  ↓
Pod
```

This is the flow you often troubleshoot in production incidents.

***

## Storage Architecture

Applications often require persistent storage.

```text
Pod
 ↓
PVC (Persistent Volume Claim)
 ↓
PV (Persistent Volume)
 ↓
Azure Disk / Azure Files
```

Examples:

* Azure Disk → Database workloads
* Azure Files → Shared storage across pods

***

## Identity Architecture

AKS uses:

### Cluster Managed Identity

Used by AKS to:

* Create Load Balancers
* Attach Disks
* Manage VMSS

### Workload Identity

Used by applications to:

* Access Key Vault
* Access Storage Accounts
* Access Azure Services

Without storing secrets inside pods.

***

## Azure Resources Created for an AKS Cluster

When AKS is deployed:

```text
Resource Group
│
├── AKS Cluster
├── Node Pools (VMSS)
├── Load Balancer
├── Public IP
├── NSG
├── Route Table
├── Managed Disks
└── Managed Resource Group (MC_*)
```

The **MC\_ resource group** contains the infrastructure resources backing the cluster.

***

## Interview Question: Who Manages What?

| Component                | Managed By |
| ------------------------ | ---------- |
| API Server               | Microsoft  |
| etcd                     | Microsoft  |
| Scheduler                | Microsoft  |
| Controller Manager       | Microsoft  |
| Node Pools               | Customer   |
| Pods                     | Customer   |
| Ingress                  | Customer   |
| Applications             | Customer   |
| Networking Configuration | Customer   |

***

## Easy Way to Remember

```text
Control Plane = Brain
Node Pool     = Worker Machines
Pod           = Application
Service       = Load Balancer inside Kubernetes
Ingress       = Layer 7 Entry Point
PVC/PV        = Storage
```

For an AKS support engineer, the most commonly investigated architecture path is:

```text
Client
 → Load Balancer
 → Ingress Controller
 → Service
 → Pod
 → Node
 → Node Pool
 → Control Plane
```

Understanding this flow helps you quickly isolate whether an issue belongs to **Ingress, Networking, DNS, Node, Pod, VMSS, or the AKS control plane**.


## AKS Architecture Overview

Think of AKS as a **managed Kubernetes cluster running on Azure**.

AKS is divided into two major sections:

```text
+---------------------------------------------------+
|               AKS Cluster                         |
+---------------------------------------------------+

        Control Plane (Microsoft Managed)
                 |
                 |
    ----------------------------------
                 |
                 |
        Data Plane (Customer Managed)
```

### 1. Control Plane (Managed by Microsoft)

The Control Plane is the **brain of the cluster**.

When you create an AKS cluster, Microsoft creates and manages the control plane for you. You cannot SSH into it or directly manage it. [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture), [\[michaeldurkan.com\]](https://michaeldurkan.com/2026/02/08/aks-architecture-fundamentals/)

It contains:

```text
API Server
Scheduler
Controller Manager
etcd
Cloud Controller Manager
```

***

### API Server (kube-apiserver)

The API Server is the front door of Kubernetes.

Whenever you run:

```bash
kubectl get pods
kubectl create deployment
kubectl apply -f deploy.yaml
```

the request goes to the API Server.

```text
kubectl
    |
    v
API Server
```

The API Server validates requests and stores cluster state in etcd. [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

***

### etcd

etcd is the database of Kubernetes.

It stores:

* Pod information
* Deployments
* Secrets
* ConfigMaps
* Services
* Node information

Example:

```yaml
replicas: 5
image: nginx
```

This desired state is stored in etcd.

If etcd is lost, the cluster loses its state. Microsoft manages and protects it in AKS. [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

***

### Scheduler

The Scheduler decides:

> Which node should run the new pod?

Example:

Cluster has 3 worker nodes:

```text
Node1 -> 80% CPU
Node2 -> 20% CPU
Node3 -> 60% CPU
```

New pod created:

```text
Scheduler
      |
      +--> Node2
```

The scheduler evaluates:

* CPU
* Memory
* Taints
* Tolerations
* Affinity rules
* Resource requests

before selecting a node. [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

***

### Controller Manager

Controllers continuously compare:

```text
Desired State
      vs
Actual State
```

Example:

Deployment requires:

```yaml
replicas: 3
```

Current pods:

```text
pod1
pod2
```

One pod crashed.

Controller detects:

```text
Desired = 3
Current = 2
```

Creates a new pod automatically. [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

***

### Cloud Controller Manager

This component connects Kubernetes with Azure.

When you create:

```yaml
type: LoadBalancer
```

Cloud Controller Manager talks to Azure APIs and creates:

* Azure Load Balancer
* Public IP
* Route updates

for the service. [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture)

***

# 2. Data Plane

The Data Plane is where your applications actually run. [\[deepwiki.com\]](https://deepwiki.com/Azure/AKS/1.2-components-and-architecture), [\[michaeldurkan.com\]](https://michaeldurkan.com/2026/02/08/aks-architecture-fundamentals/)

```text
Node Pool
   |
   +---- Node
             |
             +---- Pod
                        |
                        +--- Container
```

***

## Node Pools

A Node Pool is a collection of Azure VMs.

Example:

```text
System Node Pool
==============
node-1
node-2

User Node Pool
==============
node-3
node-4
node-5
```

### System Node Pool

Hosts critical Kubernetes services.

Examples:

```text
CoreDNS
Metrics Server
CNI Pods
Konnectivity
Kube Proxy
```

Microsoft recommends keeping application workloads away from the system pool. [\[medium.com\]](https://medium.com/@mbnarayn/understanding-the-key-components-of-azure-kubernetes-service-aks-c0015067bc2b), [\[michaeldurkan.com\]](https://michaeldurkan.com/2026/02/08/aks-architecture-fundamentals/)

***

### User Node Pool

Runs application pods.

Example:

```text
Frontend
Backend
Redis
RabbitMQ
API
```

Most customer workloads run here. [\[medium.com\]](https://medium.com/@mbnarayn/understanding-the-key-components-of-azure-kubernetes-service-aks-c0015067bc2b)

***

# Pods and Containers

A Pod is the smallest deployable unit in Kubernetes.

```text
Node
 |
 +---- Pod
         |
         +--- Container (nginx)
```

or

```text
Pod
  |
  +--- App Container
  +--- Sidecar Container
```

Examples:

* nginx
* java app
* .NET app
* python app

***

# Networking Flow

This is one of the most important sections for AKS support.

Suppose a customer accesses:

```text
https://shop.contoso.com
```

Traffic flow:

```text
Internet
   |
Public IP
   |
Azure Load Balancer
   |
Ingress Controller
   |
Service
   |
Pod
```

### Step 1

Request reaches:

```text
Public IP
```

***

### Step 2

Azure Standard Load Balancer receives traffic.

```text
Azure Load Balancer
```

***

### Step 3

Traffic reaches Ingress Controller.

Example:

```text
NGINX Ingress
AGIC
Application Gateway
```

Ingress is Layer 7 (HTTP/HTTPS).

Ingress decides which backend service should get traffic.

```text
/app1 --> App1 Service
/app2 --> App2 Service
```

***

### Step 4

Kubernetes Service

Example:

```yaml
kind: Service
```

Service provides stable endpoint.

```text
Service
    |
    +--> Pod1
    +--> Pod2
    +--> Pod3
```

***

### Step 5

Application Pod processes request.

```text
User
  |
LoadBalancer
  |
Ingress
  |
Service
  |
Pod
```

This entire flow is what we usually troubleshoot in AKS networking cases.

***

# Storage Architecture

Applications often need persistent storage.

```text
Pod
 |
PVC
 |
PV
 |
Azure Disk / Azure Files
```

### Azure Disk

Mostly used by:

```text
Database
MongoDB
MySQL
SQL
ElasticSearch
```

Provides high performance.

***

### Azure Files

Used when multiple pods need the same storage.

```text
Pod1
Pod2
Pod3
   |
Azure Files
```

***

# AKS Identity Architecture

## Cluster Identity

AKS itself uses a Managed Identity.

```text
AKS
  |
Managed Identity
```

Used to:

* Create Load Balancers
* Create Public IPs
* Attach Disks
* Operate VMSS

***

## Workload Identity

Used by applications.

```text
Application Pod
        |
Workload Identity
        |
Azure Key Vault
```

No need to store secrets inside pods.

***

# VMSS Architecture

Every AKS node is actually part of a Virtual Machine Scale Set.

```text
AKS
 |
Node Pool
 |
VMSS
 |
Nodes
```

Example:

```text
aks-userpool-123456-vmss

Node1
Node2
Node3
```

When we scale AKS:

```bash
az aks nodepool scale
```

AKS adds VMs to the VMSS.

This is why many AKS incidents ultimately involve:

* VMSS Team
* Compute Team
* AKS Team

working together.

***

# What AKS Support Engineers Troubleshoot

A good AKS engineer should visualize every issue as:

```text
Client
 |
DNS
 |
Public IP
 |
Load Balancer
 |
Ingress
 |
Service
 |
Pod
 |
Node
 |
VMSS
 |
Control Plane
```

When a customer says:

> "Application is down"

You should immediately check:

```text
DNS?
Load Balancer?
Ingress?
Service?
Pod?
Node?
VMSS?
```

This troubleshooting flow helps quickly determine whether the issue belongs to AKS, Networking, VMSS, Storage, or another Azure team.

For AKS interviews and real support work, always remember:

```text
Control Plane = Cluster Brain
Node Pool     = Group of VMs
Node          = Worker Machine
Pod           = Runs Containers
Service       = Stable Endpoint
Ingress       = Layer 7 Routing
PVC/PV        = Storage
VMSS          = Underlying Compute
```

This single diagram explains nearly 80% of AKS architecture:

```text
kubectl
   |
API Server
   |
----------------------------
| Control Plane            |
| Scheduler                |
| Controller Manager       |
| etcd                     |
----------------------------
           |
           v
----------------------------
| Node Pool (VMSS)         |
|  Node1                  |
|    Pods                 |
|  Node2                  |
|    Pods                 |
----------------------------
           |
       Service
           |
       Ingress
           |
    Azure Load Balancer
           |
         Internet
```

This is the architecture model I typically use to explain AKS to new Azure support engineers.
