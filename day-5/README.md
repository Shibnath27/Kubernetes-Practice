# Day 5 – Persistent Volumes (PV) and Persistent Volume Claims (PVC)

---

## 🚀 Overview

Today I solved one of the most critical problems in Kubernetes:

👉 **Containers are ephemeral — storage is not**

To handle persistent data, Kubernetes provides:

* **PersistentVolume (PV)** → actual storage resource
* **PersistentVolumeClaim (PVC)** → request for storage

---



# 🧠 Task 1: Ephemeral Storage Problem

## Pod with emptyDir

```
apiVersion: v1
kind: Pod
metadata:
  name: ephemeral-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "date >> /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: temp-storage
      mountPath: /data
  volumes:
  - name: temp-storage
    emptyDir: {}
```

---

## Test

```
kubectl exec ephemeral-pod -- cat /data/message.txt
```
<img width="844" height="49" alt="image" src="https://github.com/user-attachments/assets/00c20ff5-369c-4007-9095-ac7f8c5b6fff" />

---

## Result

✔️ File exists

---

## After Deletion & Recreation

```
kubectl delete pod ephemeral-pod
kubectl apply -f ephemeral-pod.yaml
```

✔️ Old data is gone
✔️ New timestamp created

👉 **Data is NOT persistent**

---

# 🧩 Task 2: Create PersistentVolume

## PV Manifest

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  hostPath:
    path: /tmp/k8s-pv-data
```

---

## Apply

```
kubectl apply -f pv.yaml
kubectl get pv
```
<img width="1119" height="112" alt="image" src="https://github.com/user-attachments/assets/cab2ecdc-65db-4ff3-85e4-589a716f0113" />

---

## Verification

✔️ STATUS: **Available**

---

# 🧩 Task 3: Create PersistentVolumeClaim

## PVC Manifest

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  resources:
    requests:
      storage: 500Mi
```

---

## Apply

```
kubectl apply -f pvc.yaml
kubectl get pvc
kubectl get pv
```
<img width="1129" height="114" alt="image" src="https://github.com/user-attachments/assets/f3d74a62-aafe-4335-850d-98bb799d30f5" />

---

## Verification

✔️ PVC STATUS: **Bound**
✔️ PV STATUS: **Bound**

👉 PVC successfully matched to PV

---

# 🧩 Task 4: Use PVC in Pod

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: persistent-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "date >> /data/message.txt && sleep 3600"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: manual-pvc
```

---

## Test

```
kubectl exec persistent-pod -- cat /data/message.txt
```
<img width="854" height="49" alt="image" src="https://github.com/user-attachments/assets/e7c0d5bf-e796-43bf-8515-23e620b16b16" />

---

## Delete & Recreate

```
kubectl delete pod persistent-pod
kubectl apply -f persistent-pod.yaml
kubectl exec persistent-pod -- cat /data/message.txt
```
<img width="849" height="294" alt="image" src="https://github.com/user-attachments/assets/a6471cfc-2a03-4f25-88b5-64d7583b639f" />

Why the timestamp changed
The new pod ran its init/command again and overwrote message.txt with the current time. The persistence is working — the file survived pod deletion, it just got overwritten on restart.
If you want to prove the old data survives without overwriting, you could:
```
kubectl exec persistent-pod -- ls -la /data/

```
Or append instead of overwrite in your pod command (>> instead of >).
---

## Verification

✔️ File contains **multiple timestamps**

👉 **Data persisted across Pod recreation**

---

# 🧠 Task 5: StorageClasses

## Commands

```
kubectl get storageclass
kubectl describe storageclass
```
<img width="1192" height="281" alt="image" src="https://github.com/user-attachments/assets/8bd679c9-89b2-4e56-bc7e-b6a033ef396f" />

---

## Observation

* Found default StorageClass
* Includes:

  * Provisioner
  * Reclaim policy
  * Volume binding mode

---

## Insight

👉 With StorageClasses:

* You create PVC
* Kubernetes automatically provisions PV

---

# ⚙️ Task 6: Dynamic Provisioning

## PVC Manifest (Dynamic)

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 500Mi
```

---

## Apply

```
kubectl apply -f dynamic-pvc.yaml
kubectl get pvc
kubectl get pv
```
<img width="1122" height="104" alt="image" src="https://github.com/user-attachments/assets/31c21668-636b-4f45-b1f3-81a87c863516" />

Why dynamic-pvc is Pending
standard StorageClass exists ✅, but its binding mode is WaitForFirstConsumer — meaning it won't create the PV until a Pod actually requests it.
It's waiting for a pod to use the PVC before provisioning storage.
## Create a pod that uses dynamic-pvc
```
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo Dynamic PV Works > /data/message.txt && sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: dynamic-storage
  volumes:
  - name: dynamic-storage
    persistentVolumeClaim:
      claimName: dynamic-pvc

```
## Apply

```
kubectl apply -f dynamic-pod.yaml
kubectl get pvc
kubectl get pv
```
<img width="1190" height="278" alt="image" src="https://github.com/user-attachments/assets/6dda52f2-186b-4f45-9ca6-e2912b8a9626" />

---

## Verification

✔️ New PV created automatically
✔️ PVC bound successfully

---

## Insight

* manual-pv → static provisioning
* dynamic-pvc → dynamic provisioning

---

# 🧹 Task 7: Cleanup

```
kubectl delete pod persistent-pod ephemeral-pod
kubectl delete pvc manual-pvc dynamic-pvc
kubectl get pv
```

---

## Observation

* Dynamic PV → deleted automatically
* Manual PV → STATUS = Released

---

## Final Cleanup

```
kubectl delete pv manual-pv
```

---

# 📌 Key Takeaways

* Containers are ephemeral → storage must not be
* PV = actual storage
* PVC = request for storage
* emptyDir = temporary
* StorageClasses enable automation
* Reclaim policies matter:

  * Retain → manual cleanup
  * Delete → auto cleanup

---

# 🧩 Conclusion

Today I learned how Kubernetes handles **stateful workloads**:

Stateless → Pods
Stateful → PV + PVC

This is the foundation for running:

* Databases
* Logs
* Persistent applications
