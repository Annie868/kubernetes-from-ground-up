# Using Templating Tools

Managing Kubernetes applications efficiently often requires **templating and packaging tools**. They help you deploy repeatable, configurable applications, reduce manual YAML edits, and simplify upgrades. This section covers **raw YAML deployment, Helm, and Kustomize**.

---

## 1. Running Applications from YAML Files

The simplest way to deploy Kubernetes resources is by applying **YAML manifests** directly.

### 🔹 Example: Deploy an Nginx Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
```

Apply the manifest:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
```

> Pros: Full control over every detail.
> Cons: Hard to manage for large applications or multiple environments.

---

## 2. The Helm Package Manager

**Helm** is the de facto package manager for Kubernetes. It allows you to **deploy, upgrade, and rollback applications** using charts. Charts are pre-packaged sets of Kubernetes manifests with configurable values.

### 🔹 Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### 🔹 Add a chart repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### 🔹 Install an application (example: Nginx)

```bash
helm install my-nginx bitnami/nginx
helm list
```

---

## 3. Creating a Template from a Helm Chart

You can generate Kubernetes manifests from a Helm chart **without deploying**, which is useful for inspection or customization.

```bash
helm template my-nginx bitnami/nginx --values custom-values.yaml > rendered.yaml
```

* `--values custom-values.yaml` allows overriding defaults
* `rendered.yaml` contains fully expanded Kubernetes manifests ready for review

---

## 4. Managing Applications with Helm

Helm makes **upgrades, rollbacks, and uninstalling** seamless:

### 🔹 Upgrade an application

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
```

### 🔹 Rollback to a previous release

```bash
helm rollback my-nginx 1
```

### 🔹 Uninstall an application

```bash
helm uninstall my-nginx
```

Helm keeps track of **release history** for easy rollbacks and auditing.

---

## 5. Using Kustomize

**Kustomize** allows you to **customize plain YAML manifests** without templates. It is built into `kubectl` (`kubectl apply -k`).

### 🔹 Example: Directory structure

```
my-app/
├── deployment.yaml
├── service.yaml
└── kustomization.yaml
```

### 🔹 `kustomization.yaml` example

```yaml
resources:
  - deployment.yaml
  - service.yaml

namePrefix: dev-
commonLabels:
  app: my-app
replicas:
  deployment/nginx-deployment: 3
```

### 🔹 Apply with Kustomize

```bash
kubectl apply -k my-app/
```

Kustomize is ideal for **environment-specific configurations** (dev, staging, prod) while keeping base manifests unchanged.

---

## Summary

Templating tools make Kubernetes application management **scalable and maintainable**.

* **Raw YAML**: Full control, good for small apps or learning
* **Helm**: Package, install, upgrade, and rollback applications with ease
* **Kustomize**: Customize manifests for different environments without modifying the base YAML

Together, these tools help streamline deployment workflows and reduce human error.

Do you want me to do that?
