# Auto-Provisioning Storage

Kubernetes supports **dynamic storage provisioning**, which allows Pods to request storage without manually creating PersistentVolumes. This is managed using **StorageClasses** and **provisioners**, enabling scalable and automated storage allocation.

---

## 1. Using StorageClass

A **StorageClass** defines how storage should be dynamically provisioned, including the type of storage and reclaim policies.

### 🔹 Example StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

* `provisioner`: Determines the storage backend
* `reclaimPolicy`: What happens to the volume after PVC deletion (`Delete`, `Retain`)
* `volumeBindingMode`: Controls when PVs are bound to PVCs

Create a PVC using this StorageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-auto
spec:
  storageClassName: fast-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

The PV is **automatically provisioned** when the PVC is applied.

---

## 2. Understanding Storage Provisioners

**Provisioners** are responsible for creating storage dynamically. Common provisioners include:

* `kubernetes.io/aws-ebs` – AWS Elastic Block Store
* `kubernetes.io/gce-pd` – Google Persistent Disk
* `kubernetes.io/cinder` – OpenStack Cinder
* `kubernetes.io/no-provisioner` – For pre-existing storage like NFS

Provisioners allow Kubernetes to **abstract storage backends**, making Pods portable across clusters.

---

## 3. Setting Up a NFS Storage Provisioner

**NFS provisioner** provides dynamic storage using an existing NFS server.

### 🔹 Example Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-provisioner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-provisioner
  template:
    metadata:
      labels:
        app: nfs-provisioner
    spec:
      containers:
      - name: nfs-provisioner
        image: quay.io/external_storage/nfs-client-provisioner:latest
        env:
        - name: PROVISIONER_NAME
          value: example.com/nfs
        - name: NFS_SERVER
          value: 10.0.0.1
        - name: NFS_PATH
          value: /exported/path
        volumeMounts:
        - name: nfs-root
          mountPath: /persistentvolumes
      volumes:
      - name: nfs-root
        nfs:
          server: 10.0.0.1
          path: /exported/path
```

Create a StorageClass using the provisioner:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: example.com/nfs
reclaimPolicy: Delete
```

Now, PVCs requesting `nfs-storage` are **dynamically created and bound** to NFS volumes.

---

## Summary

Auto-provisioning storage in Kubernetes allows **dynamic volume creation** without manual PV management:

* Use **StorageClass** to define dynamic storage behavior
* Provisioners handle **backend storage creation**
* NFS provisioners allow **network-shared storage** for dynamic workloads

This setup simplifies storage management, scales automatically with workload demands, and integrates seamlessly with PVCs.
