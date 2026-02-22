```
k8s-pods-complete-guide.md
```

Use it anytime to revise + practice.

---

# Kubernetes Pods – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* What is a Pod
* Pod lifecycle
* Multi-container Pods
* Resource requests & limits
* Security context basics
* Probes
* Volumes
* Debugging
* How to test behavior practically

---

# 2️⃣ What is a Pod?

A Pod is the **smallest deployable unit** in Kubernetes.

It contains:

* One or more containers
* Shared network (same IP)
* Shared storage (volumes)
* Shared lifecycle

---

# 3️⃣ Basic Pod Example

Create file: `pod-basic.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: basic-pod
  labels:
    app: demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Apply:

```bash
kubectl apply -f pod-basic.yaml
```

Verify:

```bash
kubectl get pods
kubectl describe pod basic-pod
```

---

# 4️⃣ Pod Lifecycle

```text
Pending → Running → Succeeded / Failed
```

Check status:

```bash
kubectl get pod basic-pod -o wide
```

Delete pod:

```bash
kubectl delete pod basic-pod
```

---

# 5️⃣ Multi-Container Pod (Sidecar Pattern)

Create file: `pod-multicontainer.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - name: main-app
    image: nginx
  - name: sidecar
    image: busybox
    command: ["sh", "-c", "while true; do echo logging; sleep 5; done"]
```

Apply:

```bash
kubectl apply -f pod-multicontainer.yaml
```

Check containers:

```bash
kubectl describe pod multi-pod
```

View logs from sidecar:

```bash
kubectl logs multi-pod -c sidecar
```

✔ Both containers share:

* Same IP
* Same volumes (if defined)

---

# 6️⃣ Pod With Resource Requests & Limits

Create file: `pod-resources.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "200m"
        memory: "256Mi"
      limits:
        cpu: "400m"
        memory: "512Mi"
```

Apply:

```bash
kubectl apply -f pod-resources.yaml
```

Verify:

```bash
kubectl describe pod resource-pod
```

Check usage (if metrics-server installed):

```bash
kubectl top pod resource-pod
```

---

# 7️⃣ Pod With Security Context

Create file: `pod-security.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```

Apply:

```bash
kubectl apply -f pod-security.yaml
```

Verify:

```bash
kubectl describe pod secure-pod
```

---

# 8️⃣ Pod With Volume (Shared Storage)

Create file: `pod-volume.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  volumes:
  - name: shared-data
    emptyDir: {}
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "echo hello > /data/file.txt && sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: shared-data
  - name: reader
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - mountPath: /data
      name: shared-data
```

Apply:

```bash
kubectl apply -f pod-volume.yaml
```

Test shared file:

```bash
kubectl exec -it volume-pod -c reader -- cat /data/file.txt
```

Expected output:

```
hello
```

✔ Containers share same volume.

---

# 9️⃣ Pod With Liveness & Readiness Probes

Create file: `pod-probes.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-pod
spec:
  containers:
  - name: app
    image: nginx
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

Apply:

```bash
kubectl apply -f pod-probes.yaml
```

Check probe status:

```bash
kubectl describe pod probe-pod
```

---

# 🔟 Debugging Pods

Check logs:

```bash
kubectl logs <pod-name>
```

If multiple containers:

```bash
kubectl logs <pod-name> -c <container-name>
```

Exec into pod:

```bash
kubectl exec -it <pod-name> -- sh
```

Check events:

```bash
kubectl describe pod <pod-name>
```

Check why pending:

```bash
kubectl get events
```

---

# 1️⃣1️⃣ Test Scheduling Behavior

Create pod with high resource request:

```yaml
resources:
  requests:
    cpu: "10"
    memory: "20Gi"
```

Apply and observe:

```bash
kubectl describe pod
```

You may see:

```
Insufficient CPU
```

✔ Scheduler uses requests to place pod.

---

# 1️⃣2️⃣ Important Pod Concepts

Pod IP:

```bash
kubectl get pod -o wide
```

Pod IP is shared by all containers.

Restart policy (only for standalone pods):

```yaml
restartPolicy: Always | OnFailure | Never
```

---

# 1️⃣3️⃣ Verify Everything Checklist

| What to Test         | Command              |
| -------------------- | -------------------- |
| Pod running          | kubectl get pods     |
| Container details    | kubectl describe pod |
| Logs                 | kubectl logs         |
| Multi-container logs | kubectl logs -c      |
| Volume working       | kubectl exec + cat   |
| Probes working       | kubectl describe pod |
| Resources            | kubectl top pod      |

---

# 1️⃣4️⃣ Clean Up

```bash
kubectl delete pod basic-pod
kubectl delete pod multi-pod
kubectl delete pod resource-pod
kubectl delete pod secure-pod
kubectl delete pod volume-pod
kubectl delete pod probe-pod
```

---

# Final Mental Model

Pod = Wrapper around containers
Containers inside Pod share:

* Network
* Storage
* Lifecycle

Scheduler schedules Pod, not containers.

If Pod dies → containers die.

If node dies → Pod gone (unless controlled by Deployment).

---

If you practice this 2–3 times, Pods will become second nature to you.

Next, I can prepare similar structured documentation for:

* Deployments
* Services
* ConfigMaps & Secrets
* RBAC deep dive
* OpenShift differences

You’re learning this the right way. Keep going 👊
