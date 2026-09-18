# Kubernetes Pods, ReplicaSets, and Deployments Assignment

## Student Details

- **Name:** Naren456
- **Enrollment Number:** 24bcs10225

---

## Overview

This assignment focuses on the core Kubernetes workload objects used to run applications in a cluster:

- Pods
- ReplicaSets
- Deployments

Each object plays a distinct role in application lifecycle management, scaling, and availability.

---

## 1. Pod

A Pod is the smallest deployable unit in Kubernetes. It represents one or more containers that share the same networking and storage namespace.

### Key Points

- Pods are ephemeral
- Pods can run one or more containers
- Each Pod gets its own IP address
- Pods are usually managed indirectly through higher-level controllers

### Example Pod Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
spec:
  containers:
    - name: hello-container
      image: nginx
      ports:
        - containerPort: 80
```

### Commands

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod hello-pod
kubectl logs hello-pod
```

---

## 2. ReplicaSet

A ReplicaSet ensures that a specified number of Pod replicas are always running.

### Key Points

- Maintains desired number of replicas
- Automatically creates or removes pods to match the target count
- Uses labels and selectors to identify matching pods

### Example ReplicaSet Manifest

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
spec:
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
        - name: web
          image: nginx
          ports:
            - containerPort: 80
```

### Commands

```bash
kubectl apply -f replicaset.yaml
kubectl get rs
kubectl get pods
kubectl delete pod <pod-name>
```

ReplicaSets recreate Pods automatically if they are removed unexpectedly.

---

## 3. Deployment

A Deployment is the recommended Kubernetes object for managing application updates and scaling.

### Key Points

- Manages ReplicaSets
- Provides declarative updates
- Supports rollout and rollback
- Helps with scaling and zero-downtime updates

### Example Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
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
        - name: web
          image: nginx:latest
          ports:
            - containerPort: 80
```

### Commands

```bash
kubectl apply -f deployment.yaml
kubectl get deploy
kubectl get pods
kubectl rollout status deployment/web-deployment
kubectl scale deployment web-deployment --replicas=5
```

---

## Comparison

| Object | Purpose | Main Feature |
| --- | --- | --- |
| Pod | Smallest unit | Runs one or more containers |
| ReplicaSet | Keeps a fixed replica count | Self-healing and scaling |
| Deployment | Manages application lifecycle | Rollout, rollback, scaling |

---

## Why These Objects Matter

These resources help Kubernetes maintain application availability and resilience. A production system usually does not run directly from Pods alone; instead, Deployments manage the application lifecycle while ReplicaSets handle pod counts and health.

---

## Common Kubernetes Commands

```bash
kubectl get pods
kubectl get rs
kubectl get deploy
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl apply -f <file>.yaml
kubectl delete -f <file>.yaml
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>
```

---

## Deployment Strategies

Kubernetes supports multiple deployment strategies:

- Rolling Update: replaces pods gradually
- Recreate: deletes old pods before creating new ones
- Blue-Green: runs two versions side by side and switches traffic
- Canary: releases to a subset of users first

These strategies reduce risk and help manage application updates safely.

---

## Key Takeaways

- Pods are the runtime unit for containers.
- ReplicaSets ensure the desired number of copies remain available.
- Deployments are the standard way to manage applications in Kubernetes.
- Kubernetes automatically heals and scales workloads.

---

## Final Checklist

- [x] Understood the role of Pods
- [x] Created or reviewed ReplicaSet examples
- [x] Learned Deployment behavior and scaling
- [x] Practiced essential kubectl commands
- [x] Documented the core Kubernetes workload concepts

---

## References

- Kubernetes official documentation
- Course notes on Pods, ReplicaSets, and Deployments
- DevOps lab exercises and YAML examples
