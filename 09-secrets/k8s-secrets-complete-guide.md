Vidhu 🔥 this is an important one.
If ConfigMaps are for normal config, **Secrets are for sensitive data** — and in real projects, this matters a lot.

Below is your structured, revisitable documentation.

Save this as:

```bash
k8s-secrets-complete-guide.md
```

---

# Kubernetes Secrets – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* What is a Secret
* Types of Secrets
* How to create
* Use as environment variable
* Use as volume (file)
* Difference between ConfigMap & Secret
* Base64 concept
* Real-world usage patterns

---

# 2️⃣ What is a Secret?

A Secret is used to store:

✔ Passwords
✔ API tokens
✔ Database credentials
✔ TLS certificates
✔ SSH keys

Secrets are:

* Base64 encoded (not encrypted by default)
* Stored in etcd
* Meant for sensitive data

⚠ Important: Base64 ≠ Encryption

---

# 3️⃣ Types of Secrets

Run:

```bash
kubectl explain secret
```

Common types:

| Type                                | Use Case              |
| ----------------------------------- | --------------------- |
| Opaque                              | Generic secret        |
| kubernetes.io/dockerconfigjson      | Docker registry login |
| kubernetes.io/tls                   | TLS certs             |
| kubernetes.io/service-account-token | ServiceAccount token  |

Most common → **Opaque**

---

# 4️⃣ Create Secret (YAML Method)

First encode values:

Example:

```bash
echo -n "admin" | base64
echo -n "mypassword" | base64
```

Suppose output is:

```text
YWRtaW4=
bXlwYXNzd29yZA==
```

Create file: `secret-basic.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=
  password: bXlwYXNzd29yZA==
```

Apply:

```bash
kubectl apply -f secret-basic.yaml
```

Verify:

```bash
kubectl get secret
kubectl describe secret app-secret
```

---

# 5️⃣ Create Secret (CLI Method – Easier)

```bash
kubectl create secret generic app-secret-cli \
  --from-literal=username=admin \
  --from-literal=password=mypassword
```

Kubernetes encodes automatically.

Verify:

```bash
kubectl get secret app-secret-cli -o yaml
```

---

# 6️⃣ Use Secret as Environment Variable

Create file: `pod-secret-env.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c"]
    args:
      - echo $USERNAME && echo $PASSWORD && sleep 3600
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
```

Apply:

```bash
kubectl apply -f pod-secret-env.yaml
```

Check logs:

```bash
kubectl logs secret-env-pod
```

Expected:

```text
admin
mypassword
```

---

# 7️⃣ Use Secret as Volume (File-Based)

Create file: `pod-secret-volume.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret
      readOnly: true
```

Apply:

```bash
kubectl apply -f pod-secret-volume.yaml
```

Verify inside pod:

```bash
kubectl exec -it secret-volume-pod -- sh
ls /etc/secret
cat /etc/secret/username
cat /etc/secret/password
```

✔ Each key becomes a file.

---

# 8️⃣ Secret Update Behavior

If Secret is mounted as:

* Environment variable → Pod restart required
* Volume → File updates automatically (eventually)

Edit secret:

```bash
kubectl edit secret app-secret
```

Change value → check file again.

---

# 9️⃣ Decode Secret Value

Check encoded value:

```bash
kubectl get secret app-secret -o yaml
```

Decode:

```bash
echo YWRtaW4= | base64 -d
```

---

# 🔟 Docker Registry Secret Example

For private image pull:

```bash
kubectl create secret docker-registry my-registry-secret \
  --docker-username=myuser \
  --docker-password=mypass \
  --docker-email=myemail@example.com
```

Use in pod:

```yaml
imagePullSecrets:
- name: my-registry-secret
```

---

# 1️⃣1️⃣ Secret vs ConfigMap

| ConfigMap      | Secret         |
| -------------- | -------------- |
| Non-sensitive  | Sensitive      |
| Plain text     | Base64 encoded |
| App config     | Credentials    |
| Stored in etcd | Stored in etcd |

In production:
Secrets should be encrypted at rest.

---

# 1️⃣2️⃣ Real Production Practices

✔ Use external secret manager (Vault, AWS Secrets Manager)
✔ Enable etcd encryption
✔ Restrict RBAC access
✔ Mount as files instead of env if possible
✔ Never commit secrets to Git

---

# 1️⃣3️⃣ Debug Commands

Check secrets:

```bash
kubectl get secret
```

Describe:

```bash
kubectl describe secret app-secret
```

Check pod env:

```bash
kubectl exec -it secret-env-pod -- env
```

Check mounted files:

```bash
kubectl exec -it secret-volume-pod -- ls /etc/secret
```

---

# 1️⃣4️⃣ Clean Up

```bash
kubectl delete pod secret-env-pod
kubectl delete pod secret-volume-pod
kubectl delete secret app-secret
kubectl delete secret app-secret-cli
```

---

# Final Mental Model

Image = Code
ConfigMap = Configuration
Secret = Sensitive configuration

Never hardcode credentials inside image.

---

If this is clear, your Kubernetes fundamentals are now solid.

Next logical topics:

* Probes deep dive
* Ingress
* StatefulSets
* NetworkPolicies
* Full real-world architecture flow

Tell me what you want next 🔥
