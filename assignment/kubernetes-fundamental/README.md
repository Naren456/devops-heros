# Kubernetes Fundamentals Assignment

## Student Details

- **Name:** Naren456
- **Enrollment Number:** 24bcs10225
- **GitHub:** narendrase666@gmail.com
- **Repository:** devops-heros / `assignment/kubernetes-fundamental/`

---

## Overview

This assignment covers the core building blocks of Kubernetes and the fundamentals required to deploy, scale, expose, and update containerized applications in a cluster.

The focus is on understanding how Kubernetes manages workloads using the main objects:

- Pods
- ReplicaSets
- Deployments
- Services
- Namespaces
- Rollout strategies

---

## Objective

The goal of this assignment is to understand:

1. What a Pod is and how it differs from a container.
2. How ReplicaSets maintain the desired number of replicas.
3. How Deployments manage application updates and scale.
4. How Services enable reliable communication between workloads.
5. How Kubernetes ensures availability and rescheduling.
6. How different deployment strategies work in real-world production scenarios.

---

## Core Kubernetes Concepts

### 1. Pod
A Pod is the smallest deployable unit in Kubernetes. It can run one or more containers that share the same network namespace and storage.

Example:

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

### 2. ReplicaSet
A ReplicaSet ensures that the desired number of Pod replicas are always running.

Example:

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
```

### 3. Deployment
A Deployment manages ReplicaSets and provides declarative updates, rollback, scaling, and rollout control.

Example:

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

### 4. Service
A Service provides a stable network endpoint for accessing Pods, even when Pods are recreated or rescheduled.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

### 5. Namespace
Namespaces allow you to divide a cluster into logical groups for applications, teams, or environments.

Example:

```bash
kubectl create namespace dev
kubectl get ns
```

---

## Common Kubernetes Commands

```bash
kubectl get pods
kubectl get svc
kubectl get deploy
kubectl get rs
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl apply -f filename.yaml
kubectl delete -f filename.yaml
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>
```

---

## Assignment Tasks

### Task 1: Create and Run a Pod

Create a simple Pod manifest and deploy it in the cluster.

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod hello-pod
```

Verify the Pod is running and reachable.

### Task 2: Create a ReplicaSet

Deploy a ReplicaSet with multiple replicas and confirm that Kubernetes maintains the desired count.

```bash
kubectl apply -f replicaset.yaml
kubectl get rs
kubectl get pods
```

### Task 3: Create a Deployment

Deploy an application using a Deployment and scale it to multiple replicas.

```bash
kubectl apply -f deployment.yaml
kubectl get deployment
kubectl scale deployment web-deployment --replicas=5
kubectl get pods
```

### Task 4: Expose the Application with a Service

Create a Service to expose the Deployment internally.

```bash
kubectl apply -f service.yaml
kubectl get svc
kubectl get endpoints
```

This demonstrates how Kubernetes provides stable connectivity even when the Pod IP changes.

### Task 5: Upgrade and Rollout Strategies

Kubernetes supports several rollout strategies:

- Rolling Update
- Recreate
- Blue-Green
- Canary

These strategies are used to minimize downtime and reduce risk during releases.

#### Rolling Update
This is the default strategy for Deployments. Kubernetes replaces Pods gradually to avoid downtime.

#### Recreate
All old Pods are terminated before new Pods are created. This may cause downtime but is simple.

#### Blue-Green
Two environments run in parallel. Traffic is switched from the old version to the new version once validation is complete.

#### Canary
A small number of Pods for the new version are deployed first, and traffic gradually shifts to them.

---

## Deployment Strategies Comparison

| Strategy | Description | Advantage | Trade-off |
| --- | --- | --- | --- |
| Rolling Update | Gradual replacement of Pods | Zero or minimal downtime | Slower rollout |
| Recreate | Terminate old version before new one | Simple approach | Downtime possible |
| Blue-Green | Run old and new stacks side by side | Fast rollback and switch | Requires extra resources |
| Canary | Release to a subset first | Reduced risk | More complex setup |

---

## Example of Rollback

```bash
kubectl rollout history deployment/web-deployment
kubectl rollout undo deployment/web-deployment
kubectl get pods
```

This command helps restore the previous stable version of the application if a new release fails.

---

## Why Kubernetes Is Important

Kubernetes provides:

- Self-healing when Pods fail
- Auto-restart and rescheduling
- Horizontal scaling
- Service discovery and load balancing
- Declarative infrastructure management
- Automated rolling updates and rollback

This makes it the standard platform for modern microservices and cloud-native applications.

---

## Key Takeaways

- Pods are the basic execution unit.
- ReplicaSets maintain desired capacity.
- Deployments are the recommended way to manage app releases.
- Services provide stable communication endpoints.
- Kubernetes manages health, recovery, and scaling automatically.
- Rollout strategies help reduce risk during updates.

---

## Final Checklist

- [x] Understand Pod architecture
- [x] Understand ReplicaSet behavior
- [x] Understand Deployment management
- [x] Understand Service networking
- [x] Learn rollout and update strategies
- [x] Practice Kubernetes commands
- [x] Document the assignment in Markdown

---

## References

- Kubernetes official documentation
- DevOps course notes and lab material
- Open-source Kubernetes examples and YAML manifests
- `session10-k8s-core-objects/` lab exercises

This assignment forms the foundation for Kubernetes workloads, networking, and production deployment concepts that are used in real DevOps and cloud environments.
