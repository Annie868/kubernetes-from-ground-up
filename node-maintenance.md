# Performing Node Maintenance

Maintaining Kubernetes nodes ensures cluster stability, data integrity, and high availability. Proper maintenance involves monitoring performance, backing up and restoring critical components like **etcd**, upgrading nodes safely, and configuring a **highly available (HA)** control plane.

---

## 1. Using Metrics Server to Monitor Node and Pod Performance

The **metrics-server** collects resource usage data from the kubelets and provides aggregated metrics for Pods and nodes.

### 🔹 Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### 🔹 Verify installation

```bash
kubectl get deployment metrics-server -n kube-system
```

### 🔹 View node and Pod metrics

```bash
# Node-level metrics
kubectl top nodes

# Pod-level metrics
kubectl top pods -A
```

Metrics like **CPU** and **memory usage** help identify resource bottlenecks and guide scaling decisions.

---

## 2. Backing Up the etcd Database

**etcd** stores the entire cluster state — backing it up is critical before upgrades or maintenance.

### 🔹 Snapshot the etcd data

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/backup/etcd-snapshot.db
```

### 🔹 Verify the snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot status /opt/backup/etcd-snapshot.db
```

> Always store snapshots in a secure, versioned location (e.g., NFS, S3).

---

## 3. Restoring the etcd Database

If etcd data becomes corrupted or lost, restore from a snapshot.

### 🔹 Restore snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot restore /opt/backup/etcd-snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

### 🔹 Update kubelet manifest

Edit the etcd Pod manifest (usually `/etc/kubernetes/manifests/etcd.yaml`) to use the restored data directory:

```yaml
- --data-dir=/var/lib/etcd-from-backup
```

### 🔹 Verify etcd recovery

```bash
kubectl get nodes
kubectl get pods -A
```

---

## 4. Performing Cluster Node Upgrades

Upgrading control plane and worker nodes ensures security patches and feature updates are applied safely.

### 🔹 Upgrade the control plane

```bash
# View available versions
kubeadm version
apt update && apt list -a kubeadm

# Upgrade kubeadm first
sudo apt install -y kubeadm=<version>

# Apply the upgrade
sudo kubeadm upgrade apply <version>
```

### 🔹 Upgrade kubelet and kubectl

```bash
sudo apt install -y kubelet=<version> kubectl=<version>
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

## 5. Performing Worker Node Upgrades

### 🔹 Drain the node

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

### 🔹 Upgrade worker components

```bash
sudo apt install -y kubeadm=<version>
sudo kubeadm upgrade node
sudo apt install -y kubelet=<version> kubectl=<version>
sudo systemctl restart kubelet
```

### 🔹 Bring node back online

```bash
kubectl uncordon <node-name>
```

---

## 6. Understanding Cluster High Availability (HA)

High availability ensures your cluster continues running even if one or more control plane nodes fail.
In an HA setup:

* Multiple **control plane nodes** share responsibility for cluster management.
* **etcd** runs as a replicated cluster.
* A **load balancer** distributes traffic among API servers.

> HA minimizes downtime and supports production-grade reliability.

---

## 7. Setting Up a Highly Available Kubernetes Cluster

### 🔹 Architecture Overview

```
[Clients]
   |
[Load Balancer]
   |
---------------------------------------------------
| Control Plane 1 | Control Plane 2 | Control Plane 3 |
| etcd            | etcd            | etcd            |
---------------------------------------------------
          |                 |                |
         Worker Nodes (kubelet + kube-proxy)
```

### 🔹 Steps to deploy HA

1. **Provision multiple control plane nodes**
2. **Install kubeadm, kubelet, kubectl** on all nodes
3. **Initialize the first control plane node**

   ```bash
   kubeadm init --control-plane-endpoint "LOADBALANCER_DNS:6443" --upload-certs
   ```
4. **Join additional control planes**

   ```bash
   kubeadm join LOADBALANCER_DNS:6443 --control-plane --certificate-key <key>
   ```
5. **Join worker nodes**

   ```bash
   kubeadm join LOADBALANCER_DNS:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

### 🔹 Verify the cluster

```bash
kubectl get nodes
kubectl get pods -A
```

---

## Summary

Performing node maintenance keeps your Kubernetes cluster **secure, consistent, and reliable**.
Key actions include:

* Monitoring with **metrics-server**
* Backing up and restoring **etcd**
* Performing **safe upgrades** for control and worker nodes
* Designing **highly available** architectures for production resilience.

Would you like me to split this into separate `.md` files (e.g., `node-maintenance.md`, `etcd-backup.md`, `ha-setup.md`) so it fits your existing repo structure?
