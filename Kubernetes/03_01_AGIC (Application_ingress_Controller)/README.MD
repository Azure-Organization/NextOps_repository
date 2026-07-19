### Azure Application Gateway Ingress Controller (AGIC) Setup for an AKS Cluster

To enable the **Application Gateway Ingress Controller (AGIC)** for an AKS cluster, navigate to the **AKS Cluster** in the Azure portal.

1. Under **Settings**, select **Networking**.

2. In the **Virtual Network Integration** section, locate the **Application Gateway Ingress Controller** option.
   * By default, this option may be disabled if AGIC is not currently configured for the AKS cluster.

3. Click **Manage**. A new configuration window will open.

4. Enable the **Ingress Controller** checkbox.

5. You can either:
   * Select an existing Application Gateway, or
   * Click **Create New** to deploy a new Application Gateway.

When creating a new Application Gateway, Azure automatically performs the following tasks:

* Creates a dedicated subnet for the Application Gateway.
* Deploys an Application Gateway, typically named **ingress-appgateway** (default naming).
* Creates a **Public IP Address** for the frontend listener.
* Creates a **User Assigned Managed Identity** for AGIC.
* Assigns the required **Network Contributor** role permissions to the managed identity so it can modify the Application Gateway configuration.
* Creates the required **Network Security Group (NSG)** and networking components.

Once the Application Gateway integration is completed, AKS deploys the **Application Gateway Ingress Controller (AGIC)** pod in the **kube-system** namespace.

You can verify the deployment using:

```bash
kubectl get pods -n kube-system | grep ingress-appgw
```

Example output:

```bash
ingress-appgw-deployment-6bcd787444-cbhf8    1/1 Running
```

### What Happens After AGIC Is Enabled?

* AGIC continuously watches Kubernetes **Ingress** resources.
* When a new Ingress resource is created or updated, AGIC automatically updates the Application Gateway configuration.
* AGIC creates and manages:
  * Backend Pools
  * Backend HTTP Settings
  * Listeners
  * Routing Rules
  * Health Probes

This removes the need to manually configure Application Gateway components.

### Traffic Flow

```text
Internet/User
      |
      v
Public IP
      |
      v
Azure Application Gateway
      |
      v
AGIC (Ingress Controller)
      |
      v
Kubernetes Service
      |
      v
Pods
```

### Verification Commands

Check AGIC pod status:

```bash
kubectl get pods -n kube-system -l app=ingress-appgw
```

Check AGIC logs:

```bash
kubectl logs -n kube-system deployment/ingress-appgw-deployment
```

Verify Application Gateway backend pools:

```bash
az network application-gateway address-pool list \
  --resource-group <resource-group> \
  --gateway-name <app-gateway-name>
```

### Common Requirements

Before enabling AGIC, ensure:

* AKS and Application Gateway are deployed in the same Azure region.
* The Application Gateway subnet is dedicated and does not contain other resources.
* Sufficient permissions are assigned to the AGIC managed identity.
* The AKS cluster has access to the Virtual Network hosting the Application Gateway.
* Proper DNS records are configured if host-based routing is used.

### Notes

* AGIC is a **Layer 7 (HTTP/HTTPS) Ingress Controller**.
* It translates Kubernetes Ingress resources into Azure Application Gateway configuration.
* SSL/TLS termination can be performed directly on the Application Gateway.
* AGIC supports:
  * Host-based routing
  * Path-based routing
  * SSL Offloading
  * Web Application Firewall (WAF)
  * End-to-End TLS

Once AGIC is configured successfully, any Kubernetes Ingress resource created in the cluster can automatically expose applications through the Azure Application Gateway without requiring manual Application Gateway configuration changes.
