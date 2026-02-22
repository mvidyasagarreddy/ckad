```bash
k8s-commands-arguments-complete-guide.md
```

---

# Kubernetes Commands & Arguments – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* What are `command` and `args`
* Difference between Docker ENTRYPOINT and CMD
* How Kubernetes overrides container defaults
* Exec form vs Shell form
* Practical testing steps
* Common mistakes

---

# 2️⃣ Basic Concept

In Kubernetes:

```yaml
command:  → overrides Docker ENTRYPOINT
args:     → overrides Docker CMD
```

Important mapping:

| Dockerfile | Kubernetes |
| ---------- | ---------- |
| ENTRYPOINT | command    |
| CMD        | args       |

---

# 3️⃣ Example Dockerfile Understanding

Suppose Dockerfile has:

```dockerfile
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

If you don’t specify anything in Pod:

It runs:

```bash
nginx -g "daemon off;"
```

---

# 4️⃣ Basic Pod Without command/args

Create file: `pod-default.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: default-cmd
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f pod-default.yaml
kubectl get pods
```

Container runs using default ENTRYPOINT + CMD.

---

# 5️⃣ Override Only Arguments

Create file: `pod-args.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: args-demo
spec:
  containers:
  - name: nginx
    image: nginx
    args: ["-g", "daemon off;"]
```

This:

* Keeps ENTRYPOINT from image
* Overrides CMD

---

# 6️⃣ Override Command Completely

Create file: `pod-command.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
spec:
  containers:
  - name: test
    image: busybox
    command: ["sleep"]
    args: ["3600"]
```

Apply:

```bash
kubectl apply -f pod-command.yaml
kubectl get pods
```

This runs:

```bash
sleep 3600
```

---

# 7️⃣ Exec Form vs Shell Form

## ✅ Exec Form (Recommended)

```yaml
command: ["sleep", "3600"]
```

Better:

* No shell
* No signal issues
* Production recommended

---

## ⚠ Shell Form

```yaml
command: ["sh", "-c", "sleep 3600"]
```

Used when:

* You need pipes
* Environment variables
* Multiple commands

Example:

```yaml
command: ["sh", "-c"]
args:
  - |
    echo "Starting..."
    sleep 3600
```

---

# 8️⃣ Practical Testing

Create file: `pod-multi-command.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-command
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c"]
    args:
      - |
        echo "Pod started"
        sleep 3600
```

Apply:

```bash
kubectl apply -f pod-multi-command.yaml
```

Check logs:

```bash
kubectl logs multi-command
```

You’ll see:

```
Pod started
```

---

# 9️⃣ Verify Running Command Inside Pod

Exec inside pod:

```bash
kubectl exec -it multi-command -- sh
```

Check process:

```bash
ps
```

You’ll see running command.

---

# 🔟 What Happens If You Misconfigure?

Example:

```yaml
command: ["sleep 3600"]
```

This is wrong.

It treats entire string as single binary.

Pod will crash with:

```
executable file not found
```

Correct way:

```yaml
command: ["sleep", "3600"]
```

---

# 1️⃣1️⃣ Environment Variable Usage

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: test
    image: busybox
    env:
    - name: MESSAGE
      value: "Hello Vidhu"
    command: ["sh", "-c"]
    args:
      - echo $MESSAGE && sleep 3600
```

Apply and check logs:

```bash
kubectl logs env-demo
```

---

# 1️⃣2️⃣ Common Real-World Use Cases

| Use Case                | Example                |
| ----------------------- | ---------------------- |
| Debug pod               | sleep 3600             |
| One-time script         | sh -c script.sh        |
| Init container setup    | sh -c setup commands   |
| Override image behavior | custom startup command |

---

# 1️⃣3️⃣ Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cmd-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cmd-demo
  template:
    metadata:
      labels:
        app: cmd-demo
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sleep"]
        args: ["3600"]
```

Apply normally.

---

# 1️⃣4️⃣ Important Debug Commands

Check logs:

```bash
kubectl logs <pod>
```

Check container crash reason:

```bash
kubectl describe pod <pod>
```

Check restart count:

```bash
kubectl get pods
```

---

# 1️⃣5️⃣ Mental Model

Container Image = Default behavior
Kubernetes command = Override ENTRYPOINT
Kubernetes args = Override CMD

If command provided → ENTRYPOINT replaced
If only args provided → CMD replaced

---

# 1️⃣6️⃣ Clean Up

```bash
kubectl delete pod default-cmd
kubectl delete pod args-demo
kubectl delete pod command-demo
kubectl delete pod multi-command
kubectl delete pod env-demo
kubectl delete deployment cmd-deploy
```

---

# Final Mental Model

Docker decides default startup.

Kubernetes lets you override it cleanly.

Exec form preferred.
Shell form when needed.

---

If this is clear now, your container fundamentals are getting solid.

Next good topics:

* Init Containers
* ConfigMaps & Secrets
* Probes deep dive
* StatefulSets
* Or complete production architecture

Tell me what you want next 🔥
