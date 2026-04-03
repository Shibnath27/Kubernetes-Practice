# Day 4 – Kubernetes ConfigMaps and Secrets

---

## 🚀 Overview

Today I implemented **configuration management in Kubernetes using YAML manifests**, focusing on:

* ConfigMaps (non-sensitive data)
* Secrets (sensitive data)

Also explored **dynamic updates and consumption patterns**.

---


# 🧠 Task 1: ConfigMap from Literals

## Command Used

```
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

## Verification

```
kubectl get configmap app-config -o yaml
```
<img width="600" height="264" alt="image" src="https://github.com/user-attachments/assets/8202db6d-4756-4f43-87ed-9ef7e7258833" />

✔️ All key-value pairs visible
✔️ Stored as plain text

---

# 🧩 Task 2: ConfigMap from File (YAML)

## Nginx Config File (`default.conf`)

```
server {
    listen 80;

    location /health {
        return 200 'healthy';
    }
}
```

---

## ConfigMap YAML: `nginx-config.yaml`

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;

        location /health {
            return 200 'healthy';
        }
    }
```

---

## Apply

```
kubectl apply -f nginx-config.yaml
```

---

## Verification

```
kubectl get configmap nginx-config -o yaml
```
<img width="1183" height="398" alt="image" src="https://github.com/user-attachments/assets/c68c2e28-b496-442b-9a6b-e16adc8ac776" />

✔️ File content correctly embedded

---

# ⚙️ Task 3: Use ConfigMaps in Pods

## Env-based Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
```

---

## Volume-based Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-config-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
```

---

## Test

```
kubectl exec nginx-config-pod -- sh -c "curl -s http://localhost/health && echo"

```

✔️ Response: **healthy**

---

# 🔐 Task 4: Secret (YAML)

## Secret YAML: `db-secret.yaml`

```
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  DB_USER: YWRtaW4=              # base64(admin)
  DB_PASSWORD: czNjdXJlUEBzc3cwcmQ=   # base64(s3cureP@ssw0rd)
```

---

## Apply

```
kubectl apply -f db-secret.yaml
```

---

## Verification

```
kubectl get secret db-credentials -o yaml
```
<img width="1192" height="280" alt="image" src="https://github.com/user-attachments/assets/1f49a710-ecde-4a7f-bc6e-d634b344f808" />

---

## Decode Example

```
echo 'czNjdXJlUEBzc3cwcmQ=' | base64 --decode
```

✔️ Output: `s3cureP@ssw0rd`

---

## Insight

* Base64 = encoding, NOT encryption
* Security relies on access control (RBAC)

---

# 🔑 Task 5: Use Secret in Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: DB_USER
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/db-credentials
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
```

---

## Verification

✔️ Env variable injected
✔️ Files mounted in `/etc/db-credentials`

### Important

* Mounted values are **plaintext**, not base64

---

# 🔄 Task 6: Live Config Update

## Create ConfigMap

```
kubectl create configmap live-config --from-literal=message=hello
```

---

## Pod Manifest

```
apiVersion: v1
kind: Pod
metadata:
  name: live-config-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do cat /config/message; sleep 5; done"]
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    configMap:
      name: live-config
```

---

## Update ConfigMap

```
kubectl patch configmap live-config \
  --type merge \
  -p '{"data":{"message":"world"}}'
```

---

## Verification

✔️ Value updated automatically (no restart)

---

## Key Insight

* Volume mount → dynamic update
* Env variables → static

---

# 🧹 Task 7: Cleanup

```
kubectl delete pod configmap-env-pod nginx-config-pod secret-pod live-config-pod
kubectl delete configmap app-config nginx-config live-config
kubectl delete secret db-credentials
```

---

# 📌 Key Takeaways

* ConfigMaps externalize configuration
* Secrets manage sensitive data
* Volume mounts enable real-time updates
* Env variables are fixed at startup
* Base64 is not security

---

# 🧩 Conclusion

This was a critical step toward **production-ready systems**:

Config is no longer baked into images —
it is **dynamic, decoupled, and environment-aware**.

