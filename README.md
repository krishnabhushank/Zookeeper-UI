# Zookeeper-UI
UI for Zookeeper

To deploy Zookeeper UI on Kubernetes, we need to create Kubernetes resources such as deployments, services, and config maps. We'll assume you have a Zookeeper cluster already running on your Kubernetes cluster.

Here are the steps to deploy Zookeeper UI:

1. **Create a Namespace (optional):** It's a good practice to isolate your resources within a namespace.
2. **Create ConfigMap:** This will contain configuration files for the Zookeeper UI.
3. **Create Deployment:** This will define the Zookeeper UI application.
4. **Create Service:** This will expose the Zookeeper UI to be accessible.

### Step 1: Create a Namespace (Optional)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: zookeeper-ui
```

### Step 2: Create ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: zookeeper-ui-config
  namespace: zookeeper-ui
data:
  application.properties: |
    zookeeper.hosts=localhost:2181
    server.port=9090
```

### Step 3: Create Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zookeeper-ui
  namespace: zookeeper-ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zookeeper-ui
  template:
    metadata:
      labels:
        app: zookeeper-ui
    spec:
      containers:
        - name: zookeeper-ui
          image: qnib/plain-zookeeperui:latest
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: config-volume
              mountPath: /config/application.properties
              subPath: application.properties
      volumes:
        - name: config-volume
          configMap:
            name: zookeeper-ui-config
```

### Step 4: Create Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-ui
  namespace: zookeeper-ui
spec:
  selector:
    app: zookeeper-ui
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9090
  type: LoadBalancer
```

### Full YAML File

Here's a full YAML file that includes the namespace, ConfigMap, Deployment, and Service:

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  name: zookeeper-ui
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: zookeeper-ui-config
  namespace: zookeeper-ui
data:
  application.properties: |
    zookeeper.hosts=localhost:2181
    server.port=9090
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: zookeeper-ui
  namespace: zookeeper-ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zookeeper-ui
  template:
    metadata:
      labels:
        app: zookeeper-ui
    spec:
      containers:
        - name: zookeeper-ui
          image: qnib/plain-zookeeperui:latest
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: config-volume
              mountPath: /config/application.properties
              subPath: application.properties
      volumes:
        - name: config-volume
          configMap:
            name: zookeeper-ui-config
---
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-ui
  namespace: zookeeper-ui
spec:
  selector:
    app: zookeeper-ui
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9090
  type: LoadBalancer
```

### Apply the Configuration

Save the above YAML content into a file named `zookeeper-ui.yaml` and apply it using kubectl:

```sh
kubectl apply -f zookeeper-ui.yaml
```

This will create the necessary resources and expose the Zookeeper UI via a LoadBalancer service.

**Next Steps:**
**a.** Verify the deployment and service status using `kubectl get pods -n zookeeper-ui` and `kubectl get svc -n zookeeper-ui`.
**b.** Access the Zookeeper UI using the external IP provided by the LoadBalancer service.
