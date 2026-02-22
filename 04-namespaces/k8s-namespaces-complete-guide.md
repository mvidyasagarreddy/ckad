```bash
k8s-namespaces-complete-guide.md
```

---

# Kubernetes Namespaces – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* What is a Namespace
* Why it is needed
* Default namespaces
* Resource isolation
* RBAC per namespace
* ResourceQuota & LimitRange
* Context switching
* Practical verification steps

---

# 2️⃣ What is a Namespace?

A Namespace is a **logical partition inside a Kubernetes cluster**.

It helps:

* Organize resources
* Isolate environments
* Apply RBAC rules
* Apply resource limits
* Avoid name conflicts

---

# 3️⃣ Default Namespaces in Every Cluster

Run:

```bash
kubectl get namespaces
```

You’ll see:

| Namespace       | Purpose                                 |
| --------------- | --------------------------------------- |
| default         | Where resources go if you don’t specify |
| kube-system     | System components                       |
| kube-public     | Public cluster info                     |
| kube-node-lease | Node heartbeats                         |

Never deploy apps in `kube-system`.

---

# 4️⃣ Basic Namespace Creation

Create file: `ns-dev.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

Apply:

```bash
kubectl apply -f ns-dev.yaml
```

Verify:

```bash
kubectl get ns
```

---

# 5️⃣ Deploy Pod Inside Namespace

Create file: `pod-dev.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-dev
  namespace: dev
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f pod-dev.yaml
```

Verify:

```bash
kubectl get pods -n dev
```

Without `-n dev`, you won’t see it.

---

# 6️⃣ Namespace Isolation

Create same pod name in another namespace:

Create namespace:

```bash
kubectl create ns prod
```

Create same pod in prod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-dev
  namespace: prod
spec:
  containers:
  - name: nginx
    image: nginx
```

Now:

```bash
kubectl get pods -n dev
kubectl get pods -n prod
```

Same pod name allowed in different namespaces.

✔ Namespaces isolate names.

---

# 7️⃣ Set Default Namespace for kubectl

Instead of using `-n dev` every time:

```bash
kubectl config set-context --current --namespace=dev
```

Now:

```bash
kubectl get pods
```

Will show dev namespace pods.

Check current namespace:

```bash
kubectl config view --minify | grep namespace
```

---

# 8️⃣ ResourceQuota Per Namespace

Limit total CPU/memory for namespace.

Create file: `quota-dev.yaml`

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
    pods: "5"
```

Apply:

```bash
kubectl apply -f quota-dev.yaml
```

Verify:

```bash
kubectl describe quota dev-quota -n dev
```

Now try creating 6 pods → last one will fail.

✔ Namespace-level resource control.

---

# 9️⃣ LimitRange Per Namespace

Set default limits if user forgets.

Create file: `limitrange-dev.yaml`

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: dev
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"
```

Apply:

```bash
kubectl apply -f limitrange-dev.yaml
```

Now create pod without resources → it automatically gets defaults.

Verify:

```bash
kubectl describe pod <pod-name> -n dev
```

---

# 🔟 RBAC Per Namespace

Create ServiceAccount in dev:

```bash
kubectl create sa dev-sa -n dev
```

Create Role in dev:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

Bind it:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: dev-sa
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Now this SA can read pods only in `dev`, not `prod`.

Test:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:dev:dev-sa -n dev
kubectl auth can-i list pods --as=system:serviceaccount:dev:dev-sa -n prod
```

---

# 1️⃣1️⃣ Cross-Namespace Communication

Pods in different namespaces:

DNS format:

```text
service-name.namespace.svc.cluster.local
```

Example:

```text
backend.dev.svc.cluster.local
```

Namespaces isolate logically — not network by default.

For network isolation, use **NetworkPolicy**.

---

# 1️⃣2️⃣ Delete Namespace

```bash
kubectl delete ns dev
```

⚠ This deletes EVERYTHING inside namespace.

Pods
Deployments
Services
ConfigMaps
Secrets

Everything.

---

# 1️⃣3️⃣ Important Concepts

Namespaces isolate:

* Names
* RBAC
* ResourceQuota
* LimitRange

Namespaces do NOT isolate:

* Nodes
* Cluster-wide resources (like PersistentVolumes)
* Network (unless NetworkPolicy used)

---

# 1️⃣4️⃣ When to Use Namespaces

Common pattern:

| Namespace  | Purpose             |
| ---------- | ------------------- |
| dev        | Development         |
| qa         | Testing             |
| staging    | Pre-production      |
| prod       | Production          |
| monitoring | Prometheus, Grafana |
| logging    | EFK stack           |

---

# 1️⃣5️⃣ Clean Up Lab

```bash
kubectl delete ns dev
kubectl delete ns prod
```

---

# Final Mental Model

Cluster = Apartment Building
Namespace = Separate Flats
Pods = People inside flats

They share building infrastructure
But live in separate logical spaces.

---

If this is clear now, your foundation is getting strong.

Next natural step:

* Services (networking)
* ConfigMaps & Secrets
* Ingress
* NetworkPolicies
* Or full production architecture flow
