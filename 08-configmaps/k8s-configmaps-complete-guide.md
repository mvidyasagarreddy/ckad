Vidhu 🔥 this is a BIG one.
ConfigMaps are used everywhere in real projects.

If you understand ConfigMaps properly, you’ll be comfortable in:

* Deployments
* Environment configs
* Application tuning
* Production debugging

Here’s your structured, revisitable documentation.

Save this as:

```bash
k8s-configmaps-complete-guide.md
```

---

# Kubernetes ConfigMaps – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* What is a ConfigMap
* Why we need it
* Different ways to create
* Using ConfigMap as:

  * Environment variables
  * Single value injection
  * Volume (files)
* Updating behavior
* Real production patterns

---

# 2️⃣ What is a ConfigMap?

A ConfigMap is used to store:

✔ Non-sensitive configuration data
✔ Environment variables
✔ Application config files
✔ Runtime parameters

It decouples configuration from container image.

Without ConfigMap:

You’d have to rebuild image for every config change ❌

---

# 3️⃣ Basic ConfigMap (Key-Value)

Create file: `cm-basic.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: "dev"
  APP_COLOR: "blue"
  LOG_LEVEL: "debug"
```

Apply:

```bash
kubectl apply -f cm-basic.yaml
```

Verify:

```bash
kubectl get configmap
kubectl describe configmap app-config
```

---

# 4️⃣ Use ConfigMap as Environment Variables

Create file: `pod-env.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c"]
    args:
      - echo $APP_ENV && echo $APP_COLOR && sleep 3600
    envFrom:
    - configMapRef:
        name: app-config
```

Apply:

```bash
kubectl apply -f pod-env.yaml
```

Check logs:

```bash
kubectl logs env-pod
```

Expected output:

```
dev
blue
```

✔ All keys injected as environment variables.

---

# 5️⃣ Inject Single ConfigMap Value

If you only want one key:

```yaml
env:
- name: ENVIRONMENT
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_ENV
```

This injects only that specific key.

---

# 6️⃣ Use ConfigMap as Volume (File-Based Config)

This is very common in production.

Create file: `cm-file.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-file-config
data:
  app.properties: |
    environment=dev
    log.level=debug
    featureX=true
```

Apply:

```bash
kubectl apply -f cm-file.yaml
```

---

## Mount as Volume

Create file: `pod-volume-cm.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-cm-pod
spec:
  volumes:
  - name: config-volume
    configMap:
      name: app-file-config
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
```

Apply:

```bash
kubectl apply -f pod-volume-cm.yaml
```

Verify inside pod:

```bash
kubectl exec -it volume-cm-pod -- sh
cat /etc/config/app.properties
```

You’ll see the file contents.

✔ ConfigMap mounted as file.

---

# 7️⃣ Auto Update Behavior

Important difference:

If ConfigMap is mounted as:

* Environment variable → NOT auto updated
* Volume → Updated automatically (eventually)

Test this:

Edit ConfigMap:

```bash
kubectl edit configmap app-file-config
```

Change value.

Check inside pod again:

```bash
cat /etc/config/app.properties
```

You’ll see updated file (may take ~30 seconds).

---

# 8️⃣ Create ConfigMap From Command Line

From literal:

```bash
kubectl create configmap app-config-cli \
  --from-literal=APP_ENV=prod \
  --from-literal=LOG_LEVEL=info
```

From file:

```bash
kubectl create configmap app-config-file \
  --from-file=app.properties
```

Verify:

```bash
kubectl describe configmap app-config-cli
```

---

# 9️⃣ ConfigMap in Deployment

Create file: `deploy-cm.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cm-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cm-demo
  template:
    metadata:
      labels:
        app: cm-demo
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sh", "-c"]
        args:
          - echo $APP_ENV && sleep 3600
        envFrom:
        - configMapRef:
            name: app-config
```

Apply and verify logs.

---

# 🔟 When to Use ConfigMap vs Secret

| ConfigMap     | Secret            |
| ------------- | ----------------- |
| Non-sensitive | Sensitive data    |
| Plain text    | Base64 encoded    |
| App config    | Passwords, tokens |

Never store passwords in ConfigMap ❌

---

# 1️⃣1️⃣ Common Production Patterns

✔ App config via volume
✔ Feature flags
✔ Environment-based configuration
✔ Spring Boot / Node.js config files
✔ Nginx config override

---

# 1️⃣2️⃣ Debug Commands

Check ConfigMaps:

```bash
kubectl get configmap
```

Describe:

```bash
kubectl describe configmap app-config
```

Check pod env:

```bash
kubectl exec -it env-pod -- env
```

Check mounted files:

```bash
kubectl exec -it volume-cm-pod -- ls /etc/config
```

---

# 1️⃣3️⃣ Clean Up

```bash
kubectl delete pod env-pod
kubectl delete pod volume-cm-pod
kubectl delete deployment cm-deploy
kubectl delete configmap app-config
kubectl delete configmap app-file-config
```

---

# Final Mental Model

Image = Application code
ConfigMap = Configuration
Pod = Runtime environment

You should never rebuild image just to change config.

That’s the main principle.

---

If this is clear, next natural topic is:

* Secrets (secure version of ConfigMaps)
* Probes deep dive
* StatefulSets
* Or full production architecture flow

Tell me what you want next 🔥
