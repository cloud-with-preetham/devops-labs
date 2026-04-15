# Day 84: GitOps with ArgoCD & Helm on Amazon EKS

> **90 Days of DevOps Challenge**
> Day 84 focuses on implementing a production-style GitOps workflow using **ArgoCD**, **Helm**, and **Amazon EKS**. The application is deployed directly from a Git repository, enabling automated synchronization, continuous reconciliation, and self-healing.

---

## Project Overview

GitOps is a modern approach to Kubernetes application delivery where **Git serves as the single source of truth**. Instead of manually applying Kubernetes manifests, ArgoCD continuously monitors a Git repository and automatically reconciles the Kubernetes cluster whenever changes occur.

In this project, I deployed the **AI BankApp** to an **Amazon EKS** cluster using **Helm** and managed the entire deployment lifecycle through **ArgoCD**.

---

## Architecture

```text
                    +----------------------+
                    |   GitHub Repository  |
                    |  (Source of Truth)   |
                    +----------+-----------+
                               |
                               |
                        Git Repository
                               |
                               ▼
                    +----------------------+
                    |       ArgoCD         |
                    | Watches Git Changes  |
                    +----------+-----------+
                               |
                     Automated Synchronization
                               |
                               ▼
                    +----------------------+
                    |     Amazon EKS       |
                    +----------+-----------+
                               |
         +---------------------+----------------------+
         |                     |                      |
         ▼                     ▼                      ▼
   AI BankApp             MySQL Pod             Ollama Pod
```

---

# Technologies Used

| Category                | Technology       |
| ----------------------- | ---------------- |
| Cloud                   | AWS EKS          |
| Container Orchestration | Kubernetes       |
| GitOps                  | ArgoCD           |
| Package Manager         | Helm             |
| Version Control         | Git & GitHub     |
| Storage                 | Amazon EBS CSI   |
| CLI Tools               | kubectl, AWS CLI |

---

# Project Structure

```text
AI-BankApp-DevOps/
│
├── argocd/
│   └── application.yml
│
├── helm-chart/
│   └── bankapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-dev.yaml
│       ├── values-prod.yaml
│       ├── values-staging.yaml
│       └── templates/
│
├── terraform/
│
└── README.md
```

---

# Prerequisites

Before starting, ensure you have:

- AWS Account
- Amazon EKS Cluster
- kubectl
- AWS CLI
- Helm
- ArgoCD Installed
- GitHub Repository
- IAM Permissions

---

# Step 1 — Configure kubectl

Update kubeconfig for the EKS cluster.

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name bankapp-eks
```

Verify connectivity.

```bash
kubectl get nodes
```

Expected Output

```text
Ready
Ready
Ready
```

---

# Step 2 — Verify ArgoCD

Check all ArgoCD components.

```bash
kubectl get pods -n argocd
```

Expected pods:

- argocd-server
- argocd-repo-server
- argocd-application-controller
- argocd-dex-server
- argocd-redis
- argocd-notifications-controller

---

# Step 3 — Create ArgoCD Application

Application Manifest

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bankapp
  namespace: argocd
```

Repository

```yaml
repoURL: https://github.com/cloudwithpreetham/AI-BankApp-DevOps.git
targetRevision: feat/gitops
path: helm-chart/bankapp
```

Destination

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: bankapp
```

Sync Policy

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

Deploy

```bash
kubectl apply -f argocd/application.yml
```

---

# Step 4 — Monitor Synchronization

Check application status.

```bash
kubectl get applications -n argocd
```

Final Result

```text
NAME      SYNC STATUS   HEALTH STATUS
bankapp   Synced        Healthy
```

---

# Step 5 — Verify Kubernetes Resources

## Pods

```bash
kubectl get pods -n bankapp
```

```text
bankapp
bankapp-mysql
bankapp-ollama
```

All pods are **Running**.

---

## Deployments

```bash
kubectl get deployments -n bankapp
```

```text
bankapp
bankapp-mysql
bankapp-ollama
```

All deployments are **Available**.

---

## Services

```bash
kubectl get svc -n bankapp
```

Created services:

- bankapp-service
- bankapp-mysql
- bankapp-ollama

---

## Persistent Volume Claims

```bash
kubectl get pvc -n bankapp
```

```text
Bound
Bound
```

Amazon EBS volumes were provisioned dynamically.

---

# Step 6 — Demonstrate ArgoCD Self-Healing

Check current replicas.

```bash
kubectl get deployment bankapp -n bankapp
```

Scale the deployment manually.

```bash
kubectl scale deployment bankapp \
-n bankapp \
--replicas=3
```

ArgoCD detected the configuration drift and automatically restored the deployment to the desired state defined in Git.

Verify:

```bash
kubectl get deployment bankapp -n bankapp
```

Final Output

```text
READY   1/1
```

This confirms that **ArgoCD Self-Healing** is functioning correctly.

---

# GitOps Workflow

```text
Developer
     │
git push
     │
     ▼
GitHub Repository
     │
     ▼
ArgoCD
     │
Continuously Watches Repository
     │
Detects Drift
     │
Automatic Sync
     │
     ▼
Amazon EKS
     │
Deploy Application
```

---

# Resources Created

| Resource               | Count |
| ---------------------- | ----: |
| Namespace              |     1 |
| Deployments            |     3 |
| Services               |     3 |
| ConfigMaps             |     1 |
| Secrets                |     1 |
| PersistentVolumeClaims |     2 |
| PreSync Hook Job       |     1 |

---

# Features Demonstrated

- GitOps Workflow
- Amazon EKS
- ArgoCD
- Helm Deployment
- Automated Sync
- Self-Healing
- Namespace Auto Creation
- Helm Values
- Git as Source of Truth
- Continuous Reconciliation
- Production GitOps Practices

---

# Commands Used

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name bankapp-eks

kubectl get nodes

kubectl get pods -n argocd

kubectl apply -f argocd/application.yml

kubectl get applications -n argocd

kubectl describe application bankapp -n argocd

kubectl get pods -n bankapp

kubectl get deployments -n bankapp

kubectl get svc -n bankapp

kubectl get pvc -n bankapp

kubectl scale deployment bankapp \
-n bankapp \
--replicas=3

kubectl get deployment bankapp -n bankapp
```

---

# Screenshots

Create the following directory:

```text
screenshots/day-84/
```

Capture the following screenshots:

| Screenshot                       | Description                                    |
| -------------------------------- | ---------------------------------------------- |
| 01-argocd-dashboard.png          | ArgoCD dashboard                               |
| 02-application-resource-tree.png | ArgoCD application resource tree               |
| 03-self-healing-before.png       | Manual scaling to 3 replicas                   |
| 04-self-healing-after.png        | Deployment automatically restored to 1 replica |
| 05-running-pods.png              | Running application pods                       |

---

# Key Learnings

- Git becomes the single source of truth for Kubernetes deployments.
- ArgoCD continuously reconciles cluster state with Git.
- Automated synchronization removes manual deployment steps.
- Self-healing automatically fixes configuration drift.
- Helm simplifies Kubernetes application packaging and deployment.
- GitOps improves reliability, consistency, and auditability.

---

# Outcome

Successfully implemented a complete **GitOps pipeline** using **ArgoCD**, **Helm**, and **Amazon EKS**. Verified automated synchronization, namespace creation, Helm-based deployments, persistent storage provisioning, and ArgoCD self-healing by restoring the application to its desired state after an intentional configuration drift.

---

## Connect With Me

**GitHub:** https://github.com/cloudwithpreetham

**LinkedIn:** https://www.linkedin.com/in/preetham-pereira

---

## If you found this project helpful

Give this repository a ⭐ and follow my **90 Days of DevOps** journey!
