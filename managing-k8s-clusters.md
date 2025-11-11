# Managing Kubernetes Clusters

This guide covers essential node-level management operations in Kubernetes, including analyzing nodes, managing containers with `crictl`, running static Pods, controlling node state, and managing node services.

---

## 1. Analyzing Cluster Nodes

Cluster nodes are the backbone of Kubernetes. You can inspect their health, configuration, and resource usage directly with `kubectl`.

### List and Describe Nodes

```bash
# List all cluster nodes
kubectl get nodes

# Detailed information about a specific node
kubectl describe node <node-name>
```

### 💡 Check Node Conditions

Look for:

* **Ready / NotReady** → Node health
* **MemoryPressure / DiskPressure** → Resource constraints
* **Taints / Labels** → Scheduling behavior

To view node resource utilization:

```bash
kubectl top nodes
```

---

## 2. Using `crictl` to Manage Node Containers

`crictl` lets you interact directly with the **Container Runtime Interface (CRI)**.
It’s useful when `kubectl` can’t reach the API server or for low-level debugging.

### Basic Usage

```bash
# List all running containers on the node
sudo crictl ps

# List all containers (including stopped ones)
sudo crictl ps -a

# Inspect a specific container
sudo crictl inspect <container-id>

# View container logs
sudo crictl logs <container-id>
```

### Managing Containers

```bash
# Stop a running container
sudo crictl stop <container-id>

# Remove a stopped container
sudo crictl rm <container-id>

# View available images
sudo crictl images

# Remove an unused image
sudo crictl rmi <image-id>
```

These commands allow direct control over node containers without relying on the Kubernetes control plane.

---

## 3. Running Static Pods

**Static Pods** are managed directly by the **kubelet**, not the API server.
They’re ideal for critical components like control plane services or node-local agents.

### Configure Static Pods

1. Find the static Pod manifest path (default: `/etc/kubernetes/manifests`):

   ```bash
   sudo cat /var/lib/kubelet/config.yaml | grep staticPodPath
   ```

2. Create a manifest file in that directory:

   ```bash
   sudo vi /etc/kubernetes/manifests/monitor-agent.yaml
   ```

3. Example manifest:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: monitor-agent
     namespace: kube-system
   spec:
     containers:
     - name: agent
       image: busybox
       command: ["sh", "-c", "while true; do echo 'Node healthy'; sleep 10; done"]
   ```

4. Verify kubelet created the Pod:

   ```bash
   kubectl get pods -n kube-system -o wide
   ```

Static Pods restart automatically if deleted and always run on their host node.

---

## 4. Managing Node State

Sometimes you need to temporarily remove nodes from scheduling (for upgrades or maintenance).

### Cordon / Drain / Uncordon

```bash
# Mark node unschedulable (new Pods won’t be scheduled)
kubectl cordon <node-name>

# Safely evict Pods and prepare node for maintenance
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data

# Restore node to schedulable state
kubectl uncordon <node-name>
```

### Check Node Readiness

```bash
kubectl get nodes
```

Look at the **STATUS** column for `Ready`, `NotReady`, or `SchedulingDisabled`.

---

## 5. Managing Node Services

Each node runs several key services that keep it functional:

| Service                | Description                                                      |
| ---------------------- | ---------------------------------------------------------------- |
| **kubelet**            | Registers node and manages Pods                                  |
| **kube-proxy**         | Maintains network rules for services                             |
| **containerd / CRI-O** | Container runtime responsible for pulling and running containers |

### Check and Restart Services

```bash
# Check kubelet service
sudo systemctl status kubelet

# Restart kubelet if misbehaving
sudo systemctl restart kubelet

# Check container runtime
sudo systemctl status containerd

# Restart runtime service
sudo systemctl restart containerd
```

### 📊 View Logs

```bash
# View kubelet logs
sudo journalctl -u kubelet -f

# View container runtime logs
sudo journalctl -u containerd -f
```

---

## Conclusion

Managing Kubernetes clusters effectively means mastering node-level control.
By:

* **Analyzing nodes** with `kubectl`,
* **Debugging containers** using `crictl`,
* **Running static Pods** for essential workloads,
* **Controlling node state** safely, and
* **Maintaining node services** diligently,

you can ensure a stable, resilient, and maintainable Kubernetes environment.
