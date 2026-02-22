```bash
k8s-services-complete-guide.md
```

---

# Kubernetes Services – Complete Practical Guide

---

# 1️⃣ Objective

Understand and verify:

* Why Services are needed
* ClusterIP
* NodePort
* LoadBalancer
* Headless Service
* How Service selects Pods
* How kube-proxy works (basic idea)
* How to test everything practically

---

# 2️⃣ Why Do We Need Services?

Problem:

Pods are:

* Ephemeral
* Dynamic IPs
* Can die anytime

If a Pod dies:

* Its IP changes
* Other Pods can’t track it

Solution:

Service gives:
✔ Stable IP
✔ Stable DNS name
✔ Load balancing

---

# 3️⃣ Architecture Flow

```text
Client
   ↓
Service (Stable IP)
   ↓
kube-proxy
   ↓
One of the matching Pods
```

Service uses **labels** to find Pods.

---

# 4️⃣ Create Deployment First (Backend)

Create file: `deploy-nginx.yaml`

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
        image: nginx
        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f deploy-nginx.yaml
kubectl get pods
```

---

# 5️⃣ ClusterIP Service (Default)

Create file: `svc-clusterip.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

Apply:

```bash
kubectl apply -f svc-clusterip.yaml
```

Verify:

```bash
kubectl get svc
```

You’ll see:

```text
TYPE        CLUSTER-IP
ClusterIP   10.x.x.x
```

---

# 6️⃣ Test ClusterIP (Internal Access)

Exec into any pod:

```bash
kubectl exec -it <pod-name> -- sh
```

Install curl if needed:

```bash
apk add curl
```

Access service:

```bash
curl nginx-service
```

✔ DNS works
✔ Load balancing works

Try multiple times → different pods respond.

---

# 7️⃣ Understand Port vs TargetPort

```yaml
ports:
- port: 80
  targetPort: 80
```

* port → Service port
* targetPort → Container port

Service forwards traffic from port → targetPort.

---

# 8️⃣ NodePort Service

Allows external access via Node IP.

Create file: `svc-nodeport.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007
```

Apply:

```bash
kubectl apply -f svc-nodeport.yaml
kubectl get svc
```

Access:

```text
http://<NodeIP>:30007
```

✔ External access works.

NodePort range: 30000–32767

---

# 9️⃣ LoadBalancer Service

Used in cloud environments (AWS, Azure, GCP).

Create file: `svc-loadbalancer.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

Apply:

```bash
kubectl apply -f svc-loadbalancer.yaml
kubectl get svc
```

Cloud provider creates external IP.

✔ Production standard way.

---

# 🔟 Headless Service

Used for StatefulSets.

Create file: `svc-headless.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - port: 80
```

Apply:

```bash
kubectl apply -f svc-headless.yaml
```

This:

* Does NOT create cluster IP
* Returns pod IPs directly via DNS

---

# 1️⃣1️⃣ Verify Service Endpoints

Check endpoints:

```bash
kubectl get endpoints nginx-service
```

Shows:

```text
IP1:80
IP2:80
IP3:80
```

These are pod IPs.

Service forwards traffic to these endpoints.

---

# 1️⃣2️⃣ Test Load Balancing

Run:

```bash
kubectl exec -it <any-pod> -- curl nginx-service
```

Repeat multiple times.

You’ll see responses from different pods.

✔ kube-proxy handles round-robin.

---

# 1️⃣3️⃣ Service Without Matching Pods

Delete deployment:

```bash
kubectl delete deployment nginx-deploy
```

Check service:

```bash
kubectl get endpoints nginx-service
```

Endpoints will be empty.

Service exists, but no pods backing it.

---

# 1️⃣4️⃣ DNS Format

Inside cluster:

```text
service-name.namespace.svc.cluster.local
```

Example:

```text
nginx-service.default.svc.cluster.local
```

Usually just `nginx-service` works inside same namespace.

---

# 1️⃣5️⃣ Debug Commands

Check services:

```bash
kubectl get svc
```

Check endpoints:

```bash
kubectl get endpoints
```

Describe service:

```bash
kubectl describe svc nginx-service
```

Check selector:

```bash
kubectl get pods --show-labels
```

---

# 1️⃣6️⃣ Service Types Summary

| Type         | Use Case               |
| ------------ | ---------------------- |
| ClusterIP    | Internal communication |
| NodePort     | External via node IP   |
| LoadBalancer | Cloud production       |
| Headless     | Stateful workloads     |

---

# 1️⃣7️⃣ Clean Up

```bash
kubectl delete svc nginx-service
kubectl delete svc nginx-nodeport
kubectl delete svc nginx-lb
kubectl delete svc nginx-headless
kubectl delete deployment nginx-deploy
```

---

# Final Mental Model

Pods = Dynamic workers
Service = Stable front desk
kube-proxy = Traffic manager

Client talks to Service.
Service routes to Pods.

Service never dies when pods restart.

---

If this is clear now, your networking fundamentals are forming nicely.

Next logical steps:

* Ingress (HTTP routing)
* ConfigMaps & Secrets
* NetworkPolicies
* StatefulSets
* Or Full production flow (App → Service → Ingress → LoadBalancer)