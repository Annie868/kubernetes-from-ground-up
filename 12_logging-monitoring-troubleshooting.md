# Logging, Monitoring, and Troubleshooting

Effective Kubernetes management requires **continuous monitoring, logging, and systematic troubleshooting**. These practices ensure your applications and cluster nodes remain healthy, performant, and accessible.

---

## 1. Monitoring Kubernetes Resources

Monitoring allows you to **track performance, resource usage, and availability** of Pods, nodes, and services.

* **Metrics Server**: Provides CPU and memory usage metrics for Pods and nodes.

```bash
kubectl top nodes
kubectl top pods -A
```

* **Prometheus & Grafana**: Advanced monitoring and visualization for clusters.
* Monitor resource usage to **identify bottlenecks** or overloaded nodes.

---

## 2. Understanding the Troubleshooting Flow

A structured troubleshooting approach is key:

1. **Observe symptoms** – e.g., Pod crash, high latency, service unavailability.
2. **Gather logs** – check Pods, containers, and cluster components.
3. **Check resources** – node status, memory/CPU usage, and PVC/PV states.
4. **Investigate networking** – services, ingress, network policies.
5. **Apply fixes** – restart Pods, adjust configurations, or scale resources.

---

## 3. Troubleshooting Kubernetes Applications

When applications misbehave:

* **Check Pod status**:

```bash
kubectl get pods -A
kubectl describe pod <pod-name>
```

* **View container logs**:

```bash
kubectl logs <pod-name> [-c <container-name>]
```

* **Debug with an interactive shell**:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

* **Identify crash loops or resource limits** causing restarts.

---

## 4. Troubleshooting Cluster Nodes

Node issues affect the entire cluster:

* **Check node status**:

```bash
kubectl get nodes
kubectl describe node <node-name>
```

* **Identify network or kubelet issues** by inspecting system logs:

```bash
journalctl -u kubelet
```

* **Drain a node** safely for maintenance:

```bash
kubectl drain <node-name> --ignore-daemonsets
kubectl uncordon <node-name>
```

---

## 5. Fixing Application Access Problems

Application connectivity issues can arise from services, ingress, or network policies:

* **Verify Services**:

```bash
kubectl get svc -A
kubectl describe svc <service-name>
```

* **Check Ingress or Gateway configuration**:

```bash
kubectl get ingress -A
kubectl describe ingress <ingress-name>
```

* **Inspect NetworkPolicies** that may restrict Pod traffic.
* Use **port forwarding** to test direct access:

```bash
kubectl port-forward pod/<pod-name> 8080:80
```

---

## Summary

Logging, monitoring, and troubleshooting in Kubernetes involve:

* **Monitoring resource usage** with metrics and dashboards
* Following a **structured troubleshooting workflow**
* **Investigating Pods and containers** with logs and exec commands
* **Inspecting and maintaining nodes** for cluster stability
* **Diagnosing networking and access issues** with services, ingress, and port forwarding

Implementing these practices ensures your cluster is **healthy, resilient, and performant**, making it easier to resolve issues proactively.
