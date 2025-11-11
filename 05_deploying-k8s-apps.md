## Deploying Kubernetes Applications

Deploying applications in Kubernetes involves more than just running Pods. You need to consider **reliability, scaling, persistence, and observability**. This section covers the key Kubernetes objects for application deployment and how to manage them effectively.

---

## 1. Using Deployments

**Deployments** provide declarative updates for Pods and ReplicaSets. They ensure the desired number of Pods is running and handle rolling updates automatically.

### 🔹 Example Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

Apply the deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

Check status:

```bash
kubectl get deployments
kubectl get pods -l app=nginx
```

---

## 2. Running Agents with DaemonSets

**DaemonSets** ensure that a copy of a Pod runs on **every node** (or a subset of nodes). Useful for monitoring, logging, or networking agents.

### 🔹 Example DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-agent
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: node-agent
  template:
    metadata:
      labels:
        app: node-agent
    spec:
      containers:
      - name: agent
        image: busybox
        command: ["sh", "-c", "while true; do echo Hello; sleep 60; done"]
```

Apply:

```bash
kubectl apply -f node-agent.yaml
kubectl get daemonsets -n kube-system
```

---

## 3. Using StatefulSets

**StatefulSets** manage stateful applications requiring **stable network identities** and persistent storage (e.g., databases).

### 🔹 Example StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
  volumeClaimTemplates:
  - metadata:
      name: web-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

---

## 4. The Case for Running Individual Pods

Sometimes, individual Pods are sufficient for **short-lived jobs, testing, or debugging**.

Example:

```bash
kubectl run test-pod --image=busybox --command -- sleep 3600
kubectl get pods
```

Use this approach **sparingly**, as Deployments and StatefulSets provide better lifecycle management.

---

## 5. Managing Pod Initialization

**Init containers** run before app containers and prepare the environment.

### 🔹 Example with Init Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'echo Initializing...; sleep 5']
  containers:
  - name: app
    image: nginx
```

---

## 6. Scaling Applications

Scale Deployments manually:

```bash
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods -l app=nginx
```

---

## 7. Configuring Autoscaling

Horizontal Pod Autoscaler (HPA) adjusts replicas based on CPU or custom metrics.

### 🔹 Example HPA

```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10
kubectl get hpa
```

---

## 8. Using Sidecar Containers for Application Logging

**Sidecars** run alongside your main container to handle logging, monitoring, or proxying.

### 🔹 Example Sidecar

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logger
spec:
  containers:
  - name: app
    image: nginx
  - name: log-agent
    image: busybox
    command: ["sh", "-c", "tail -f /var/log/app.log"]
```

This pattern helps **centralize logs** without modifying your application code.

---

## Summary

Deploying Kubernetes applications effectively requires understanding:

* **Deployments** for stateless workloads
* **DaemonSets** for node-wide agents
* **StatefulSets** for persistent apps
* When to run **individual Pods**
* Using **Init Containers** for setup tasks
* **Scaling and autoscaling** to handle load
* **Sidecar containers** for logging and observability

These tools allow you to deploy reliable, scalable, and maintainable applications in a Kubernetes cluster.
