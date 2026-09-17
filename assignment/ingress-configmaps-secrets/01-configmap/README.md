# ConfigMap — Decoupling Plain-Text Configuration from Container Images

> **Session-12 Reference:** `session-12-ingress-configmaps-secrets/01-configmap` + `01-configmap/README.md`

---

## 1. Why Do We Need ConfigMap?

### Problem: Configuration Baked Into Docker Images
Imagine a Python app with hardcoded values:
```python
LOG_LEVEL = "DEBUG"
PORT = 5000
DATABASE_HOST = "localhost"
```
If configuration is inside the Docker image, every config change requires rebuilding and re-pushing the image. This violates the **12-Factor App principle**:

> **Your code must be identical across all environments. Only the configuration changes.**

### Solution: ConfigMap
A `ConfigMap` stores plain-text key-value pairs **outside** the container image. The application reads them at runtime as environment variables or volume-mounted files.

```
WITHOUT ConfigMap:                    WITH ConfigMap:
-------------------                 --------------------
Image: v1 (LOG=DEBUG)                 Image: v1 (no hardcoded config)
Image: v2 (LOG=INFO)                        |
Image: v3 (PORT=8080)            ConfigMap: LOG=INFO, PORT=5000
                                        |
                                Same image deployed everywhere!
```

---

## 2. Important Points

| Point | Detail |
|---|---|
| **Use case** | Non-sensitive data only: log levels, port numbers, feature flags, API base URLs |
| **Never store** | Passwords, tokens, certificates in a ConfigMap |
| **Consumption** | `envFrom` (environment variables) or mounted as files inside a container |
| **Update behavior** | Updating a ConfigMap does **NOT** automatically restart pods. Pods must be restarted to pick up new values (unless using a volume mount with live reload) |
| **Size limit** | **1 MiB** maximum |

---

## 3. Real-World Use Cases

- Storing `LOG_LEVEL`, `ENVIRONMENT` (dev/staging/prod), `CACHE_TTL`, `MAX_CONNECTIONS`
- Mounting an entire Nginx `nginx.conf` into a pod via ConfigMap volume
- Passing feature-flag toggles (`FEATURE_DARK_MODE: "true"`) without rebuilding images

---

## 4. YAML Manifest (Session-12 Lab)

### ConfigMap (`configmap.yaml`)
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

### Applying and Inspecting
```bash
kubectl apply -f configmap.yaml
kubectl get configmap yatri-app-config
kubectl describe configmap yatri-app-config
```

Expected Output:
```
NAME               DATA   AGE
yatri-app-config   5      8s

Name:         yatri-app-config
Data
====
DEFAULT_CURRENCY:  INR
ENVIRONMENT:       production
LOG_LEVEL:         INFO
MAX_BOOKING_DAYS:  30
PORT:            5000
```

### Reading a Value Live
```bash
kubectl get configmap yatri-app-config -o jsonpath='{.data.LOG_LEVEL}'
```
Output: `INFO`

---

## 5. How It Works Under the Hood

- ConfigMaps are stored in `etcd` alongside other Kubernetes objects
- The Kubernetes API server validates and stores the key-value pairs
- When a Pod requests `configMapRef`, the Kubelet injects the values as env vars or mounts them as files into the container's filesystem at startup
- Changes to an existing ConfigMap require pod restart (or use volume mount with `subPath` + `readOnly: false` for live reload)

---

## 6. ConfigMap vs Secret (Interview Favorite)

| Aspect | ConfigMap | Secret |
|---|---|---|
| **Data type** | Plain-text key-value | Base64-encoded strings |
| **Sensitivity** | Non-sensitive (log levels, ports) | Sensitive (passwords, tokens, TLS certs) |
| **Visibility** | Visible via `kubectl get/config describe` to any RBAC user | Masked via `kubectl describe`; relies on RBAC + etcd encryption-at-rest |
| **Size** | Up to 1 MiB | Up to 1 MiB (per Secret) |
| **Use case** | App config, feature flags | DB credentials, API keys, TLS private keys |

> **Exam Tip:** If the data could be read from source code or Docker image → use ConfigMap. If the data is a password/key/secret → use Secret.

---

## 7. Cleanup

```bash
kubectl delete configmap yatri-app-config
```

---

## 8. Key Interview Points

- **Why not store passwords in ConfigMap?** ConfigMaps are plain-text and visible to any user with `kubectl` read access. Secrets use Base64 encoding (not encryption) and rely on Kubernetes RBAC + etcd encryption-at-rest for protection.
- **What is the ConfigMap size limit?** 1 MiB.
- **Does updating ConfigMap auto-restart pods?** No — pods must be restarted unless using a volume mount with live reload.

**Reference:** `session-12-ingress-configmaps-secrets/01-configmap/app-config.yaml` + `01-configmap/README.md`