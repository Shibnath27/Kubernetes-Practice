# Day 59 – Helm: Kubernetes Package Manager

---

## 🚀 Overview

After manually creating multiple Kubernetes resources (Pods, Deployments, Services, ConfigMaps, Secrets, PVCs), today I streamlined everything using **Helm**.

👉 Helm = **package manager for Kubernetes**
👉 Think: `apt` for Ubuntu, but for Kubernetes applications

---



# 🧠 Task 1: Install Helm

## Installation
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
```
## Verify Installation

```
helm version
helm env
```
<img width="1188" height="558" alt="image" src="https://github.com/user-attachments/assets/e6980477-011d-4548-b1db-13f2a04d1a2e" />

---

## Core Concepts

* **Chart** → Collection of Kubernetes manifests (templates)
* **Release** → Running instance of a chart
* **Repository** → Collection of charts

---

## Result

✔️ Helm installed and ready

---

# 📦 Task 2: Add Repository & Search

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## Search Charts

```
helm search repo nginx
helm search repo bitnami
```

---

## Result

✔️ Bitnami repo contains **hundreds of production-ready charts**

---

# ⚙️ Task 3: Install a Chart

```
helm install my-nginx bitnami/nginx
```

---

## Verify

```
kubectl get all
helm list
helm status my-nginx
helm get manifest my-nginx
```
<img width="1210" height="240" alt="image" src="https://github.com/user-attachments/assets/8954f253-f08c-457b-98d1-72781f83dbc9" />

---

## Insight

👉 One Helm command created:

* Deployment
* Service
* Supporting resources

---

## Result

✔️ Nginx running via Helm release

---

# 🎛️ Task 4: Customize with Values

## View Default Values

```
helm show values bitnami/nginx
```

---

## Override via CLI

```
helm install nginx-custom bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=NodePort
```

---

## Using Values File

### custom-values.yaml

```
replicaCount: 3

service:
  type: NodePort

resources:
  limits:
    cpu: 200m
    memory: 256Mi
```

---

## Install with Values File

```
helm install nginx-values bitnami/nginx -f custom-values.yaml
```

---

## Verify

```
helm get values nginx-values
kubectl get pods
kubectl get svc
```
<img width="775" height="303" alt="image" src="https://github.com/user-attachments/assets/88d497a1-26b1-4da4-afd8-fe9cb778b253" />

---

## Result

✔️ Custom replicas and service type applied

---

# 🔁 Task 5: Upgrade & Rollback

## Upgrade

```
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```

---

## History

```
helm history my-nginx
```

---

## Rollback

```
helm rollback my-nginx 1
```
<img width="725" height="303" alt="image" src="https://github.com/user-attachments/assets/c98b87f2-5e90-4c58-ada8-76ef42c97d7e" />

---

## Result

✔️ Rollback created new revision

---

## Insight

👉 Helm maintains **full release history**
👉 Similar to Deployment rollouts, but broader scope

---

# 🏗️ Task 6: Create Your Own Chart

## Scaffold

```
helm create my-app
```

---

## Key Files

* `Chart.yaml` → metadata
* `values.yaml` → configuration
* `templates/` → Kubernetes manifests

---

## Modify Values

```
replicaCount: 3

image:
  repository: nginx
  tag: 1.25
```

---

## Validate & Preview

```
helm lint my-app
helm template my-release ./my-app
```

---

## Install

```
helm install my-release ./my-app --namespace my-app --create-namespace
```
<img width="1199" height="318" alt="image" src="https://github.com/user-attachments/assets/196b37db-55de-4e0f-a131-9c22d06e35ab" />

---

## Upgrade

```
helm upgrade my-release ./my-app --set replicaCount=5 -n my-app
```

---

## Result

✔️ Custom chart deployed and scaled

---

# 🧹 Task 7: Clean Up

```
helm uninstall my-nginx
helm uninstall nginx-custom
helm uninstall nginx-values
helm uninstall my-release
```

---

## Verify

```
helm list
```

---

## Result

✔️ No active releases

---

# 📌 Key Takeaways

* Helm simplifies complex Kubernetes deployments
* Charts = reusable infrastructure templates
* Values = dynamic configuration
* Upgrade/Rollback = version control for infrastructure

---

## 🧠 Big Insight

Without Helm:
👉 Dozens of YAML files

With Helm:
👉 One command deploys everything

---

# 🧩 Conclusion

Today I moved from:

❌ Manual YAML management
➡️
✅ **Packaged, reusable, version-controlled deployments**

👉 This is how real-world Kubernetes applications are managed at scale.
