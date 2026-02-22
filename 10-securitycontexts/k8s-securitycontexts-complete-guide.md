Vidhu 🔥 I think you meant **SecurityContexts** (not “secretcontexts”) — and this is a very important production topic.

SecurityContext controls:

* User permissions
* Privilege escalation
* Filesystem behavior
* Linux capabilities
* Container isolation

This is where Kubernetes security really starts.

Save this as:

```bash
k8s-securitycontexts-complete-guide.md
```

---

# Kubernetes SecurityContext – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* Pod-level vs Container-level SecurityContext
* runAsUser
* runAsNonRoot
* fsGroup
* allowPrivilegeEscalation
* readOnlyRootFilesystem
* privileged containers
* Capabilities
* Real-world production patterns

---

# 2️⃣ What is SecurityContext?

SecurityContext defines:

> How a Pod or container should run from a security perspective.

Without it:

* Containers often run as root ❌
* Can escalate privileges ❌
* Can write anywhere ❌

SecurityContext reduces attack surface.

---

# 3️⃣ Pod-Level vs Container-Level

| Pod-Level                 | Container-Level          |
| ------------------------- | ------------------------ |
| Applies to all containers | Specific container       |
| runAsUser                 | privileged               |
| fsGroup                   | capabilities             |
| seccompProfile            | allowPrivilegeEscalation |

---

# 4️⃣ Basic SecurityContext Example

Create file: `pod-sec-basic.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsNonRoot: true
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f pod-sec-basic.yaml
```

Verify:

```bash
kubectl exec -it secure-pod -- id
```

Expected output:

```
uid=1000
```

✔ Container running as non-root.

---

# 5️⃣ runAsNonRoot

If image tries to run as root and:

```yaml
runAsNonRoot: true
```

Pod will fail.

Test it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fail-pod
spec:
  securityContext:
    runAsNonRoot: true
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
```

If image runs as root → pod may fail.

Check:

```bash
kubectl describe pod fail-pod
```

---

# 6️⃣ fsGroup (Very Important for Volumes)

Used when mounting volumes.

Create file: `pod-fsgroup.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fsgroup-pod
spec:
  securityContext:
    fsGroup: 2000
  volumes:
  - name: data
    emptyDir: {}
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: data
      mountPath: /data
```

Check inside pod:

```bash
kubectl exec -it fsgroup-pod -- sh
ls -ld /data
```

✔ Group ownership applied.

Used heavily with PersistentVolumes.

---

# 7️⃣ Container-Level Security Settings

Create file: `pod-sec-advanced.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: advanced-sec-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
```

Apply and verify:

```bash
kubectl describe pod advanced-sec-pod
```

---

# 8️⃣ allowPrivilegeEscalation

Prevents container from:

* Using sudo
* setuid binaries
* Gaining higher privileges

Production best practice:

```yaml
allowPrivilegeEscalation: false
```

---

# 9️⃣ readOnlyRootFilesystem

Makes root filesystem read-only.

Prevents:

* Malware writing files
* Unauthorized changes

If app needs write access → mount volume.

---

# 🔟 Capabilities

Linux splits root privileges into smaller pieces.

Example:

```yaml
capabilities:
  drop:
    - ALL
  add:
    - NET_BIND_SERVICE
```

This:

* Drops everything
* Only allows binding to ports < 1024

Very secure pattern.

---

# 1️⃣1️⃣ Privileged Container (Dangerous)

```yaml
securityContext:
  privileged: true
```

This gives:

* Almost full node access
* Access to host devices

Used only for:

* CNI plugins
* System agents
* Low-level monitoring tools

Never for application containers.

---

# 1️⃣2️⃣ seccompProfile

Restricts Linux syscalls.

```yaml
securityContext:
  seccompProfile:
    type: RuntimeDefault
```

Helps protect against kernel exploits.

---

# 1️⃣3️⃣ Production Hardened Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hardened-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: nginx
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
```

This is close to real enterprise setup.

---

# 1️⃣4️⃣ Debug Commands

Check effective user:

```bash
kubectl exec -it <pod> -- id
```

Check permissions:

```bash
kubectl exec -it <pod> -- ls -l
```

Check events:

```bash
kubectl describe pod <pod>
```

---

# 1️⃣5️⃣ SecurityContext vs Secret (Important)

SecurityContext:
→ Controls how container runs

Secret:
→ Stores sensitive data

They are completely different concepts.

---

# Final Mental Model

Container by default = root + full privileges ❌

SecurityContext lets you:

* Remove root
* Remove privilege escalation
* Remove unnecessary capabilities
* Restrict filesystem
* Harden workload

SecurityContext = Runtime Security
Secret = Sensitive Data Storage

---

Vidhu, if you understand this:

You’re now touching production-grade Kubernetes security.

Next powerful topics:

* Pod Security Admission (Restricted / Baseline)
* NetworkPolicies
* Ingress
* StatefulSets
* Full real-world architecture design

Tell me where you want to go next 🔥
