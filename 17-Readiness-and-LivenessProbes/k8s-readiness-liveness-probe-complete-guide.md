---
k8s-readiness-liveness-probe-complete-guide.md
---

# Kubernetes Readiness & Liveness Probes — Practical Guide

---

# 1️⃣ Objective

Understand and verify:

- What readiness and liveness probes are
- Differences and when to use each
- How to configure probes in Pod specs
- How to test probes in-cluster
- Best practices and common pitfalls

---

# 2️⃣ Quick Overview

- Liveness probe: determines whether the container is alive. If it fails, kubelet restarts the container.
- Readiness probe: determines whether the container is ready to accept traffic. If it fails, the Pod is removed from Service endpoints until it succeeds.

---

# 3️⃣ Probe Types

All probes support these handlers:

- `httpGet` — perform an HTTP GET against the container
- `tcpSocket` — open a TCP connection to the container
- `exec` — run a command inside the container

Common timing fields:

- `initialDelaySeconds` — wait before starting probes
- `periodSeconds` — probe frequency
- `timeoutSeconds` — probe timeout
- `failureThreshold` — consecutive failures before marking unhealthy
- `successThreshold` — consecutive successes to mark healthy

---

# 4️⃣ Liveness Probe Example

Add to your container spec:

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 3
```

Behavior: if the `/healthz` endpoint returns non-2xx for 3 consecutive checks, kubelet restarts the container.

---

# 5️⃣ Readiness Probe Example

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 2
```

Behavior: if `/ready` fails, the Pod is removed from `Endpoints` and stops receiving Service traffic until it becomes healthy again.

---

# 6️⃣ Combined Minimal Pod Example

Create or edit a pod file (example shows both probes):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: demo-app
    image: hashicorp/http-echo:0.2.3
    args:
      - "-text=ok"
    ports:
      - containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 2
      periodSeconds: 5
      failureThreshold: 2
```

Apply:

```bash
kubectl apply -f k8s-readiness-liveness-pod.yaml
```

Verify status:

```bash
kubectl get pods
kubectl describe pod probe-demo
```

---

# 7️⃣ Testing & Troubleshooting

- View probe results in `kubectl describe pod <pod>` under `Events` and container `State`/`Last Probe` information.
- Use `kubectl logs` to inspect container logs for probe-related errors.
- To simulate a failing readiness probe: make the `/ready` handler return non-200 or add a sleep in the command.
- To simulate a failing liveness probe: have the process enter a deadlock or exit non-zero; kubelet should restart the container.

Examples:

```bash
# watch pod status
kubectl get pods -w

# describe to see probe events
kubectl describe pod probe-demo
```

---

# 8️⃣ Best Practices

- Use readiness for traffic control and liveness for crash recovery.
- Set sensible `initialDelaySeconds` for slow-starting apps.
- Prefer low `timeoutSeconds` and short `periodSeconds` for fast detection, balanced with tolerance for transient failures.
- Avoid overly aggressive `failureThreshold` that causes flapping.
- Use `exec` probes for internal checks that are not HTTP/TCP-accessible.

---

# 9️⃣ References & Further Reading

- Kubernetes probes docs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
- `kubectl` debugging: https://kubernetes.io/docs/tasks/debug-application-cluster/debug-pod-replication-controller/

---

# 10️⃣ Local files

- Example pod manifest in this folder: `pod-definition.yaml` — use it to add probes and test behavior.
