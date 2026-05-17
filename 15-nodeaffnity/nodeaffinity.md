Vidhu 🔥 perfect timing.

Now we’re moving from **basic scheduling (nodeSelector)** to **advanced scheduling (NodeAffinity)** — and this is 100% CKAD exam material.

I’ll first explain clearly.
Then I’ll give you a **clean CKAD-ready markdown lab file**.

---

# 🧠 PART 1 — Conceptual Understanding

## 1️⃣ Why NodeAffinity?

You already learned:

* nodeSelector → simple key=value match

But what if you need:

* OR conditions
* NOT conditions
* Multiple values
* Soft preference instead of strict rule

nodeSelector cannot do that.

NodeAffinity solves it.

---

# 2️⃣ What is NodeAffinity?

NodeAffinity is a more expressive way to:

> Schedule pods on nodes based on labels.

Defined inside Pod spec:

```yaml
affinity:
  nodeAffinity:
```

---

# 3️⃣ Two Types (Very Important)

## 🔹 requiredDuringSchedulingIgnoredDuringExecution

Hard rule.

If no match → Pod stays Pending.

---

## 🔹 preferredDuringSchedulingIgnoredDuringExecution

Soft rule.

Scheduler tries to match, but if not possible → schedules anyway.

---

# 4️⃣ NodeAffinity Flow

```text
Pod created
       ↓
Scheduler checks required rules
       ↓
If matched → schedule
If not → Pending

Then:
Scheduler tries preferred rules
       ↓
Chooses best node
```

---

# 5️⃣ Operators You Must Know (CKAD Critical)

| Operator     | Meaning               |
| ------------ | --------------------- |
| In           | Value must match list |
| NotIn        | Value must NOT match  |
| Exists       | Key must exist        |
| DoesNotExist | Key must not exist    |
| Gt           | Greater than          |
| Lt           | Less than             |

Memorize these.

---

# 6️⃣ NodeSelector vs NodeAffinity

| NodeSelector     | NodeAffinity  |
| ---------------- | ------------- |
| Simple key=value | Expressions   |
| Only AND logic   | AND + OR      |
| No soft rule     | Has preferred |
| Basic            | Advanced      |

---

Now let’s build your CKAD-ready documentation.

---

# 📘 Kubernetes NodeAffinity – CKAD Guide

Save this as:

```bash
k8s-nodeaffinity-ckad-guide.md
```

---

# Kubernetes NodeAffinity – Complete CKAD Guide

---

# 1️⃣ Exam-Relevant Concepts

You must know:

* requiredDuringSchedulingIgnoredDuringExecution
* preferredDuringSchedulingIgnoredDuringExecution
* matchExpressions
* Operators (In, NotIn, Exists)
* Troubleshooting Pending pods

---

# 2️⃣ Label a Node

```bash
kubectl get nodes
```

Add labels:

```bash
kubectl label nodes <node-name> disktype=ssd
kubectl label nodes <node-name> environment=prod
```

Verify:

```bash
kubectl get nodes --show-labels
```

---

# 3️⃣ Hard Requirement Example

Create file: `pod-nodeaffinity-required.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: required-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f pod-nodeaffinity-required.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

If no node matches → Pod stays Pending.

---

# 4️⃣ Soft Requirement Example

Create file: `pod-nodeaffinity-preferred.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: preferred-affinity-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: environment
            operator: In
            values:
            - prod
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f pod-nodeaffinity-preferred.yaml
```

Pod will prefer prod nodes, but won’t fail if unavailable.

---

# 5️⃣ Multiple Conditions (AND Logic)

```yaml
matchExpressions:
- key: disktype
  operator: In
  values:
  - ssd
- key: environment
  operator: In
  values:
  - prod
```

Both must match.

---

# 6️⃣ OR Logic

Use multiple nodeSelectorTerms:

```yaml
nodeSelectorTerms:
- matchExpressions:
  - key: disktype
    operator: In
    values:
    - ssd
- matchExpressions:
  - key: environment
    operator: In
    values:
    - dev
```

Means:

(disktype=ssd) OR (environment=dev)

Very important exam trick.

---

# 7️⃣ Exists Operator Example

```yaml
- key: gpu
  operator: Exists
```

If node has label "gpu", pod schedules.

---

# 8️⃣ Troubleshooting Pending Pods

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```
node(s) didn't match node affinity
```

Check labels:

```bash
kubectl get nodes --show-labels
```

---

# 9️⃣ Remove Node Label

```bash
kubectl label nodes <node-name> disktype-
```

---

# 🔟 Quick CKAD Snippet (Memorize)

Hard rule template:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: key-name
          operator: In
          values:
          - value-name
```

---

# 1️⃣1️⃣ Clean Up

```bash
kubectl delete pod required-affinity-pod
kubectl delete pod preferred-affinity-pod
```

---

# Final Mental Model

nodeSelector = Basic filter
NodeAffinity = Advanced scheduling rules

required = Hard
preferred = Soft

Scheduler:
1️⃣ Check required
2️⃣ Score preferred
3️⃣ Schedule best node

---

Vidhu 🔥 now you officially understand advanced scheduling.

Next logical CKAD topics:

* Pod Affinity & Anti-Affinity
* NetworkPolicies
* Init Containers
* Exam-style rapid practice
* Or full scheduling comparison sheet

Tell me what you want next 👊
