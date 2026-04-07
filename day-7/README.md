# Day 7 – Resource Requests, Limits, and Probes

---

## 🚀 Overview

Today I focused on **production-grade reliability in Kubernetes**:

👉 Resource management (CPU & Memory)
👉 Health checking (Probes)

This is where Kubernetes shifts from *running containers* → *operating systems for containers*.

---


# 🧠 Task 1: Resource Requests & Limits

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "250m"
        memory: "256Mi"
```

---

## Apply & Inspect

```
kubectl apply -f resource-pod.yaml
kubectl describe pod resource-pod
```
<img width="1195" height="613" alt="image" src="https://github.com/user-attachments/assets/3ebf28cf-ba7f-4ac7-8079-6e0e4d2af287" />

---

## Verification

✔️ Requests: 100m CPU / 128Mi memory
✔️ Limits: 250m CPU / 256Mi memory

### QoS Class

✔️ **Burstable**

👉 Because requests ≠ limits

---

# 💣 Task 2: OOMKilled

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: stress-pod
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--vm-hang", "1"]
    resources:
      limits:
        memory: "100Mi"
```

---

## Apply

```
kubectl apply -f stress-pod.yaml
kubectl describe pod stress-pod
```
<img width="1200" height="614" alt="image" src="https://github.com/user-attachments/assets/5827245b-6736-4f6a-a14e-75b097d91357" />

---

## Verification

✔️ STATUS: CrashLoopBackOff
✔️ Reason: **OOMKilled**
✔️ Exit Code: **137**

👉 Memory exceeded → container killed immediately

---

# 🚫 Task 3: Pending Pod (Insufficient Resources)

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: huge-request-pod
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      requests:
        cpu: "100"
        memory: "128Gi"
```

---

## Apply

```
kubectl apply -f huge-request-pod.yaml
kubectl describe pod huge-request-pod
```
<img width="1199" height="609" alt="image" src="https://github.com/user-attachments/assets/e0c5a0f9-3768-4f64-a37c-817fa5a68e2f" />

---

## Verification

✔️ STATUS: Pending

### Event Message

```
0/1 nodes are available: insufficient cpu, insufficient memory
```

👉 Scheduler cannot place the Pod

---

# 🔁 Task 4: Liveness Probe

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600"]
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
```

---

## Watch

```
kubectl get pod liveness-pod -w
```
<img width="801" height="132" alt="image" src="https://github.com/user-attachments/assets/67ed3333-722d-4d42-82ba-d655bfd3ac43" />

---

## Verification

✔️ Container restarted automatically
✔️ Restart count increases

👉 Liveness probe detects failure → restarts container

---

# 🌐 Task 5: Readiness Probe

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
  labels:
    app: readiness-app
spec:
  containers:
  - name: nginx
    image: nginx
    readinessProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 5
```

---

## Expose Service

```
kubectl expose pod readiness-pod --port=80 --name=readiness-svc
kubectl get endpoints readiness-svc
```

---

## Break Readiness

```
kubectl exec readiness-pod -- rm /usr/share/nginx/html/index.html
```

---

## Verification

✔️ Pod shows **0/1 READY**
✔️ Endpoints become empty
✔️ Container is NOT restarted

👉 Readiness controls traffic, not lifecycle

---

# ⏳ Task 6: Startup Probe

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: startup-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 20; touch /tmp/started; sleep 600"]
    startupProbe:
      exec:
        command:
        - cat
        - /tmp/started
      periodSeconds: 5
      failureThreshold: 12
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/started
      periodSeconds: 5
```

---

## Verification

✔️ Startup probe allows 60 seconds (5 × 12)
✔️ Liveness starts only after startup succeeds

---

## Insight

👉 If failureThreshold = 2:

* Container killed after ~10 seconds
* App would never start

---

# 🧹 Task 7: Cleanup

```
kubectl delete pod resource-pod stress-pod huge-request-pod liveness-pod readiness-pod startup-pod
kubectl delete svc readiness-svc
```

---

# 📌 Key Takeaways

* Requests → scheduling guarantee
* Limits → runtime enforcement
* CPU → throttled
* Memory → killed (OOMKilled)

---

## Probes Summary

| Probe     | Purpose               | Action              |
| --------- | --------------------- | ------------------- |
| Liveness  | Detect dead/stuck app | Restart container   |
| Readiness | Control traffic       | Remove from service |
| Startup   | Handle slow startup   | Delay other probes  |

---

# 🧩 Conclusion

Today was about **resilience and reliability**:

Kubernetes is not just deploying apps —
it is actively **monitoring, healing, and optimizing them**.