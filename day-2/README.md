# Day 2 – Kubernetes Namespaces and Deployments

---

## 🚀 Overview

Today I moved from standalone Pods to **real production-grade Kubernetes workloads using Deployments** and learned how to organize resources using **Namespaces**.



# 🧠 Task 1: Default Namespaces

## Command

```
kubectl get namespaces
```

## Default Namespaces Observed

* **default** → Default workspace
* **kube-system** → Kubernetes internal components
* **kube-public** → Public resources
* **kube-node-lease** → Node heartbeat tracking

---

## Inspect kube-system

```
kubectl get pods -n kube-system
```

### Observation

* Multiple pods running (API server, scheduler, etcd, controller manager, DNS, proxy)

👉 These are critical system components — should NOT be modified.

---

# 🧩 Task 2: Custom Namespaces

## Create Namespaces

```
kubectl create namespace dev
kubectl create namespace staging
```

## Create via YAML

```
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

```
kubectl apply -f namespace.yaml
```

---

## Run Pods in Namespaces

```
kubectl run nginx-dev --image=nginx:latest -n dev
kubectl run nginx-staging --image=nginx:latest -n staging
```

---

## Verify

```
kubectl get pods
kubectl get pods -A
```
<img width="1021" height="312" alt="image" src="https://github.com/user-attachments/assets/8cf735b8-bad6-4ff7-92eb-23c983b5d5e4" />

### Insight

* `kubectl get pods` → only default namespace
* `kubectl get pods -A` → all namespaces

---

# 🏗️ Task 3: First Deployment

## Manifest: `nginx-deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
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
        image: nginx:1.24
        ports:
        - containerPort: 80
```

---

## Apply

```
kubectl apply -f nginx-deployment.yaml
```

## Verify

```
kubectl get deployments -n dev
kubectl get pods -n dev
```
<img width="670" height="297" alt="image" src="https://github.com/user-attachments/assets/029becc8-a19a-4275-aebe-6385a5737758" />

---

## Deployment Columns Meaning

* **READY** → Pods ready / desired
* **UP-TO-DATE** → Pods updated to latest spec
* **AVAILABLE** → Pods available to serve traffic

---

# 🔁 Task 4: Self-Healing

## Delete a Pod

```
kubectl delete pod <pod-name> -n dev
kubectl get pods -n dev
```
<img width="815" height="277" alt="image" src="https://github.com/user-attachments/assets/f4ed6751-666a-4088-8484-2951eb67b460" />

### Observation

* New pod created automatically
* Replacement pod has a **different name**

👉 Deployment ensures desired replicas are maintained

---

# 📈 Task 5: Scaling

## Scale Up

```
kubectl scale deployment nginx-deployment --replicas=5 -n dev
kubectl get pods -n dev
```

## Scale Down

```
kubectl scale deployment nginx-deployment --replicas=2 -n dev
kubectl get pods -n dev
```

---

### Observation

* Scaling up → new pods created
* Scaling down → extra pods terminated

---

# 🔄 Task 6: Rolling Update

## Update Image

```
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
```

## Monitor Rollout

```
kubectl rollout status deployment/nginx-deployment -n dev
```

---

## Rollout History

```
kubectl rollout history deployment/nginx-deployment -n dev
```

---

## Rollback

```
kubectl rollout undo deployment/nginx-deployment -n dev
kubectl rollout status deployment/nginx-deployment -n dev
```

---

## Verify Image

```
kubectl describe deployment nginx-deployment -n dev | grep Image
```

### Result

* Rolled back to **nginx:1.24**

---

# 🧹 Task 7: Cleanup

```
kubectl delete deployment nginx-deployment -n dev
kubectl delete pod nginx-dev -n dev
kubectl delete pod nginx-staging -n staging
kubectl delete namespace dev staging production
```

---

## Verification

```
kubectl get namespaces
kubectl get pods -A
```

### Result

✔️ All resources successfully removed

---

# 📌 Key Takeaways

* Namespaces provide **logical isolation** within a cluster
* Deployments manage Pods and ensure **self-healing**
* Scaling adjusts system capacity dynamically
* Rolling updates ensure **zero downtime deployments**
* Standalone Pods are not suitable for production

---

# 🧩 Conclusion

Today marked a major shift:

Pods → Deployments → Real-world application management

Kubernetes is not just running containers —
it is **continuously managing system state at scale**.
