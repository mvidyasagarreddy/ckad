```bash
k8s-replicasets-complete-guide.md
```

---

# Kubernetes ReplicaSets – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* What is a ReplicaSet
* Why we need it
* How it maintains desired replicas
* Label selector behavior
* Scaling
* Self-healing behavior
* How it differs from Pod and Deployment
* Practical verification steps

---

# 2️⃣ What is a ReplicaSet?

A ReplicaSet ensures:

> A specified number of identical Pods are always running.

If:

* A Pod crashes → ReplicaSet creates new one.
* A Pod is deleted manually → ReplicaSet recreates it.
* You scale replicas → It adjusts automatically.

ReplicaSet does **NOT** provide rolling updates properly.
That’s why Deployments use ReplicaSets internally.

---

# 3️⃣ Basic ReplicaSet Example

Create file: `rs-basic.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
        image: nginx
```

Apply:

```bash
kubectl apply -f rs-basic.yaml
```

Verify:

```bash
kubectl get rs
kubectl get pods
```

Expected:
3 Pods running.

---

# 4️⃣ Understanding How ReplicaSet Works

Important section:

```yaml
selector:
  matchLabels:
    app: nginx
```

ReplicaSet controls Pods that have:

```yaml
labels:
  app: nginx
```

If labels don’t match → ReplicaSet won’t manage them.

---

# 5️⃣ Self-Healing Test

Delete one Pod manually:

```bash
kubectl get pods
kubectl delete pod <one-pod-name>
```

Immediately check:

```bash
kubectl get pods
```

You will see:

* Old pod gone
* New pod created automatically

✔ This proves self-healing behavior.

---

# 6️⃣ Scaling ReplicaSet

Scale to 5 replicas:

```bash
kubectl scale rs nginx-rs --replicas=5
```

Verify:

```bash
kubectl get pods
```

Now 5 pods running.

Scale down:

```bash
kubectl scale rs nginx-rs --replicas=2
```

Extra pods are terminated.

---

# 7️⃣ Label Selector Experiment (Very Important)

Create a standalone Pod with same label:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply it.

Now check:

```bash
kubectl get pods
```

You’ll notice:
ReplicaSet may adopt this pod.

Because label matches selector.

⚠️ This is why label design is important.

---

# 8️⃣ What Happens Internally

Flow:

```
ReplicaSet Created
        ↓
Controller watches cluster
        ↓
Counts matching pods
        ↓
If count < replicas → Create pod
If count > replicas → Delete pod
```

ReplicaSet continuously reconciles desired state.

---

# 9️⃣ ReplicaSet With Resources

Create file: `rs-resources.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: resource-rs
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

Apply and verify:

```bash
kubectl describe rs resource-rs
kubectl get pods
```

---

# 🔟 Verify Ownership

Check which ReplicaSet owns a Pod:

```bash
kubectl describe pod <pod-name>
```

Look for:

```
Controlled By:  ReplicaSet/nginx-rs
```

That proves controller relationship.

---

# 1️⃣1️⃣ ReplicaSet vs Pod

| Pod             | ReplicaSet         |
| --------------- | ------------------ |
| Runs 1 instance | Maintains multiple |
| No self-healing | Self-healing       |
| Manual scaling  | Automatic scaling  |

---

# 1️⃣2️⃣ ReplicaSet vs Deployment

| ReplicaSet                | Deployment          |
| ------------------------- | ------------------- |
| Basic scaling             | Rolling updates     |
| No rollout history        | Has rollout history |
| Manual image update risky | Safe updates        |

In real production:
You rarely create ReplicaSet directly.
Deployment manages ReplicaSets for you.

---

# 1️⃣3️⃣ Observe Controller Behavior

Check events:

```bash
kubectl describe rs nginx-rs
```

Check current state:

```bash
kubectl get rs -o wide
```

Watch in real-time:

```bash
kubectl get pods -w
```

Now delete a pod and watch recreation live.

---

# 1️⃣4️⃣ Failure Scenario Test

Edit RS to very high replicas:

```bash
kubectl scale rs nginx-rs --replicas=100
```

If cluster resources insufficient:

Check:

```bash
kubectl describe rs nginx-rs
```

You’ll see scheduling failures.

This shows scheduler + controller interaction.

---

# 1️⃣5️⃣ Clean Up

```bash
kubectl delete rs nginx-rs
kubectl delete rs resource-rs
kubectl delete pod manual-pod
```

---

# Final Mental Model

ReplicaSet = Pod Manager
It ensures:

Desired replicas == Actual running pods

It constantly reconciles state.

Scheduler schedules pods.
ReplicaSet ensures correct count.
API server stores desired state.

---

If you now understand:

* Selector importance
* Self-healing
* Scaling
* Controller behavior

Then ReplicaSets are fully clear.

Next, I strongly recommend learning:

👉 Deployments (because they control ReplicaSets internally)