# Managing Security Settings

Security in Kubernetes revolves around controlling **who** can access the cluster, **what** they can do, and **under what conditions** Pods and containers run.
This section explains how to manage API access, configure security contexts, and set up **Role-Based Access Control (RBAC)** for users and service accounts.

---

## 1. Understanding API Access

The **Kubernetes API Server** is the central communication hub for all cluster operations. Every request — whether from `kubectl`, a Pod, or an external user — goes through it.

You can view the API endpoints with:

```bash
kubectl get --raw / | jq .
```

Access is secured using:

* **Authentication** — Verifies identity (users, service accounts, etc.)
* **Authorization** — Determines allowed actions
* **Admission Control** — Enforces policies before objects are persisted

To inspect who you are authenticated as:

```bash
kubectl config view --minify | grep name:
```

---

## 2. Managing Security Contexts

A **Security Context** defines privilege and access control settings for a Pod or container — such as user IDs, file permissions, and capabilities.

Example Pod manifest with a security context:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: nginx
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

Apply the manifest:

```bash
kubectl apply -f secure-pod.yaml
```

Check the Pod’s security configuration:

```bash
kubectl describe pod secure-pod | grep -A5 Security
```

Security contexts help **enforce least privilege**, ensuring containers run as non-root and with minimal capabilities.

---

## 3. Users, Service Accounts, and API Access

Kubernetes distinguishes between:

* **Users** — External entities (humans or systems) authenticated via certificates or identity providers.
* **Service Accounts** — In-cluster identities used by Pods to communicate with the API server.

### 🔹 View service accounts

```bash
kubectl get serviceaccounts -n kube-system
```

### 🔹 Create a new service account

```bash
kubectl create serviceaccount deployer -n default
```

### 🔹 Assign service account to a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: deploy-pod
spec:
  serviceAccountName: deployer
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo 'Running as deployer'; sleep 3600"]
```

### 🔹 Inspect current access permissions

```bash
kubectl auth can-i create pods --as=system:serviceaccount:default:deployer
```

---

## 4. Understanding Role-Based Access Control (RBAC)

RBAC determines **who can perform what actions** on which Kubernetes resources.

Core RBAC components:

| Component              | Description                                                             |
| ---------------------- | ----------------------------------------------------------------------- |
| **Role**               | Defines permissions (verbs on resources) within a namespace             |
| **ClusterRole**        | Like Role, but applies cluster-wide                                     |
| **RoleBinding**        | Grants Role permissions to users or service accounts within a namespace |
| **ClusterRoleBinding** | Grants ClusterRole permissions across the cluster                       |

View current RBAC rules:

```bash
kubectl get roles,clusterroles,rolebindings,clusterrolebindings -A
```

---

## 5. Setting Up RBAC for Service Accounts

Example: Allow a service account to manage Pods within a namespace.

### 🔹 Create Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-manager
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "delete"]
```

### 🔹 Create RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-manager-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: deployer
  namespace: default
roleRef:
  kind: Role
  name: pod-manager
  apiGroup: rbac.authorization.k8s.io
```

Apply both:

```bash
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
```

Verify:

```bash
kubectl auth can-i create pods --as=system:serviceaccount:default:deployer
```

---

## 6. Configuring ClusterRoles and ClusterRoleBindings

For permissions that span across namespaces (e.g., managing nodes or PersistentVolumes), use ClusterRoles.

Example ClusterRole and ClusterRoleBinding:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: [""]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-reader-binding
subjects:
- kind: User
  name: devops-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f clusterrole.yaml
```

---

## 7. Setting Up RBAC for Users

Users typically authenticate via **client certificates** or external identity providers (OIDC, LDAP, etc.).

### 🔹 Create a client certificate for a user

```bash
openssl genrsa -out devops.key 2048
openssl req -new -key devops.key -out devops.csr -subj "/CN=devops/O=devops-group"
sudo openssl x509 -req -in devops.csr -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out devops.crt -days 365
```

### 🔹 Add the user to kubeconfig

```bash
kubectl config set-credentials devops --client-certificate=devops.crt --client-key=devops.key
kubectl config set-context devops@kubernetes --cluster=kubernetes --user=devops
kubectl config use-context devops@kubernetes
```

### 🔹 Bind RBAC role

Use a `ClusterRoleBinding` to grant permissions:

```bash
kubectl create clusterrolebinding devops-admin \
  --clusterrole=admin \
  --user=devops
```

Validate access:

```bash
kubectl auth can-i delete pods --user=devops
```

---

## Summary

Kubernetes security management centers on **API access control** and **RBAC enforcement**.
By:

* Understanding how API authentication and authorization work,
* Applying **SecurityContexts** to enforce least privilege,
* Managing **ServiceAccounts** for Pod-level access, and
* Using **RBAC** to precisely control user and account permissions,

you ensure your cluster remains secure, auditable, and compliant with best practices.
