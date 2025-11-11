# Managing Scheduling

Scheduling in Kubernetes determines **which Pods run on which nodes**, ensuring optimal use of cluster resources, high availability, and adherence to operational policies. Effective scheduling involves **affinity rules, resource management, and prioritization**.

---

## 1. Exploring the Scheduling Process

The **Kubernetes Scheduler** watches for unscheduled Pods and assigns them to nodes based on:

* Resource availability (CPU, memory)
* Node conditions (ready, disk pressure, memory pressure)
* Scheduling constraints (affinity, taints, tolerations, priorities)

Check a Pod’s scheduling status:

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

---

## 2. Setting Node Preferences

Node preferences help the scheduler make smarter decisions. You can **target specific nodes** using **labels** and **nodeSelector**.

### 🔹 Example Pod with nodeSelector

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disktype: ssd
```

Label nodes:

```bash
kubectl label nodes node1 disktype=ssd
kubectl get nodes --show-labels
```

---

## 3. Managing Affinity and Anti-Affinity Rules

**Affinity** rules attract Pods to certain nodes or Pods, while **anti-affinity** prevents Pods from co-locating.

### 🔹 Example Pod with affinity

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - nginx
        topologyKey: "kubernetes.io/hostname"
```

This ensures Pods run on **nodes with SSDs** but avoid nodes already hosting an Nginx Pod.

---

## 4. Managing Taints and Tolerations

**Taints** prevent Pods from scheduling on certain nodes unless the Pod **tolerates** the taint.

### 🔹 Example: Taint a node

```bash
kubectl taint nodes node1 key=value:NoSchedule
```

### 🔹 Example: Pod toleration

```yaml
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
```

Taints and tolerations help **isolate workloads**, e.g., keeping critical Pods on dedicated nodes.

---

## 5. Configuring Resource Limits and Requests

Requests and limits control **CPU and memory allocation** per Pod, influencing scheduling decisions.

### 🔹 Example

```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

* **Requests**: Minimum guaranteed resources (scheduler uses this to place Pods)
* **Limits**: Maximum resources a container can use

---

## 6. Setting Namespace Quotas

**ResourceQuotas** enforce limits on total resource usage within a namespace.

### 🔹 Example

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

Apply:

```bash
kubectl apply -f resourcequota.yaml
kubectl describe quota -n development
```

---

## 7. Configuring LimitRange

**LimitRange** defines **default requests and limits** for Pods and containers in a namespace.

### 🔹 Example

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit
  namespace: development
spec:
  limits:
  - default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 250m
      memory: 128Mi
    type: Container
```

---

## 8. Configuring Pod Priorities

**PodPriority** and **Preemption** determine which Pods get scheduled first when resources are tight.

### 🔹 Example PriorityClass

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "High priority Pods"
```

### 🔹 Apply to Pod

```yaml
spec:
  priorityClassName: high-priority
```

High-priority Pods can **preempt lower-priority Pods** if resources are insufficient.

---

## Summary

Managing scheduling in Kubernetes involves balancing **performance, availability, and resource efficiency**:

* Use **nodeSelector, affinity, and anti-affinity** to influence placement
* Apply **taints and tolerations** to isolate workloads
* Define **requests, limits, and quotas** to control resource usage
* Use **LimitRange** and **Pod priorities** for consistent and critical workloads

This ensures Pods run optimally across nodes while respecting cluster policies and resource constraints.
