# Day 56 – Kubernetes StatefulSets

---

## 🚀 Overview

Today I moved from **stateless workloads → stateful workloads**.

👉 Deployments are great for stateless apps
👉 But databases need **identity, order, and persistence**

This is where **StatefulSets** come in.

---


# 🧠 Task 1: Why StatefulSets?

## Deployment Behavior

* Pod names: `nginx-xyz-abc` (random)
* On deletion → new pod gets **different name**

❌ Problem for databases:

* Replication depends on **stable identity**
* Nodes must know each other (e.g., `db-0`, `db-1`)

---

## Key Difference

| Feature   | Deployment  | StatefulSet               |
| --------- | ----------- | ------------------------- |
| Pod Names | Random      | Stable (`web-0`, `web-1`) |
| Startup   | Parallel    | Ordered                   |
| Storage   | Shared      | Per-pod                   |
| Network   | No identity | Stable DNS                |

---

# 🧩 Task 2: Headless Service

## Manifest: `headless-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
    name: http
```

---

## Apply

```
kubectl apply -f headless-service.yaml
kubectl get svc
```
<img width="1125" height="126" alt="image" src="https://github.com/user-attachments/assets/16f8de4f-03d8-4651-87b0-ee06ca577b83" />

---

## Verification

✔️ CLUSTER-IP: **None**

👉 Enables direct pod DNS instead of load balancing

---

# 🧩 Task 3: StatefulSet

## Manifest: `statefulset.yaml`

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        volumeMounts:
        - name: web-data
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: web-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 100Mi
```

---

## Apply & Watch

```
kubectl apply -f statefulset.yaml
kubectl get pods -l app=web -w
```
<img width="1196" height="317" alt="image" src="https://github.com/user-attachments/assets/e3ba466b-3d2b-4ab3-a0c2-47d117574eab" />

---

## Verification

✔️ Pods created in order:

* web-0
* web-1
* web-2

✔️ PVCs created:

* web-data-web-0
* web-data-web-1
* web-data-web-2

---

# 🌐 Task 4: Stable Network Identity

## DNS Format

```
<pod-name>.<service-name>.<namespace>.svc.cluster.local
```

---

## Test

```
kubectl run dns-test --image=busybox -it --rm -- sh

nslookup web-0.web.default.svc.cluster.local
nslookup web-1.web.default.svc.cluster.local
nslookup web-2.web.default.svc.cluster.local
```
<img width="1212" height="411" alt="image" src="https://github.com/user-attachments/assets/bcff77b1-814b-46f9-ab22-9db1dcfacd45" />

---

## Verification

✔️ DNS resolves to correct Pod IPs
✔️ Matches `kubectl get pods -o wide`

---

# 💾 Task 5: Persistent Storage

## Write Data

```
kubectl exec web-0 -- sh -c "echo 'Data from web-0' > /usr/share/nginx/html/index.html"
```

---

## Delete Pod

```
kubectl delete pod web-0
```

---

## Verify

```
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
```
<img width="1206" height="181" alt="image" src="https://github.com/user-attachments/assets/e04f6498-dc02-4814-9af7-fe88e386bc28" />

---

## Result

✔️ Data is still present

👉 Same PVC reattached to recreated pod

---

# ⚙️ Task 6: Ordered Scaling

## Scale Up

```
kubectl scale statefulset web --replicas=5
```

✔️ Pods created:

* web-3
* web-4

---

## Scale Down

```
kubectl scale statefulset web --replicas=3
```

✔️ Pods removed in reverse order:

* web-4
* web-3

---

## Verification

```
kubectl get pvc
```

✔️ **All 5 PVCs still exist**

👉 Kubernetes preserves storage

---

# 🧹 Task 7: Cleanup

```
kubectl delete statefulset web
kubectl delete svc web
kubectl get pvc
```

---

## Observation

✔️ PVCs are NOT deleted automatically

---

## Final Cleanup

```
kubectl delete pvc web-data-web-0 web-data-web-1 web-data-web-2 web-data-web-3 web-data-web-4
```

---

# 📌 Key Takeaways

* StatefulSets provide **stable identity**
* Each pod gets its own **persistent storage**
* Headless Service enables **direct DNS resolution**
* Scaling is **ordered and predictable**
* PVCs are preserved even after scaling down or deletion

---

# 🧩 Conclusion

StatefulSets unlock **real-world workloads**:

* Databases (MySQL, PostgreSQL)
* Messaging systems (Kafka)
* Distributed systems

👉 Kubernetes is no longer just about running containers —
it’s about managing **stateful distributed systems reliably**.
