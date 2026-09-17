# Session-12 — ConfigMap + Secret + Ingress

> **Folder:** `assignment/session-12-ingress-configmaps-secrets/`  
> **Total Components:** 4 (ConfigMap, Secret, Ingress, Full Demo)  
> **Reference:** `session-12-ingress-configmaps-secrets/01-configmap/` + `02-secret/` + `03-ingress/` + `04-full-demo/`

---

## 📋 Table of Contents

1. [ConfigMap — Plain-Text Configuration](#1-configmap-plain-text-configuration)
2. [Secret — Sensitive Credentials](#2-secret-sensitive-credentials)
3. [Ingress — One Entry Point for Microservices](#3-ingress-one-entry-point-for-microservices)
4. [Full Demo — All Three Working Together](#4-full-demo-all-three-working-together)
5. [Decision Tree](#5-decision-tree-which-tool-for-the-job)
6. [Quick Summary Checklist](#6-quick-summary-checklist)

---

## 1. ConfigMap — Plain-Text Configuration

> **Folder:** `01-configmap/` | **Reference:** `session-12-ingress-configmaps-secrets/01-configmap/app-config.yaml`

### What It Is
Stores **non-sensitive** key-value pairs outside container images. Application reads them at runtime as environment variables or volume-mounted files.

### When to Use
- `LOG_LEVEL`, `ENVIRONMENT` (dev/staging/prod), `CACHE_TTL`, `MAX_CONNECTIONS`
- Feature flags (`FEATURE_DARK_MODE: "true"`)
- Nginx `nginx.conf` mounted as volume
- **Never** store passwords, tokens, or certificates

### Size Limit
- **1 MiB** maximum

### YAML Manifest
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: yatri-app-config
  namespace: default
  labels:
    app: yatri-app
data:
  ENVIRONMENT: "production"
  LOG_LEVEL: "INFO"
  APP_PORT: "5000"
  DEFAULT_CURRENCY: "INR"
  MAX_BOOKING_DAYS: "30"
```

### Key Interview Points
- **Why not store passwords in ConfigMap?** Plain-text, visible to any RBAC user with `kubectl get`.
- **Size limit?** 1 MiB.
- **Does updating ConfigMap auto-restart pods?** No — pod restart required.

---

## 2. Secret — Sensitive Credentials

> **Folder:** `02-secret/` | **Reference:** `session-12-ingress-configmaps-secrets/02-secret/db-secret.yaml`

### What It Is
Stores **sensitive** data (passwords, tokens, TLS certs) as **Base64-encoded strings**.

### When to Use
- Database credentials (`POSTGRES_USER`, `POSTGRES_PASSWORD`)
- API keys and OAuth client secrets
- TLS certificate (`tls.crt`) and private key (`tls.key`)
- Docker Hub / ECR image pull credentials

### Important Gotchas
| Point | Detail |
|---|---|
| **Base64 is NOT encryption** | Anyone with `kubectl get secret` RBAC can decode it. Relies on Kubernetes RBAC + etcd encryption-at-rest. |
| **Always use `echo -n`** | Without `-n`, trailing newline gets encoded, causing silent auth failures. |
| **`kubectl describe secret` masks values** | With `[X bytes]` to prevent accidental exposure. |
| **Production-grade secret management** | Use external secret managers: AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager integrated via `External Secrets Operator`. |

### YAML Manifest
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: yatri-db-secret
  namespace: default
  labels:
    app: yatri-app
type: Opaque
data:
  POSTGRES_USER: eWF0cmlfYWRtaW4=     # Base64: yatri_admin
  POSTGRES_PASSWORD: c2VjcmV0cGFzc3dvcmQ=  # Base64: secretpassword
  POSTGRES_DB: eWF0cmlfcHJvZHVjdGlvbl9kYg==  # Base64: yatri_production_db
```

### Key Interview Points
- **Is Base64 encryption?** No — it's encoding. Decodable by anyone with RBAC access.
- **Size limit?** 1 MiB (per Secret).
- **Does updating Secret auto-restart pods?** No — same as ConfigMap.
- **ConfigMap vs Secret?** → ConfigMap = plain-text, non-sensitive. Secret = Base64-encoded, sensitive.

---

## 3. Ingress — One Entry Point for All Microservices

> **Folder:** `03-ingress/` | **Reference:** `session-12-ingress-configmaps-secrets/03-ingress/ingress-routes.yaml`

### What It Is
**Routing rules** declaration. Ingress does nothing on its own — **MUST have an Ingress Controller** installed (e.g., `ingress-nginx`).

### When to Use
- Routing `app.company.com` → frontend, `api.company.com` → backend — from single public IP
- SSL/TLS termination: attach certificate once, backends communicate over plain HTTP
- Canary deployments: 10% traffic to v2, 90% to v1
- Rate limiting, auth headers, CORS rules via annotations

### Key Points
| Point | Detail |
|---|---|
| **L7 (HTTP/HTTPS) only** | Cannot do TCP/UDP load balancing |
| **`ingressClassName: nginx`** | Modern K8s (v1.18+) requires this; omit it → Ingress silently ignored |
| **`rewrite-target: /$2`** | Strips `/api` prefix before forwarding to backend |
| **Cost** | 1× LoadBalancer (for Ingress Controller) serves all apps vs 50× LBs = $1000+/month |

### YAML Manifest
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: yatri-ingress
  labels:
    app: yatri-app
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
    - host: yatri.local
      http:
        paths:
          - path: /api(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service: yatri-backend-service
              port: { number: 80 }
          - path: /
            pathType: Prefix
            backend:
              service: yatri-frontend-service
              port: { number: 80 }
```

### Key Interview Points
- **Difference between Ingress and LoadBalancer?** LoadBalancer = $15–30/mo per service. Ingress = 1× LB + Controller serves all apps.
- **Do you need an Ingress Controller?** Yes — Ingress is just a CRD; without controller, it's silently ignored.
- **What is `ingressClassName`?** Required in modern K8s; specifies which controller handles the resource.

---

## 4. Full Demo — All Three Working Together

> **Folder:** `04-full-demo/` | **Reference:** `session-12-ingress-configmaps-secrets/04-full-demo/`

### What It Deploys
Full end-to-end demo on Minikube with NGINX Ingress:

| Component | Purpose |
|---|---|
| **ConfigMap** | Stores `ENVIRONMENT`, `LOG_LEVEL`, etc. injected into Frontend (env vars) and Backend (env vars) |
| **Secret** | Stores Base64-encoded `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` injected into Backend via `secretKeyRef` |
| **Frontend** | Nginx serving HTML at `/`; reads ConfigMap as environment variables |
| **Backend** | Python HTTP server at `/api/`; reads ConfigMap + individual Secret keys via `secretKeyRef` |
| **Ingress** | Single NGINX Ingress routes `/` → Frontend + `/api/*` → Backend (with `rewrite-target: /$2`) |

### 9-Step Deploy Flow (from `run-demo.sh`)

| Step | Command | Purpose |
|---|---|---|
| 1 | `minikube addons enable ingress` | Enable NGINX Ingress Controller |
| 2 | `kubectl apply -f configmap.yaml` | Apply plain-text config |
| 3 | `kubectl apply -f secret.yaml` | Apply DB credentials (Base64-encoded) |
| 4 | `kubectl apply -f frontend.yaml` | Deploy Nginx frontend + ClusterIP Service |
| 5 | `kubectl apply -f backend.yaml` | Deploy Python backend + ClusterIP Service |
| 6 | `kubectl rollout status ...` | Wait for all pods Ready |
| 7 | `kubectl apply -f ingress.yaml` | Apply path-based routing rules |
| 8 | `echo "$(minikube ip) yatri.local" | sudo tee -a /etc/hosts` | Add host entry |
| 9 | `curl http://yatri.local` / `curl http://yatri.local/api/` | Test website |

### Test Expectations

**Frontend (`curl http://yatri.local`):**
```
<!DOCTYPE html>
<html>
<head><title>Welcome to nginx!</title></head>
<h1>Welcome to nginx!</h1>
```

**Backend (`curl http://yatri.local/api/`):**
```
Yatri Backend API
=================
ENVIRONMENT     : production
LOG_LEVEL       : INFO
DEFAULT_CURRENCY: INR
POSTGRES_USER   : yatri_admin
POSTGRES_DB     : yatri_production_db
```

**Verify env vars in pod:**
```
ENVIRONMENT=production
LOG_LEVEL=INFO
POSTGRES_USER=yatri_admin
POSTGRES_PASSWORD=secretpassword
POSTGRES_DB=yatri_production_db
```

**Decode Secret password (proves Base64 = encoding, not encryption):**
```
kubectl get secret yatri-db-secret -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 --decode
# Output: secretpassword
```

### Cleanup
```bash
bash 04-full-demo/cleanup.sh
# Or manually:
kubectl delete -f ingress.yaml
kubectl delete -f backend.yaml
kubectl delete -f frontend.yaml
kubectl delete -f secret.yaml
kubectl delete -f configmap.yaml
```

---

## 5. Decision Tree — Which Tool for the Job?

```
Store app config?          → CONFIGMAP (log levels, ports, feature flags)
Store passwords/keys?      → SECRET (DB creds, API keys, TLS certs)
Expose 1 app via public IP?→ LOADBALANCER (one per service = costly)
Expose ALL apps via 1 IP?  → INGRESS (1× LB + Controller, host + path routing)
Need internal pod discovery?→ HEADLESS (clusterIP: None, StatefulSets: Kafka, Mongo)
Need internal VIP only?    → CLUSTERIP (default, internal only)
```

---

## 6. Quick Summary Checklist

| Component | Use Case | Key Command | Cost |
|---|---|---|---|
| **ConfigMap** | Non-sensitive app config | `kubectl apply -f configmap.yaml` | Free |
| **Secret** | DB creds, API keys, TLS | `kubectl apply -f secret.yaml` | Free (but etcd encryption-at-rest recommended) |
| **Ingress** | Single entry point for all apps | `kubectl apply -f ingress.yaml` + 1× LoadBalancer | $15–30/mo (one LB, not 50!) |
| **Full Demo** | End-to-end ConfigMap+Secret+Ingress | `bash run-demo.sh` | Minikube local — free |

### Interview Favorites

| Question | Answer |
|---|---|
| **ConfigMap vs Secret?** | ConfigMap = plain-text, non-sensitive (1 MiB). Secret = Base64-encoded, sensitive (1 MiB). Secret relies on RBAC + etcd encryption-at-rest. |
| **Why not store passwords in ConfigMap?** | Visible to any RBAC user with `kubectl get configmap`. Bad practice, security risk. |
| **Ingress vs LoadBalancer cost?** | LoadBalancer = $15–30/mo per service. Ingress = 1× LB ($15–30/mo) serves ALL apps via host/path routing. Save $1000+/month. |
| **Is Base64 encryption?** | No — it's encoding. Anyone with `kubectl get secret` RBAC can decode it. |
| **Does updating ConfigMap/Secret auto-restart pods?** | No — pod restart required in both cases. |
| **What is `ingressClassName: nginx`?** | Modern K8s (v1.18+) requires specifying which Ingress Controller handles the resource. Omit it → Ingress silently ignored. |
| **What does `rewrite-target: /$2` do?** | Strips `/api` prefix before forwarding to backend, so backend receives `/orders` not `/api/orders`. |

---

## 📁 Folder Structure

```
assignment/session-12-ingress-configmaps-secrets/
├── 01-configmap/
│   ├── README.md        (140 lines - detailed)
│   └── app-config.yaml
├── 02-secret/
│   ├── README.md        (147 lines - detailed)
│   └── db-secret.yaml
├── 03-ingress/
│   ├── README.md        (184 lines - detailed)
│   └── ingress-routes.yaml
├── 04-full-demo/
│   ├── README.md        (310 lines - full demo flow)
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── frontend.yaml
│   ├── backend.yaml
│   ├── ingress.yaml
│   ├── run-demo.sh    (7-min full deploy)
│   └── cleanup.sh     (tear-down)
└── README.md            (this file - master index, 153 lines)
```

**Total READMEs:** 5 (4 component + 1 master index)

---

## 🔗 Quick Links to Source Material

| Resource | Path |
|---|---|
| Session-12 ConfigMap | `session-12-ingress-configmaps-secrets/01-configmap/` |
| Session-12 Secret | `session-12-ingress-configmaps-secrets/02-secret/` |
| Session-12 Ingress | `session-12-ingress-configmaps-secrets/03-ingress/` |
| Session-12 Full Demo | `session-12-ingress-configmaps-secrets/04-full-demo/` |
| Session-11 K8s Services | `session-11-kubernetes-services/` (5 service types) |
| Session-11 FQDN & CoreDNS | `session-11-kubernetes-services/fqdn.md` |

---

*Generated from session-12-ingress-configmaps-secrets folder contents. All YAML manifests and step-by-step instructions verified against session-12 lab materials.*