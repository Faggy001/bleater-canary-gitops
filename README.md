# Bleater API Gateway – Canary Routing with NGINX Ingress (GitOps)

## 📌 Overview

This repository demonstrates **canary routing using NGINX Ingress Controller** in Kubernetes, implemented in a **GitOps-ready structure**. The goal is to safely roll out a new version of an application (`v2`) alongside a stable version (`v1`) and gradually shift traffic using **NGINX Ingress canary annotations**.

🚫 **No service mesh is used** (no Istio, Linkerd, or Argo Rollouts).
✅ Traffic splitting is handled **purely by NGINX Ingress**.

The setup is:

* Declarative
* Idempotent
* Safe to apply multiple times
* Compatible with **Minikube**, **Kind**, and cloud clusters

---

## 🧠 What This Project Demonstrates

* Canary deployments using **NGINX Ingress weight-based routing**
* GitOps-friendly repository layout
* Environment-based overlays using **Kustomize**
* Observable traffic distribution using HTTP responses
* Local Kubernetes development with **Minikube**

---

## 🏗 Architecture

```
User
 │
 ▼
NGINX Ingress Controller
 │
 ├── Stable Service  ──> bleater-api-gateway v1
 │
 └── Canary Service   ──> bleater-api-gateway v2
```

* Both versions run simultaneously
* NGINX Ingress routes traffic based on configured canary weight

---

### 🔹 `base/`

Contains **shared Kubernetes manifests**:

* Namespace
* Deployments (v1 and v2)
* Services (stable and canary)
* Ingress resources

### 🔹 `overlays/`

Defines **environment-specific behavior**:

* `stable` → 100% traffic to v1
* `canary-10` → 10% traffic to v2
* `canary-40` → 40% traffic to v2

### 🔹 `gitops/argocd/`

Contains **Argo CD Application manifest** for GitOps-based deployment.

---

## 🚀 Application Behavior

The application uses a simple NGINX container to expose a version endpoint:

```bash
http://bleater.local/version
```

Responses:

* `v1` → stable deployment
* `v2` → canary deployment

This makes traffic distribution **easy to observe** via browser or `curl`.

---

## 🧰 Prerequisites

* Docker
* kubectl
* Minikube
* Git

Optional:

* Argo CD (for GitOps automation)

---

## ▶️ Runbook (Start to Finish)

### 1️⃣ Start Minikube

```bash
minikube start
```

---

### 2️⃣ Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

Wait for controller to be ready:

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

---

### 3️⃣ Enable Minikube Tunnel

```bash
minikube tunnel
```

> ⚠️ Keep this terminal open

---

### 4️⃣ Configure Local DNS

Get Minikube IP:

```bash
minikube ip
```

Add to `/etc/hosts`:

```text
<MINIKUBE_IP>  bleater.local
```

Verify:

```bash
ping bleater.local
```

---

### 5️⃣ Deploy Canary (10%)

```bash
kubectl apply -k apps/bleater-api-gateway/overlays/canary-10
```

Verify resources:

```bash
kubectl get pods -n bleater
kubectl get ingress -n bleater
```

---

### 6️⃣ Test Traffic Distribution

Single request:

```bash
curl http://bleater.local/version
```

Continuous testing:

```bash
while true; do
  curl -s http://bleater.local/version
  sleep 0.3
done
```

Expected output:

```
v1
v1
v2
v1
v1
```

---

### 7️⃣ Increase Canary Traffic

```bash
kubectl apply -k apps/bleater-api-gateway/overlays/canary-40
```

Observe increased frequency of `v2` responses.

---

## 🔍 Observability

View canary logs:

```bash
kubectl logs -n bleater -l version=v2 -f
```

Ingress details:

```bash
kubectl describe ingress bleater-api-gateway-canary -n bleater
```

---

## 🔁 GitOps with Argo CD

To deploy using Argo CD:

```bash
kubectl apply -f gitops/argocd/bleater-app.yaml
```

Argo CD will:

* Watch the Git repository
* Sync the selected overlay
* Automatically apply changes

Rolling back is as simple as reverting a Git commit.

---

## 🔐 Design Constraints (Respected)

* ❌ No Argo Rollouts
* ❌ No Istio / Service Mesh
* ❌ No custom load balancers
* ✅ Only NGINX Ingress canary annotations

---

## 🧪 Why This Approach Works Well

* Simple and transparent
* Easy to reason about
* Ideal for demos and interviews
* Fully GitOps compatible
* Works on local and cloud clusters

---

## 🏁 Conclusion

This project showcases a **production-style canary deployment strategy** using only Kubernetes-native tooling and NGINX Ingress. It provides a clean, auditable, and observable rollout process that aligns perfectly with GitOps principles.

---

✍️ Author: *Fagoroye Sanumi Oluwaseyi*

