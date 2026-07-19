## ConfigMap

in **AKS (Azure Kubernetes Service)** is a Kubernetes object used to store **non-sensitive configuration data** as key-value pairs. It allows you to separate application configuration from the container image, making your applications easier to manage and deploy. https://learn.microsoft.com/en-us/azure/azure-app-configuration/quickstart-azure-kubernetes-service https://kubernetes.io/docs/concepts/configuration/configmap/

### Why use a ConfigMap?

Instead of hardcoding values inside your application or Docker image, you can store them in a ConfigMap and inject them into your pods.

**Common use cases:**

* Application settings
* Database hostnames
* API endpoints
* Feature flags
* Configuration files

### Example ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: "my-app"
  LOG_LEVEL: "INFO"
  DATABASE_HOST: "mysql-service"
```

Create it:

```bash
kubectl apply -f configmap.yaml
```

### Use ConfigMap as Environment Variables

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_NAME
```

### Use ConfigMap as a File

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config

containers:
- name: nginx
  image: nginx
  volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

The ConfigMap data will be mounted as files under `/etc/config`. [\[kubernetes.io\]](https://kubernetes.io/docs/tutorials/configuration/updating-configuration-via-a-configmap/), [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/configuration/configmap/)

### Useful Commands

```bash
# List ConfigMaps
kubectl get configmaps

# View a ConfigMap
kubectl describe configmap app-config

# Get YAML output
kubectl get configmap app-config -o yaml

# Delete ConfigMap
kubectl delete configmap app-config
```

### ConfigMap vs Secret

| ConfigMap                 | Secret                        |
| ------------------------- | ----------------------------- |
| Stores non-sensitive data | Stores sensitive data         |
| Plain text                | Base64-encoded                |
| App settings, URLs        | Passwords, certificates, keys |

Use **Secrets** for passwords, connection strings, certificates, and tokens. Use **ConfigMaps** for general configuration. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/configuration/configmap/)

### AKS Interview Answer

> A ConfigMap in AKS is a Kubernetes resource used to store application configuration separately from container images. The configuration can be consumed by pods as environment variables, command-line arguments, or mounted configuration files, enabling easier application management and deployment across environments. [\[kubernetes.io\]](https://kubernetes.io/docs/concepts/configuration/configmap/), [\[Quickstart...soft Learn \| Learn.Microsoft.com\]](https://learn.microsoft.com/en-us/azure/azure-app-configuration/quickstart-azure-kubernetes-service)

For AKS, I can also explain **how ConfigMaps and Secrets are used together in a real-world deployment architecture**.
