# kubernetes-from-ground-up

# **1. Understanding Cluster Node Requirements**

Before setting up a cluster, you need to understand **node roles** and **resource requirements**:

### 🔹 Node Types

| Node Type                  | Role                                                                       |
| -------------------------- | -------------------------------------------------------------------------- |
| **Control Plane (Master)** | Runs `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `etcd` |
| **Worker Node**            | Runs application workloads (Pods, Services)                                |

### 🔹 Minimum Requirements (small cluster)

| Resource | Control Plane            | Worker Node |
| -------- | ------------------------ | ----------- |
| CPU      | 2+ cores                 | 1+ cores    |
| RAM      | 2+ GB                    | 1+ GB       |
| Disk     | 20+ GB                   | 20+ GB      |
| Network  | Private IP, no conflicts | Same        |

* Use an **odd number of control plane nodes** for HA (3,5,7) to maintain etcd quorum.
* Worker nodes can outnumber control plane nodes depending on workload.

---

# **2. Provisioning Infrastructure for Hosting Kubernetes**

* **Virtual Machines / Physical Machines / Cloud Instances**

  * Example: AWS EC2, Azure VM, bare metal, VMware VMs.
* **Networking**

  * All nodes must communicate privately.
  * Ensure proper firewall rules: open ports for kubelet, kube-apiserver, etc.
* **Load Balancer (for HA clusters)**

  * Provides a single endpoint for API server.
  * Can be HAProxy, Nginx, or cloud LB.

---

# **3. Installation Procedure Overview**

1. Prepare OS on all nodes (Ubuntu/CentOS/RHEL)
2. Configure Linux kernel for networking
3. Install container runtime (containerd, CRI-O)
4. Install Kubernetes components (`kubeadm`, `kubectl`, `kubelet`)
5. Initialize control plane with `kubeadm init`
6. Configure networking (CNI plugin)
7. Join worker nodes using `kubeadm join`

---

# **4. Configuring Linux Kernel Settings for Kubernetes Networking**

Kubernetes requires certain kernel settings for networking:

```bash
# Load br_netfilter module
sudo modprobe br_netfilter

# Persist settings
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply changes
sudo sysctl --system
```

* Ensures kube-proxy and CNI plugins can manage traffic.

---

# **5. Installing Container Runtime Interface (CRI) and Tools**

* Kubernetes requires a container runtime like **containerd** or **CRI-O**.

### Example: Installing containerd

```bash
sudo apt update && sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

* Ensure runtime is running:

```bash
sudo systemctl status containerd
```

---

# **6. Installing Kubernetes Components**

```bash
sudo apt update && sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

* `kubelet` → runs on all nodes
* `kubeadm` → bootstrap Kubernetes cluster
* `kubectl` → Kubernetes client

---

# **7. Using `kubeadm init`**

On the **first control plane node**:

```bash
sudo kubeadm init \
  --pod-network-cidr=192.168.0.0/16
```

* `--pod-network-cidr` depends on your CNI plugin (Calico, Flannel, etc.)
* After success, it outputs:

  * **Join command for worker nodes**
  * **Join command for additional control plane nodes**
  * `kubeadm join ... --control-plane` (for HA control planes)

---

# **8. Configuring the Kubernetes Client**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```

* Verifies the control plane is running and `kubectl` can communicate.

---

# **9. Setting Up Node Networking**

* Kubernetes requires a CNI plugin (Container Network Interface) for Pod networking.
* Example: **Calico**

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

* Verify pods in `kube-system` namespace are running:

```bash
kubectl get pods -n kube-system
```

---

# **10. Adding Nodes to the Cluster**

* On each **worker node**, use the join command provided by `kubeadm init`:

```bash
sudo kubeadm join <LOADBALANCER_IP>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

* Node registers with the control plane and starts running workloads.

* **Add additional control plane nodes** (for HA):

```bash
sudo kubeadm join <LOADBALANCER_IP>:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane --certificate-key <key>
```

---

# **11. Using `kubeadm init` with a Configuration File**

Instead of command-line options, you can use a **YAML config file** for better reproducibility:

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: "192.168.1.10"
  bindPort: 6443
nodeRegistration:
  name: "master1"
  kubeletExtraArgs:
    cgroup-driver: systemd
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: v1.28.0
controlPlaneEndpoint: "LOADBALANCER:6443"
networking:
  podSubnet: "192.168.0.0/16"
```

* Run with:

```bash
sudo kubeadm init --config kubeadm-config.yaml
```

* Benefits:

  * Version control for cluster configs
  * Easily reproduce cluster setup
  * Supports HA cluster bootstrap

---

# ✅ **Summary Workflow**

1. **Prepare nodes** → resources, OS, networking
2. **Configure Linux kernel** → networking modules
3. **Install container runtime** → containerd/CRI-O
4. **Install kubeadm, kubelet, kubectl**
5. **Initialize first control plane** → `kubeadm init`
6. **Configure kubectl**
7. **Install CNI plugin**
8. **Join additional control plane nodes** (for HA)
9. **Join worker nodes**
10. Optional: **Use kubeadm config file** for reproducibility
