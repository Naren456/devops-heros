# Kubernetes Services — Complete 5-Service Guide

> **Reference:** `session-11-kubernetes-services/service.md` + `session-11-kubernetes-services/fqdn.md`  
> **Total Types:** 5 (ClusterIP, NodePort, LoadBalancer, ExternalName, Headless)

---

## 📋 Table of Contents

1. [ClusterIP](#1-clusterip-internal-communication)
2. [NodePort](#2-nodeport-external-host-access)
3. [LoadBalancer](#3-loadbalancer-public-cloud-exposure)
4. [ExternalName](#4-externalname-dns-alias-for-external-systems)
5. [Headless](#5-headless-direct-pod-discovery)
6. [Decision Tree](#6-decision-tree-which-service-should-you-pick)
7. [Quick Summary Checklist](#7-quick-summary-checklist)

---

## 1. ClusterIP — Internal Microservice Communication

- **Type:** Default (if type omitted)
- **Access:** Internal only — reachable from within the cluster only
- **VIP:** Stable virtual IP from service CIDR (e.g., `10.96.0.0/12`)
- **Use Cases:** Inter-service communication, internal databases/caches, behind Ingress Controllers
- **DNS:** `<service>.<namespace>.svc.cluster.local` → Single A record → ClusterIP
- **YAML:** `type: ClusterIP` (or omit for default)
- **Session-11:** `01-clusterip/` + `session-11/01-clusterip/README.md`

> **Image:** `assignment/kubernetes-service/clusterip/image.png` — ClusterIP architecture diagram

---

## 2. NodePort — External Host-Level Access

- **Type:** `NodePort`
- **Access:** Every worker node IP + high port `30000–32767`
- **Port Range:** `30000–32767` (default); can customize via `--service-node-port-range`
- **Use Cases:** Bare-metal/on-prem clusters, Minikube local dev, Ingress Controller entrypoint, non-HTTP TCP/UDP services
- **YAML:** `type: NodePort` + optional `nodePort: 30080`
- **Session-11:** `02-nodeport/` + `session-11/02-nodeport/README.md`

> **Image:** `assignment/kubernetes-service/nodeport/image.png` — NodePort on every node diagram

---

## 3. LoadBalancer — Public Cloud Exposure

- **Type:** `LoadBalancer`
- **Access:** Cloud-provisioned external IP + standard ports `80`/`443`
- **Cloud Cost:** `$15–$30/mo` per LB; **best practice: 1 LB + Ingress Controller** for all microservices
- **Underlying:** Creates ClusterIP + NodePort + Cloud LB (Russian doll pattern)
- **Use Cases:** Public web apps, public APIs, non-HTTP L4 (gaming, VoIP), Ingress entrypoint
- **YAML:** `type: LoadBalancer`
- **Session-11:** `03-loadbalancer/` + `session-11/03-loadbalancer/README.md`

> **Image:** `assignment/kubernetes-service/loadbalancer/image.png` — LoadBalancer architecture diagram

---

## 4. ExternalName — Internal DNS Alias for External Systems

- **Type:** `ExternalName`
- **Access:** CoreDNS `CNAME` redirect — **no ClusterIP, no pods, no kube-proxy**
- **Mechanism:** `externalName: api.github.com` → CoreDNS returns `CNAME → actual FQDN`
- **Use Cases:** AWS RDS, GCP CloudSQL, MongoDB Atlas, Stripe API, Twilio, on-prem legacy VM migration
- **YAML:** `type: ExternalName` + `externalName: <hostname>` (NO selector, ports, or clusterIP)
- **Session-11:** `04-externalname/` + `session-11/04-externalname/README.md`

> **Image:** `assignment/kubernetes-service/externalname/image.png` — ExternalName CNAME flow diagram

---

## 5. Headless — Direct Pod Discovery (`clusterIP: None`)

- **Type:** `ClusterIP` with `clusterIP: None`
- **Access:** No VIP; CoreDNS returns **multiple A records** = all healthy pod IPs
- **Per-Pod DNS:** With StatefulSet: `<pod-name>.<service>.<namespace>.svc.cluster.local`
- **Use Cases:** Kafka, ZooKeeper, Cassandra, MongoDB replica sets, Redis Cluster, PostgreSQL master-slave, gRPC client-side LB
- **YAML:** `spec: clusterIP: None` + `statefulServiceName` in StatefulSet
- **Session-11:** `05-headless/` + `session-11/05-headless/README.md`

> **Image:** `assignment/kubernetes-service/headless/image.png` — Headless Service DNS resolution diagram

---

## 6. Decision Tree — Which Service Should You Pick?

```
Need external access?
├── NO ──► Need direct pod-to-pod / StatefulSet clustering?
│          ├── YES → HEADLESS (clusterIP: None)
│          └── NO  → CLUSTERIP (default)
└── YES ─► Is target outside cluster (RDS FQDN / SaaS)?
           ├── YES → EXTERNALNAME (CNAME)
           └── NO  ─► On Public Cloud?
                     ├── YES ─► HTTP/HTTPS? → 1× LOADBALANCER for Ingress + ClusterIP apps
                     │         └─ No (TCP/UDP) → direct LOADBALANCER
                     └── NO (Bare-Metal / Local) → NODEPORT
```

---

## 7. Quick Summary Checklist for Interviews

| Service Type | Default? | External Access | Port Range | Cloud Needed | Key Use Case |
|---|---|---|---|---|---|
| **ClusterIP** | ✅ Yes | ❌ Internal only | Any | No | Microservice↔DB, internal APIs |
| **NodePort** | ❌ No | ✅ NodeIP:NodePort | 30000–32767 | No | Bare-metal, Minikube, Ingress entrypoint |
| **LoadBalancer** | ❌ No | ✅ Cloud LB IP | 80/443 | Yes | Public web apps, 1× LB + Ingress pattern |
| **ExternalName** | ❌ No | ❌ DNS CNAME only | None | No | RDS, SaaS APIs, external DB aliases |
| **Headless** | ❌ No | ❌ No VIP | Any | No | StatefulSets: Kafka, Mongo, Redis, etcd |

**Key Interview Q&As:**
- **Default service type?** → `ClusterIP`
- **Difference between NodePort & LoadBalancer?** → NodePort = host port 30000-32767; LoadBalancer = cloud-provisioned public IP on 80/443
- **Why use Headless Service?** → Statefull distributed systems need direct pod-to-peer communication (leader election, quorum, peer discovery)
- **ExternalName vs Service without selector?** → ExternalName = DNS CNAME only; no-selector + manual Endpoints = kube-proxy iptables DNAT to specific IPs
- **One LoadBalancer vs 50 LoadBalancers?** → Cost: $15–30/mo each. Best practice: 1× Ingress Controller via LoadBalancer + all apps as ClusterIP services behind Ingress rules.

---

## 📁 Folder Structure

```
assignment/kubernetes-service/
├── clusterip/
│   ├── README.md        (created)
│   ├── image.png
│   └── image copy.png
├── nodeport/
│   ├── README.md        (created)
│   ├── image.png
│   └── image copy.png
├── loadbalancer/
│   ├── README.md        (created)
│   ├── image.png
│   └── (no image copy)
├── externalname/
│   ├── README.md        (created)
│   └── image.png
└── headless/
    ├── README.md        (created)
    └── image.png
```

**Total READMEs created:** 5 (one per service type) + 1 master index

---

**References:**
- `session-11-kubernetes-services/service.md` — Complete service type comparison (439 lines)
- `session-11-kubernetes-services/fqdn.md` — CoreDNS, FQDN, `/etc/resolv.conf`, `ndots:5`
- `session-12-ingress-configmaps-secrets/` — ConfigMap, Secret, Ingress full demo