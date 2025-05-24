# DevOps Assignment - EKS-Style Deployment on Minikube

## 📌 Overview
This project simulates a secure AWS EKS-style microservices deployment using Minikube. It includes infrastructure provisioning, secure networking, IAM simulation, observability, and incident response mechanisms.

## 🛠 Environment Setup
### Prerequisites
- Minikube
- Docker
- kubectl
- Helm
- Optional: k9s, OPA, Kyverno, Falco

### Cluster Setup
```bash
minikube start --cpus=2 --memory=4096 --addons=ingress,metrics-server
```

### Install NGINX Ingress, Prometheus & Grafana
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack
```

## 📦 Microservices Architecture
- **gateway** (`nginxdemos/hello`): Public API gateway, exposed via Ingress
- **auth-service** (`kennethreitz/httpbin`): Internal-only service for auth
- **data-service** (`hashicorp/http-echo`): Internal-only mock business logic

## 📁 Namespaces
- `system`: Monitoring & infra
- `app`: Application workloads

## ⚙️ Deployment Strategy
- Deployed with YAML (can be converted to Helm/Kustomize)
- Resource limits and health probes added
- Ingress configured for gateway only
- Services separated by namespaces

## 🔐 IAM Simulation with MinIO
- MinIO deployed as mock S3
- Secret used for credentials
- `data-service` given a ServiceAccount with access
- `auth-service` is denied access, even if misconfigured
- Kyverno policy enforces this isolation

## 🚨 Security Incident Simulation
### Scenario
- `auth-service` leaks `Authorization` headers to logs
- Also has unintended access to external services

### Investigation
- Used `kubectl logs` to identify log leaks via `/headers` endpoint

### Fixes
- Filtered headers in deployment config
- Applied NetworkPolicy to:
  - Allow only `auth-service` → `data-service`
  - Block all egress traffic by default

### Prevention
- Kyverno policy created to detect and block Authorization logging

## 📊 Observability
- Prometheus + Grafana installed via Helm
- Grafana dashboards include:
  - Pod CPU/memory
  - Request rates and errors
  - Pod restarts

### Bonus
- Alert set for failed probes and pod restarts

## 🧪 Failure Simulation
- Pod manually deleted to simulate failure:
```bash
kubectl delete pod <pod-name>
```
- ReplicaSets restored services automatically
- Metrics captured in Grafana

## 📁 Folder Structure
Refer to the project root layout in this repo for detailed contents.

## 🧩 Assumptions & Known Issues
- Simulated IAM via ServiceAccounts and Kyverno
- Falco was not used due to time constraints
- External traffic blocked except where explicitly allowed

## 📤 Submission
- All files in repo or ZIP
- Include `README.md`, YAMLs, policies, dashboards
- Loom recording linked as walkthrough

## 🧱 Architecture Diagram
*(Included as `architecture-diagram.png`)*

```ascii
                  +-----------------+
                  |     Ingress     |
                  +--------+--------+
                           |
                      +----v----+
                      | Gateway  |
                      +----+----+
                           |
            +--------------+---------------+
            |                              |
       +----v----+                   +-----v-----+
       |  Auth    |-----> internal-->|   Data     |
       | Service  |<-----------------|  Service   |
       +---------+                   +-----------+
            |
       External Blocked
```
