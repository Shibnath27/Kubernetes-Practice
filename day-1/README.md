# Day 1 – Kubernetes Manifests and Your First Pods

---

## 🚀 Overview

Today I moved from setting up a Kubernetes cluster to actually **deploying workloads** using YAML manifests. The focus was on understanding the structure of manifests and creating Pods from scratch.

---

## ✅ Expected Output

* ✔️ 3 Pod manifests created manually
* ✔️ Pods deployed and verified
* ✔️ `day-51-pods.md` documented
* ✔️ Screenshot of `kubectl get pods` (attached separately)

---

# 🧠 Kubernetes Manifest Basics

Every Kubernetes resource requires four key fields:

### 1. apiVersion

* Defines which API version Kubernetes should use
* Example: `v1` for Pods

---

### 2. kind

* Specifies the resource type
* Example: `Pod`

---

### 3. metadata

* Identifies the object
* Includes:

  * `name` (required)
  * `labels` (used for grouping/filtering)

---

### 4. spec

* Defines the desired state
* Includes:

  * Containers
  * Images
  * Ports
  * Commands

---

# 🧩 Task 1: Nginx Pod

## Manifest: `nginx-pod.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

## Commands

```
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
kubectl logs nginx-pod
kubectl exec -it nginx-pod -- /bin/bash
```

## Verification

* Pod reached **Running** state
* Accessed container shell
* Verified Nginx using:

```
curl localhost:80
```

✔️ Nginx welcome page confirmed

---

# 🧩 Task 2: BusyBox Pod

## Manifest: `busybox-pod.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
  labels:
    app: busybox
    environment: dev
spec:
  containers:
  - name: busybox
    image: busybox:latest
    command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

## Commands

```
kubectl apply -f busybox-pod.yaml
kubectl get pods
kubectl logs busybox-pod
```

## Verification

✔️ Logs output:

```
Hello from BusyBox
```

### Key Insight

* BusyBox is not a long-running service
* Without `sleep`, container exits → Pod crashes
* This leads to **CrashLoopBackOff**

---

# 🧩 Task 3: Third Pod (Custom)

## Manifest: `custom-pod.yaml`

```
apiVersion: v1
kind: Pod
metadata:
  name: custom-pod
  labels:
    app: demo
    environment: testing
    team: devops
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

## Apply

```
kubectl apply -f custom-pod.yaml
```

---

# ⚖️ Imperative vs Declarative

## Imperative Approach

```
kubectl run redis-pod --image=redis:latest
```

* Quick and direct
* Not reusable
* Not version-controlled

---

## Declarative Approach

```
kubectl apply -f <file>.yaml
```

* Uses YAML manifests
* Version-controlled
* Repeatable and production-ready

---

## YAML Generation Trick

```
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```

* Generates manifest template
* Helps bootstrap configurations quickly

---

# 🔍 Validation

## Client-side validation

```
kubectl apply -f nginx-pod.yaml --dry-run=client
```

## Server-side validation

```
kubectl apply -f nginx-pod.yaml --dry-run=server
```

---

## Error Case

When `image` field is missing:

➡️ Kubernetes throws validation error similar to:

```
error: spec.containers[0].image: Required value
```

---

# 🏷️ Labels and Filtering

## View labels

```
kubectl get pods --show-labels
```

## Filter pods

```
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev
```

## Add label

```
kubectl label pod nginx-pod environment=production
```

## Remove label

```
kubectl label pod nginx-pod environment-
```

---

# 🧹 Cleanup

```
kubectl delete pod nginx-pod
kubectl delete pod busybox-pod
kubectl delete pod redis-pod
kubectl delete -f nginx-pod.yaml
kubectl get pods
```

---

# ⚠️ Important Insight

When you delete a standalone Pod:

* It is **permanently removed**
* Kubernetes does NOT recreate it

👉 Reason:

* No controller is managing it

---

# 📸 Final Verification

Command:

```
kubectl get pods
```

✔️ All pods reached **Running** state during testing
✔️ Successfully deleted during cleanup

(Screenshot attached separately)

---

# 📌 Key Takeaways

* Pods are the **smallest deployable unit** in Kubernetes
* YAML manifests define **desired state**
* Declarative approach is **industry standard**
* Labels are critical for **organization and selection**
* Standalone Pods are not production-ready

---

# 🧩 Conclusion

Today established the foundation for deploying workloads in Kubernetes:

* Writing manifests from scratch
* Running and debugging pods
* Understanding lifecycle and behavior

