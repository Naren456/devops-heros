# Secret — Protecting Sensitive Credentials in Kubernetes

> **Session-12 Reference:** `session-12-ingress-configmaps-secrets/02-secret` + `02-secret/README.md`

---

## 1. Why Do We Need Secrets?

### Problem: Passwords Stored in Plain Text or Baked into Images
Teams with poor security practices often commit credentials directly into code or Docker images:

```python
# BAD: Password baked into code and leaked in git history
DB_PASSWORD = "superSecretPwd123"
```

Or worse, stored in a ConfigMap in plain text, visible to every developer with `kubectl get` access.

**Real-world breach (2021):** A team hardcoded an AWS key into a `Dockerfile` committed to a public GitHub repository. The key was found by a bot within 7 minutes, leading to thousands of dollars of cloud resource abuse.

### Solution: Kubernetes Secret
A `Secret` is a dedicated Kubernetes object for sensitive data. Values are stored as **Base64-encoded strings**.

```
Raw value:    secretpassword
Base64 value: c2VjcmV0cGFzc3dvcmQ=

echo -n "secretpassword" | base64
# -> c2VjcmV0cGFzc3dvcmQ=

echo -n "c2VjcmV0cGFzc3dvcmQ=" | base64 --decode
# -> secretpassword
```

**Important:** `echo -n` is critical — without it, a trailing newline gets encoded, causing silent authentication failures.

---

## 2. Important Points

| Point | Detail |
|---|---|
| **Base64 is NOT encryption** | It is encoding. Anyone with `kubectl get secret` RBAC access can decode it. Secrets rely on Kubernetes RBAC for access control. |
| **Production-grade secret management** | Use external secret managers (AWS Secrets Manager, HashiCorp Vault, GCP Secret Manager) integrated via `External Secrets Operator`. |
| **Always use `echo -n`** | When base64-encoding values. Without `-n`, a trailing newline character gets encoded. |
| **Injection methods** | Environment variables (`secretRef`) or mounted as files into a volume (e.g., TLS `tls.key` files). |
| **Storage** | Secrets are stored in `etcd`. Enable **encryption at rest** for `etcd` in production (mandatory CIS Benchmark requirement). |
| **Visibility** | `kubectl describe secret` masks all values with `[X bytes]` to prevent accidental exposure. |

---

## 3. Real-World Use Cases

- Database credentials (`POSTGRES_USER`, `POSTGRES_PASSWORD`)
- API keys and OAuth client secrets for third-party integrations
- TLS certificate (`tls.crt`) and private key (`tls.key`) for HTTPS termination
- Docker Hub / ECR image pull credentials (`imagePullSecrets`)

---

## 4. YAML Manifest (Session-12 Lab)

### Secret (`secret.yaml`)
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
  # Base64 for 'yatri_admin' -> echo -n "yatri_admin" | base64
  POSTGRES_USER: eWF0cmlfYWRtaW4=
  # Base64 for 'secretpassword' -> echo -n "secretpassword" | base64
  POSTGRES_PASSWORD: c2VjcmV0cGFzc3dvcmQ=
  # Base64 for 'yatri_production_db' -> echo -n "yatri_production_db" | base64
  POSTGRES_DB: eWF0cmlfcHJvZHVjdGlvbl9kYg==
```

### Generating Base64 Values Yourself
```bash
echo -n "yatri_admin" | base64
# Output: eWF0cmlfYWRtaW4=

echo -n "secretpassword" | base64
# Output: c2VjcmV0cGFzc3dvcmQ=

echo -n "yatri_production_db" | base64
# Output: eWF0cmlfcHJvZHVjdGlvbl9kYg==
```

### Applying and Inspecting
```bash
kubectl apply -f secret.yaml
kubectl get secret yatri-db-secret
# Expected:
# NAME               TYPE     DATA   AGE
# yatri-db-secret    Opaque   3      5s

# Notice: kubectl describe secret masks values with [3 bytes]
kubectl get secret yatri-db-secret -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 --decode
# Output: secretpassword
```

---

## 5. How It Works Under the Hood

- Secret is stored in `etcd` as Base64-encoded key-value pairs
- When a Pod requests `secretRef` via `envFrom` or `volumeMounts`, the Kubelet injects the decoded values at container startup
- `kubectl describe secret` uses a special formatter that replaces all data values with `[X bytes]` for safety
- For production, always enable `etcd encryption at rest` and consider integrating with external secret managers (Vault, AWS Secrets Manager, GCP Secret Manager)

---

## 6. Secret vs ConfigMap (Interview Favorite)

| Aspect | ConfigMap | Secret |
|---|---|---|
| **Data type** | Plain-text key-value | Base64-encoded strings |
| **Sensitivity** | Non-sensitive | Sensitive (passwords, tokens, TLS) |
| **Visibility** | Visible via `kubectl get/config describe` | Masked via `kubectl describe`; RBAC-protected |
| **Size limit** | 1 MiB | 1 MiB (per Secret) |
| **Use case** | Log levels, ports, feature flags | DB credentials, API keys, TLS private keys |
| **Auto-restart on update** | No | No |

> **Exam Tip:** If the data could be read from source code or Docker image → ConfigMap. If the data is a password/key/secret → Secret.

---

## 7. Cleanup

```bash
kubectl delete secret yatri-db-secret
```

---

## 8. Key Interview Points

- **Why not store passwords in ConfigMap?** ConfigMaps are plain-text and visible to any user with `kubectl` read access. Secrets use Base64 encoding (not encryption) and rely on Kubernetes RBAC + etcd encryption-at-rest for protection.
- **What is the Secret size limit?** 1 MiB.
- **Does updating Secret auto-restart pods?** No — same as ConfigMap; pod restart required.
- **Is Base64 encryption?** No — it's just encoding. Anyone who can run `kubectl get secret` and has RBAC access can decode it.

**Reference:** `session-12-ingress-configmaps-secrets/02-secret/db-secret.yaml` + `02-secret/README.md`