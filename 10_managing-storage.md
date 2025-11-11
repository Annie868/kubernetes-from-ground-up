# Managing Storage

Storage in Kubernetes ensures that applications can **persist data beyond the lifecycle of a Pod**. Proper storage management allows Pods to access files, configuration data, and secrets reliably, while supporting dynamic provisioning and security.

---

## 1. Understanding Kubernetes Storage Options

Kubernetes provides several storage types:

* **Ephemeral Volumes** – exist only for the Pod’s lifetime (`emptyDir`)
* **Persistent Volumes (PV)** – cluster-wide storage resources
* **Persistent Volume Claims (PVC)** – Pods request PVs dynamically
* **ConfigMaps and Secrets** – store configuration and sensitive data

---

## 2. Accessing Storage Through Pod Volumes

Pods can mount volumes to access data:

### 🔹 Example: Mount an emptyDir volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-volume
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: html-volume
  volumes:
  - name: html-volume
    emptyDir: {}
```

This volume exists **only while the Pod is running**.

---

## 3. Configuring PersistentVolume (PV) Storage

A **PersistentVolume** represents a storage resource in the cluster.

### 🔹 Example: Define a PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

* `accessModes` can be `ReadWriteOnce`, `ReadOnlyMany`, or `ReadWriteMany`
* `reclaimPolicy` controls what happens when a PVC is deleted (`Retain`, `Recycle`, `Delete`)

---

## 4. Configuring PersistentVolumeClaims (PVCs)

A **PersistentVolumeClaim** allows Pods to request storage:

### 🔹 Example PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Bind a PV to PVC automatically if a matching PV exists.

---

## 5. Configuring Pod Storage with PVs and PVCs

Mount a PVC into a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-example
```

This allows **persistent storage even if the Pod is deleted or restarted**.

---

## 6. Using Volume Reclaim Policies

* **Retain**: Keep PV data after PVC deletion
* **Recycle**: Clear data before reuse (deprecated)
* **Delete**: Remove underlying storage automatically

Check PV status:

```bash
kubectl get pv
kubectl describe pv pv-example
```

---

## 7. Using ConfigMaps and Secrets as Volumes

Pods can mount **ConfigMaps** or **Secrets** as files:

### 🔹 ConfigMap Volume Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
    key: value
```

Mount in Pod:

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
volumeMounts:
- mountPath: /etc/config
  name: config-volume
```

Secrets work similarly for **sensitive data** like passwords and API keys.

---

## Summary

Managing storage in Kubernetes involves:

* Understanding ephemeral vs persistent storage
* Configuring **PV and PVC** for persistent data
* Mounting volumes in Pods for **application access**
* Managing **reclaim policies** for lifecycle control
* Using **ConfigMaps and Secrets** as volumes for configuration and sensitive data

Proper storage management ensures applications have **reliable, secure, and scalable access to data** across Pod lifecycles.
