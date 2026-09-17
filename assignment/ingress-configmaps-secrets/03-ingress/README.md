# Ingress — One Entry Point for All Your Microservices

> **Session-12 Reference:** `session-12-ingress-configmaps-secrets/03-ingress` + `03-ingress/README.md`

---

## 1. Why Do We Need Ingress?

### Problem: One Load Balancer Per Service = Expensive Chaos
Imagine an application with 5 microservices: Frontend, Backend API, Auth Service, Payment Service, and Admin Dashboard. Without Ingress, you need a separate `LoadBalancer` service for each one:

```text
Without Ingress:
  Frontend     -> AWS Load Balancer 1 ($25/month)
  Backend API  -> AWS Load Balancer 2 ($25/month)
  Auth         -> AWS Load Balancer 3 ($25/month)
  Payment      -> AWS Load Balancer 4 ($25/month)
  Admin        -> AWS Load Balancer 5 ($25/month)
  Total: $125/month just for load balancers
```

Users also get ugly non-standard ports and URLs like `http://3.15.22.100:30080`. There is no SSL/TLS termination, no centralized routing, and no ability to do host-based routing like `api.myapp.com` vs `myapp.com`.

### Solution: Ingress Controller + Ingress Rules
An **Ingress Controller** (e.g., NGINX Ingress Controller) is a single pod running a reverse proxy. You deploy **one** `LoadBalancer` Service pointing to it. All routing logic is declared as `Ingress` YAML rules.

```text
With Ingress:
  1 AWS Load Balancer ($25/month)
       |
   NGINX Ingress Controller
       |
       +-- yatri.local/         -> Frontend ClusterIP Service
       +-- yatri.local/api/*    -> Backend API ClusterIP Service
```

```
Public Internet
      |
      | https://yatri.local (Port 80/443)
      v
+------------------------------------------+
|   NGINX Ingress Controller Pod           |
|   (Layer 7 HTTP Reverse Proxy)           |
+------------------------------------------+
      |                        |
      | Path: /                | Path: /api/*
      v                        v
+----------------+     +---------------------+
| Frontend Svc   |     | Backend API Svc      |
| (ClusterIP)    |     | (ClusterIP)          |
+----------------+     +---------------------+
```

---

## 2. Important Points

| Point | Detail |
|---|---|
| **Ingress is just routing rules** | It does nothing on its own. MUST have an **Ingress Controller** installed (e.g., `ingress-nginx`). |
| **Layer 7 (HTTP/HTTPS)** | Can route based on hostnames (`api.shop.com`) and URL paths (`/orders`, `/users`). |
| **TLS/HTTPS termination** | Configure SSL certificate once in Ingress; all backends communicate over plain HTTP internally. |
| **Minikube** | Enable addon: `minikube addons enable ingress`. |
| **AWS EKS** | Install `ingress-nginx` via Helm; one AWS NLB is automatically provisioned. |

---

## 3. Real-World Use Cases

- Routing `app.company.com` to the frontend and `api.company.com` to the backend API — from a single public IP.
- SSL/TLS termination: attaching a certificate to `https://myapp.com` without modifying any application code.
- Canary deployments: routing 10% of `/api` traffic to a `v2` service and 90% to `v1`.
- Rate limiting, authentication headers, and CORS rules applied centrally via Ingress annotations.

---

## 4. YAML Manifest (Session-12 Lab)

### Ingress (`ingress-routes.yaml`)
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
spec:
  ingressClassName: nginx
  rules:
    - host: yatri.local
      http:
        paths:
          - path: /api(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: yatri-backend-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: yatri-frontend-service
                port:
                  number: 80
```

### Applying and Inspecting
```bash
kubectl apply -f ingress/ingress-routes.yaml
kubectl get ingress yatri-ingress
kubectl describe ingress yatri-ingress
```

Expected Output:
```
NAME            CLASS   HOSTS        ADDRESS        PORTS   AGE
yatri-ingress   nginx   yatri.local  192.168.49.2   80      12s
```

---

## 5. How It Works Under the Hood

1. **Ingress Controller** (e.g., NGINX) watches the Kubernetes API for `Ingress` resources.
2. When an Ingress is created/updated, the Controller updates its internal routing configuration (NGINX configmap, upstream services).
3. The single Cloud Load Balancer (provisioned by the Ingress Controller) routes traffic to the Ingress Controller pod.
4. The Ingress Controller evaluates the `rules.host` and `rules.http.paths` and forwards the request to the appropriate backend `ClusterIP` Service.
5. `kube-proxy` / `iptables` / `IPVS` routes from the Service VIP to the correct Pod.

> **Key:** Without an Ingress Controller installed, the `Ingress` resource is silently ignored.

---

## 6. Ingress vs Service Types (Comparison)

| Feature | ClusterIP | NodePort | LoadBalancer | Ingress |
|---|---|---|---|---|
| **Purpose** | Internal cluster only | Host-level external access | Cloud LB per service | Single entry point for all services |
| **Port range** | Any | 30000-32767 | 80/443 (cloud) | 80/443 (cloud) |
| **Cloud needed** | No | No | Yes | Yes |
| **L4 vs L7** | L4 (kube-proxy) | L4 (kube-proxy) | L4 (cloud LB) | **L7 (HTTP/HTTPS)** |
| **Host-based routing** | ❌ No | ❌ No | ❌ No | ✅ Yes |
| **Path-based routing** | ❌ No | ❌ No | ❌ No | ✅ Yes |
| **TLS termination** | ❌ No | ❌ No | ❌ No (or manual) | ✅ Yes (via annotations) |
| **Cost** | Free | Free | $15–30/mo per LB | $15–30/mo for **one** LB |

> **Best Practice:** Use **1× Ingress + 1× LoadBalancer** (for the Ingress Controller) + all apps as internal `ClusterIP` services. This avoids 50 separate LBs costing $1000+/month.

---

## 7. Key Interview Points

- **What is the difference between Ingress and LoadBalancer?** LoadBalancer provisions a cloud LB per service (costly). Ingress is a routing rule definition; one Ingress Controller + one LoadBalancer serves all apps via host/path routing.
- **Do you need an Ingress Controller?** Yes — Ingress is just a CRD; without an actual controller (e.g., NGINX, Traefik), the Ingress resource is ignored.
- **What is `ingressClassName`?** Modern K8s (v1.18+) requires specifying which Ingress Controller handles the resource. If omitted, the Ingress is silently ignored.
- **What is `rewrite-target: /$2`?** Strips the `/api` prefix from the request path before forwarding to the backend, so the backend receives `/orders` instead of `/api/orders`.
- **Can Ingress do TCP/UDP load balancing?** No — Ingress is L7 (HTTP/HTTPS) only. Use `Service type: LoadBalancer` with `protocol: TCP` for L4.

---

## 8. Cleanup

```bash
kubectl delete ingress yatri-ingress
```

---

## 9. Session-12 Demo Flow (Full Demo - Section 04)

The full demo in `session-12-ingress-configmaps-secrets/04-full-demo` combines all three concepts:

1. **ConfigMap** → Stores `ENVIRONMENT`, `LOG_LEVEL`, etc. injected into Frontend (as env vars) and Backend (as env vars).
2. **Secret** → Stores Base64-encoded `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` injected into Backend via `secretKeyRef`.
3. **Ingress** → Single NGINX Ingress routes `/` → Frontend ClusterIP and `/api/*` → Backend ClusterIP, both exposed via one public IP.

> **Total cost:** 1× LoadBalancer (for Ingress Controller) instead of 3× LoadBalancer (one per service).

**Reference:** `session-12-ingress-configmaps-secrets/03-ingress/ingress-routes.yaml` + `03-ingress/README.md`