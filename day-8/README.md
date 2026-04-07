# Day 8 – Metrics Server & Horizontal Pod Autoscaler (HPA)

---

## 🚀 Overview

Today I enabled **auto-scaling in Kubernetes** based on real-time resource usage.

👉 Yesterday: defined CPU & memory
👉 Today: used those metrics to **scale applications automatically**

Core components:

* **Metrics Server** → collects resource usage
* **HPA (Horizontal Pod Autoscaler)** → scales pods dynamically

---



# 🧠 Task 1: Install Metrics Server

## Check

```
kubectl get pods -n kube-system | grep metrics-server
```

---

## Install (if not present)

### Minikube

```
minikube addons enable metrics-server
```

### Kind / kubeadm

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

⚠️ For local clusters:

* May require `--kubelet-insecure-tls`

---

## Verify

```
kubectl top nodes
kubectl top pods -A
```
<img width="1192" height="389" alt="image" src="https://github.com/user-attachments/assets/13cbfe1c-e196-449c-889f-f22cd0e1d43b" />

---

## Result

✔️ CPU & Memory usage visible

👉 Metrics Server working correctly

---

# 📊 Task 2: Explore `kubectl top`

## Commands

```
kubectl top nodes
kubectl top pods -A
kubectl top pods -A --sort-by=cpu
```
<img width="1158" height="309" alt="image" src="https://github.com/user-attachments/assets/671b550b-2355-4778-b698-017539612dd1" />

---

## Insight

* Shows **actual usage**, not requests/limits
* Updated every ~15 seconds

---

## Verification

✔️ Identified highest CPU-consuming pod

---

# ⚙️ Task 3: Create Deployment (HPA Target)

## Manifest

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "200m"
```

---

## Apply & Expose

```
kubectl apply -f php-apache.yaml
kubectl expose deployment php-apache --port=80
```

---

## Key Insight

👉 HPA requires **CPU requests**
Without it → autoscaling fails

---

# 📈 Task 4: Create HPA (Imperative)

```
kubectl autoscale deployment php-apache \
  --cpu-percent=50 \
  --min=1 \
  --max=10
```

---

## Check

```
kubectl get hpa
kubectl describe hpa php-apache
```
<img width="1190" height="368" alt="image" src="https://github.com/user-attachments/assets/82597dfc-14d9-4fb1-a295-9c59fade72b3" />

---

## Verification

✔️ TARGETS shows CPU usage vs target

Example:

```
30% / 50%
```

---

# 🔥 Task 5: Generate Load

## Start Load

```
kubectl run load-generator \
  --image=busybox:1.36 \
  --restart=Never \
  -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
```

---

## Watch Scaling

```
kubectl get hpa php-apache --watch
```
<img width="1122" height="143" alt="image" src="https://github.com/user-attachments/assets/abdda520-b9b6-439f-b15e-313dbe01a565" />

---

## Result

✔️ CPU usage increased
✔️ Replicas scaled up automatically

---

## Stop Load

```
kubectl delete pod load-generator
```

---

## Insight

* Scale up = fast
* Scale down = slow (stabilization window)

---

# 📜 Task 6: HPA via YAML (Declarative)

## Manifest

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

---

## Apply

```
kubectl apply -f hpa.yaml
kubectl describe hpa php-apache
```
<img width="1199" height="606" alt="image" src="https://github.com/user-attachments/assets/d1501127-277b-42c0-9b0c-f4df43933a23" />

---

## Verification

✔️ Behavior controls:

* Scale-up speed (fast reaction)
* Scale-down delay (avoid flapping)
<img width="754" height="586" alt="image" src="https://github.com/user-attachments/assets/c3564ea7-34a1-44c3-8075-a7d5c7cf37bc" />

---

# 🧹 Task 7: Cleanup

```
kubectl delete hpa php-apache
kubectl delete svc php-apache
kubectl delete deployment php-apache
kubectl delete pod load-generator
```

---

# 📌 Key Takeaways

* Metrics Server → provides real-time usage
* `kubectl top` → observability tool
* HPA → automatic scaling based on metrics
* CPU requests are mandatory for HPA

---

## Scaling Logic

```
Current CPU > Target → Scale Up
Current CPU < Target → Scale Down
```

---

# 🧩 Conclusion

Today I built **self-scaling infrastructure**:

Kubernetes now:

* Observes load
* Makes decisions
* Adjusts capacity automatically

👉 This is the foundation of **cloud-native elasticity**
