```bash
k8s-deployments-complete-guide.md
```

---

# Kubernetes Deployments – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* What is a Deployment
* How it manages ReplicaSets
* Rolling updates
* Rollback
* Scaling
* Update strategy
* Zero-downtime deployment
* Real-world verification commands

---

# 2️⃣ What is a Deployment?

Deployment is a **higher-level controller** that manages:

* ReplicaSets
* Rolling updates
* Rollbacks
* Version history

You don’t directly manage ReplicaSets in production.
Deployment creates and controls them.

---

# 3️⃣ Deployment Architecture Flow

```text
Deployment
    ↓
Creates ReplicaSet
    ↓
ReplicaSet creates Pods
    ↓
Deployment monitors ReplicaSet state
```

During updates:

```text
New Deployment Version
        ↓
New ReplicaSet created
        ↓
Old ReplicaSet scaled down
        ↓
New ReplicaSet scaled up
```

---

# 4️⃣ Basic Deployment Example

Create file: `deploy-basic.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f deploy-basic.yaml
```

Verify:

```bash
kubectl get deploy
kubectl get rs
kubectl get pods
```

You will see:

* 1 Deployment
* 1 ReplicaSet
* 3 Pods

---

# 5️⃣ Understanding Ownership

Check pod ownership:

```bash
kubectl describe pod <pod-name>
```

You’ll see:

```text
Controlled By: ReplicaSet/xxxx
```

Check ReplicaSet ownership:

```bash
kubectl describe rs <rs-name>
```

You’ll see:

```text
Controlled By: Deployment/nginx-deploy
```

✔ Hierarchy confirmed.

---

# 6️⃣ Scaling Deployment

Scale to 5 replicas:

```bash
kubectl scale deployment nginx-deploy --replicas=5
```

Verify:

```bash
kubectl get pods
```

Pods increase automatically.

---

# 7️⃣ Rolling Update (Very Important)

Update image:

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.26
```

Watch rollout:

```bash
kubectl rollout status deployment nginx-deploy
```

Watch live:

```bash
kubectl get pods -w
```

You’ll see:

* New ReplicaSet created
* Old pods terminated gradually
* New pods created gradually

✔ Zero downtime rolling update.

---

# 8️⃣ Check Rollout History

```bash
kubectl rollout history deployment nginx-deploy
```

You’ll see revision numbers.

---

# 9️⃣ Rollback Deployment

Rollback to previous version:

```bash
kubectl rollout undo deployment nginx-deploy
```

Verify:

```bash
kubectl rollout status deployment nginx-deploy
```

Deployment recreates old ReplicaSet.

---

# 🔟 Deployment Strategy

Default strategy:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 25%
    maxSurge: 25%
```

Meaning:

* maxUnavailable → how many pods can go down
* maxSurge → how many extra pods can be created temporarily

---

# 1️⃣1️⃣ Custom Strategy Example

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 2
```

This gives more control during upgrades.

---

# 1️⃣2️⃣ Recreate Strategy (No Rolling)

```yaml
strategy:
  type: Recreate
```

All old pods are deleted first.
Then new pods are created.

⚠ Causes downtime.

Used rarely.

---

# 1️⃣3️⃣ Test Failure Scenario

Set wrong image:

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:wrongversion
```

Check rollout:

```bash
kubectl rollout status deployment nginx-deploy
```

Pods will fail.

Fix:

```bash
kubectl rollout undo deployment nginx-deploy
```

This is real production scenario behavior.

---

# 1️⃣4️⃣ Observe ReplicaSets During Update

Check:

```bash
kubectl get rs
```

You’ll see:

* Old RS (scaled down)
* New RS (active)

Deployment keeps old RS for rollback.

---

# 1️⃣5️⃣ Resource Example Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resource-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: resource-demo
  template:
    metadata:
      labels:
        app: resource-demo
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "400m"
            memory: "512Mi"
```

Apply and verify normally.

---

# 1️⃣6️⃣ Useful Debug Commands

Check deployment:

```bash
kubectl describe deployment nginx-deploy
```

Watch rollout:

```bash
kubectl rollout status deployment nginx-deploy
```

Check events:

```bash
kubectl get events
```

Check RS:

```bash
kubectl get rs
```

---

# 1️⃣7️⃣ Deployment vs ReplicaSet

| ReplicaSet              | Deployment          |
| ----------------------- | ------------------- |
| Maintains replica count | Manages ReplicaSets |
| No safe updates         | Rolling updates     |
| No rollback             | Rollback support    |
| Basic controller        | Production-grade    |

---

# 1️⃣8️⃣ Clean Up

```bash
kubectl delete deployment nginx-deploy
kubectl delete deployment resource-deploy
```

ReplicaSets and Pods are deleted automatically.

---

# Final Mental Model

Deployment = Application Manager
ReplicaSet = Pod Counter
Pod = Container Wrapper

Deployment ensures:

* Desired replicas
* Safe updates
* Rollback capability
* Zero downtime

---

If this is clear now, your next logical step is:

* Services (how traffic reaches pods)
* Or ConfigMaps & Secrets
* Or Ingress
