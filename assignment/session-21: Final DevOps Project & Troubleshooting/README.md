# session-21: Final DevOps Project & Troubleshooting

## Overview
This folder contains work completed for the **session-21: Final DevOps Project & Troubleshooting** assignment.

## Learning Objectives
- Learn the required concepts for this session.
- Practice the task using command-line tools, infrastructure, or application setup.
- Validate the configuration or deployment output.
- Record key screenshots and important findings.

## Tasks Completed
- Assignment setup and configuration
- Hands-on lab execution
- Verification and troubleshooting
- Documentation of the outcome

## Screenshots

### 1. TaskBoard local stack verification
`docker compose config` validated - backend:8000, frontend:3000, postgres:5432

![terminal commands](terminal-commands.png)

### 2. Application screenshots

![taskboard screenshot 1](image.png)

![taskboard screenshot 2](<image copy.png>)

![taskboard screenshot 3](<image copy 2.png>)

## Commands Used
```bash
cd ~/Documents/devops-heros/session21-python
ls
cat docker-compose.yml
docker --version
docker compose config
ls backend/app helm/taskboard k8s troubleshooting
docker images
docker compose up --build
# verify after up:
# http://localhost:3000
# http://localhost:8000/docs
# http://localhost:8000/health
```

## Outcome
Session 21 TaskBoard stack understood and verified locally: React frontend + FastAPI backend + PostgreSQL 16 via `docker-compose.yml`. Compose file validated, images present, Helm/K8s manifests (`namespace.yaml`, `values-dev.yaml`, `broken-image.yaml`, `broken-service.yaml`) reviewed. Next: `docker compose up --build`, test `/health`/`/ready`/`/metrics`, then EKS + Helm + Prometheus/Grafana demo.
