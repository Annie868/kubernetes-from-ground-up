
# Networking

Networking in Kubernetes is the backbone that enables Pods to communicate with each other, with services, and with external clients. Proper networking setup ensures **secure, reliable, and scalable traffic management**.

---

## 1. Managing Container Network Interface (CNI) and Network Plugins

Kubernetes networking relies on **CNI plugins** to provide Pod-to-Pod connectivity across nodes. Popular CNI plugins include:

* **Calico** – advanced networking with network policies and security
* **Flannel** – simple overlay networking
* **Weave Net** – easy setup with encrypted networking

### 🔹 Example: Check installed CNI plugin

```bash
kubectl get pods -n kube-system
```

Look for pods like `calico-node`, `flannel`, or `weave-net`.

---

## 2. Understanding Service Auto Registration and Kubernetes DNS

Kubernetes **DNS** automatically registers services, allowing Pods to resolve service names without manual configuration.

* **ClusterIP** services are automatically assigned DNS names:

  ```text
  <service-name>.<namespace>.svc.cluster.local
  ```
* Pods can access services using these names:

```bash
kubectl exec -it <pod-name> -- nslookup nginx-service
```

CoreDNS handles service discovery and DNS resolution across the cluster.

---

## 3. Using NetworkPolicies to Manage Traffic Between Pods

**NetworkPolicies** control which Pods can communicate with each other. By default, all Pods can communicate, so policies are used to **restrict traffic**.

### 🔹 Example: Allow traffic to Nginx Pod only from frontend Pods

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

Apply:

```bash
kubectl apply -f networkpolicy.yaml
kubectl get networkpolicy
```

---

## 4. Configuring NetworkPolicies to Manage Traffic Between Namespaces

You can use NetworkPolicies to **restrict cross-namespace communication**:

### 🔹 Example: Allow traffic from frontend namespace only

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-ns
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
```

This enforces **segmentation** between namespaces for security and isolation.

---

## 5. Managing CoreDNS

**CoreDNS** provides DNS resolution for services and Pods. Key management tasks include:

* Check CoreDNS pods:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

* Update ConfigMap for custom DNS entries:

```bash
kubectl edit configmap coredns -n kube-system
```

* Restart CoreDNS after changes:

```bash
kubectl rollout restart deployment coredns -n kube-system
```

Proper CoreDNS management ensures **reliable service discovery** in your cluster.

---

## Summary

Networking in Kubernetes involves:

* Installing and managing **CNI plugins** for Pod connectivity
* Using **Kubernetes DNS** for automatic service discovery
* Applying **NetworkPolicies** to control Pod-to-Pod and cross-namespace traffic
* Managing **CoreDNS** to ensure cluster-wide DNS reliability

These tools help enforce **security, connectivity, and scalability** for all your Kubernetes workloads.
