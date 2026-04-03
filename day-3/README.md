# Day 3 – Kubernetes Services

---

## 🚀 Overview

Today I solved a critical problem in Kubernetes:

👉 **How do you reliably communicate with Pods when their IPs keep changing?**

The answer: **Services**

Services provide a **stable endpoint + load balancing** for Pods managed by Deployments.


---

# 🧠 Why Services?

### Problem

* Pods have **dynamic IPs**
* Pods are frequently recreated
* Multiple replicas → which Pod do you hit?

---

### Solution: Service

A Service provides:

* Stable IP
* DNS name
* Load balancing across Pods

```
Client → Service → Pods
```

---

# 🧩 Task 1: Deploy Application

## Manifest: `app-deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

---

## Apply

```
kubectl apply -f app-deployment.yaml
kubectl get pods -o wide
```
<img width="1075" height="106" alt="Screenshot 2026-04-02 220925" src="https://github.com/user-attachments/assets/51c5481a-09c0-4814-ba07-fb6ac215bf19" />

---

## Verification

✔️ 3 Pods running
✔️ Each Pod has a different IP

---

# 🧩 Task 2: ClusterIP Service

## Manifest: `clusterip-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

---

## Apply

```
kubectl apply -f clusterip-service.yaml
kubectl get services
```
<img width="740" height="117" alt="Screenshot 2026-04-02 221211" src="https://github.com/user-attachments/assets/127262d2-54b3-45d4-b5d5-99626e7a19d7" />

---

## Test (Inside Cluster)

```
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside container:

```
wget -qO- http://web-app-clusterip
```
<img width="1211" height="448" alt="image" src="https://github.com/user-attachments/assets/58a2f78e-684e-436a-b0cc-f1e76f6bd7e3" />

---

## Verification

✔️ Nginx page returned
✔️ Traffic load-balanced across Pods

---

# 🌐 Task 3: Service Discovery (DNS)

## Test DNS

```
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside container:

```
wget -qO- http://web-app-clusterip
wget -qO- http://web-app-clusterip.default.svc.cluster.local
nslookup web-app-clusterip
```

---

## Verification

✔️ DNS resolves to ClusterIP
✔️ Short name and full name both work

---

# 🌍 Task 4: NodePort Service

## Manifest: `nodeport-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
```

---

## Apply

```
kubectl apply -f nodeport-service.yaml
kubectl get services
```

---

## Access

```
curl http://localhost:30080
```

---

## Verification

✔️ External access works
✔️ Nginx page visible

---

# ☁️ Task 5: LoadBalancer Service

## Manifest: `loadbalancer-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

---

## Apply

```
kubectl apply -f loadbalancer-service.yaml
kubectl get services
```

---

## Observation

* EXTERNAL-IP shows **<pending>**

---

## Explanation

* Local clusters (Kind/Minikube) do NOT provision real load balancers
* Cloud providers (AWS/GCP/Azure) assign public IP

---

# 📊 Task 6: Service Comparison

```
kubectl get services -o wide
```

---

## Comparison

| Type         | Access             | Use Case           |
| ------------ | ------------------ | ------------------ |
| ClusterIP    | Internal           | Service-to-service |
| NodePort     | External via Node  | Dev/testing        |
| LoadBalancer | External via cloud | Production         |

---

## Key Insight

👉 LoadBalancer = ClusterIP + NodePort

---

## Verify

```
kubectl describe service web-app-loadbalancer
```
<img width="781" height="309" alt="image" src="https://github.com/user-attachments/assets/06b2f81d-1760-44df-90c2-2f795e578ddb" />

✔️ Contains:

* ClusterIP
* NodePort
* LoadBalancer config

---

# 🧹 Task 7: Cleanup

```
kubectl delete -f app-deployment.yaml
kubectl delete -f clusterip-service.yaml
kubectl delete -f nodeport-service.yaml
kubectl delete -f loadbalancer-service.yaml

kubectl get pods
kubectl get services
```

---

## Verification

✔️ All resources deleted
✔️ Only default Kubernetes service remains

---

# 📌 Key Takeaways

* Pods are ephemeral → Services provide stability
* ClusterIP = internal communication
* NodePort = basic external access
* LoadBalancer = production-grade external access
* DNS is built into Kubernetes

---

# 🧩 Conclusion

Today I connected the missing piece:

Pods → Deployments → Services

Now applications are not just running —
they are **accessible, stable, and scalable**.

---