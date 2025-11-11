# Managing Application Access

Accessing applications in Kubernetes involves understanding the **networking model, services, ingress controllers, and modern Gateway APIs**. Proper configuration ensures your workloads are reachable, secure, and scalable.

---

## 1. Exploring Kubernetes Networking

Kubernetes networking provides **pod-to-pod, pod-to-service, and external connectivity**. Key concepts:

* **Pod IPs**: Each Pod gets a unique IP address.
* **Cluster networking**: Pods can communicate across nodes without NAT.
* **Services**: Abstract Pods behind stable endpoints.

Check Pod IPs:

```bash
kubectl get pods -o wide
```

---

## 2. Understanding Network Plugins

Kubernetes relies on **CNI (Container Network Interface) plugins** for pod networking. Popular options include:

* Calico
* Flannel
* Weave Net

Check CNI plugin installation:

```bash
kubectl get pods -n kube-system
```

---

## 3. Using Services to Access Applications

**Services** provide stable endpoints to Pods. Types include **ClusterIP, NodePort, LoadBalancer**.

### 🔹 Example: ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

Check Service:

```bash
kubectl get svc
kubectl describe svc nginx-service
```

---

## 4. Running an Ingress Controller

**Ingress controllers** route external traffic to services based on host/path rules. Popular controllers: NGINX, Traefik, HAProxy.

### 🔹 Install NGINX Ingress (example)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.0/deploy/static/provider/cloud/deploy.yaml
kubectl get pods -n ingress-nginx
```

---

## 5. Configuring Ingress

Ingress objects define rules for external access:

### 🔹 Example Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f ingress.yaml
kubectl get ingress
```

---

## 6. Using Port Forwarding for Direct Access

Port forwarding allows you to **access a Pod or Service locally** without exposing it externally.

```bash
kubectl port-forward pod/nginx-pod 8080:80
# Access via http://localhost:8080
```

---

## 7. Understanding Gateway API

The **Gateway API** is a modern approach to Kubernetes networking that provides:

* Improved traffic routing
* Layered abstractions (Gateways, Routes)
* Better multi-team support

It complements or replaces Ingress for complex scenarios.

---

## 8. Configuring Gateway API

Install Gateway API CRDs:

```bash
kubectl apply -k "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v0.8.0"
kubectl get crd | grep gateway
```

Define a **Gateway** to expose services:

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - protocol: HTTP
    port: 80
    name: http
```

---

## 9. Using Gateway API to Provide Access to Applications

Define **HTTPRoute** to direct traffic to services:

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: nginx-route
spec:
  parentRefs:
  - name: nginx-gateway
  rules:
  - matches:
    - path:
        type: Prefix
        value: /
    forwardTo:
    - targetRef:
        kind: Service
        name: nginx-service
        port: 80
```

Apply:

```bash
kubectl apply -f gateway.yaml
kubectl get httproute
```

---

## 10. Configuring Gateway API for TLS Access

TLS can secure Gateway traffic using certificates:

```yaml
spec:
  listeners:
  - protocol: HTTPS
    port: 443
    tls:
      certificateRefs:
      - name: nginx-cert
```

Certificates can come from **Secrets** managed manually or via **cert-manager** for automatic renewal.

---

## Summary

Managing application access in Kubernetes involves:

* Understanding **networking and CNI plugins**
* Exposing Pods via **Services**
* Routing external traffic with **Ingress controllers**
* Using **Gateway API** for advanced routing and TLS
* Leveraging **port forwarding** for testing and debugging

This ensures your applications are **reachable, secure, and scalable** across the cluster.
