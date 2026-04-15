# Day 84 – GitOps with ArgoCD on Amazon EKS

## Overview

Day 84 focused on implementing a complete **GitOps workflow** using **ArgoCD** and **Helm** on an **Amazon EKS** cluster. The application was deployed directly from a GitHub repository, demonstrating automated synchronization, namespace creation, Helm-based deployments, and ArgoCD's self-healing capabilities.

This lab simulated a production-ready GitOps workflow where Git acts as the single source of truth and ArgoCD continuously reconciles the Kubernetes cluster with the desired state stored in Git.

---

# Objectives

- Understand GitOps fundamentals
- Deploy ArgoCD on Amazon EKS
- Configure an ArgoCD Application
- Deploy a Helm chart using GitOps
- Enable automated synchronization
- Verify self-healing functionality
- Explore the ArgoCD Dashboard

---

# Architecture

```
                GitHub Repository
                       │
                (Source of Truth)
                       │
                       ▼
                  ArgoCD Server
                       │
             Watches Git Repository
                       │
        Detects Configuration Changes
                       │
                       ▼
                Amazon EKS Cluster
                       │
        Deploys Helm Release Automatically
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    AI BankApp       MySQL         Ollama
```

---

# Prerequisites

- AWS Account
- Amazon EKS Cluster
- kubectl
- AWS CLI
- Terraform
- Helm
- ArgoCD
- GitHub Repository

---

# Step 1 – Verify EKS Cluster

Configured kubectl to communicate with the EKS cluster.

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name bankapp-eks
```

Verify cluster connectivity:

```bash
kubectl get nodes
```

Output:

```text
NAME                                            STATUS   ROLES    AGE
ip-192-168-xx-xx.ap-south-1.compute.internal    Ready    <none>
ip-192-168-xx-xx.ap-south-1.compute.internal    Ready    <none>
ip-192-168-xx-xx.ap-south-1.compute.internal    Ready    <none>
```

---

# Step 2 – Verify ArgoCD Installation

Check ArgoCD components.

```bash
kubectl get pods -n argocd
```

Verified:

- argocd-server
- repo-server
- application-controller
- redis
- dex-server
- notifications-controller

All pods were in **Running** state.

---

# Step 3 – Create ArgoCD Application

Application manifest:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bankapp
  namespace: argocd
```

Repository configuration:

```yaml
repoURL: https://github.com/cloudwithpreetham/AI-BankApp-DevOps.git
targetRevision: feat/gitops
path: helm-chart/bankapp
```

Deployment destination:

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: bankapp
```

Automated Sync:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

---

# Step 4 – Deploy Application

Deploy ArgoCD Application.

```bash
kubectl apply -f argocd/application.yml
```

Verify:

```bash
kubectl get applications -n argocd
```

Initial status:

```text
OutOfSync
Missing
```

ArgoCD automatically started synchronization.

---

# Step 5 – Monitor Deployment

Application status eventually became:

```text
NAME      SYNC STATUS   HEALTH STATUS
bankapp   Synced        Healthy
```

---

# Step 6 – Verify Kubernetes Resources

Pods

```bash
kubectl get pods -n bankapp
```

Output:

```text
bankapp
bankapp-mysql
bankapp-ollama
```

All Running.

---

Deployments

```bash
kubectl get deployments -n bankapp
```

Output:

```text
bankapp
bankapp-mysql
bankapp-ollama
```

All Available.

---

Services

```bash
kubectl get svc -n bankapp
```

Created:

- bankapp-service
- bankapp-mysql
- bankapp-ollama

---

Persistent Volumes

```bash
kubectl get pvc -n bankapp
```

Output:

```text
Bound
Bound
```

Dynamic EBS volumes were successfully provisioned.

---

# Step 7 – Self-Healing Demonstration

Current replicas:

```bash
kubectl get deployment bankapp -n bankapp
```

Output:

```text
1 Replica
```

Introduce configuration drift:

```bash
kubectl scale deployment bankapp \
-n bankapp \
--replicas=3
```

Deployment scaled successfully.

ArgoCD continuously compared the cluster with Git.

Desired state in Git:

```yaml
replicaCount: 1
```

Actual cluster state:

```text
3 replicas
```

ArgoCD detected the drift and automatically reconciled the deployment.

Verification:

```bash
kubectl get deployment bankapp -n bankapp
```

Output:

```text
READY 1/1
```

No manual rollback was required.

This demonstrated ArgoCD Self-Healing.

---

# GitOps Workflow

```
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
Detect Drift
     │
Sync Application
     │
     ▼
Amazon EKS
```

---

# Resources Created

| Resource               | Count |
| ---------------------- | ----: |
| Namespace              |     1 |
| Deployments            |     3 |
| Services               |     3 |
| PersistentVolumeClaims |     2 |
| ConfigMaps             |     1 |
| Secrets                |     1 |
| PreSync Hook Job       |     1 |

---

# Features Demonstrated

- GitOps
- ArgoCD
- Helm
- Amazon EKS
- Automated Sync
- Self-Healing
- Namespace Auto Creation
- Helm Value Files
- Git as Source of Truth
- Kubernetes Reconciliation
- Production GitOps Workflow

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

kubectl get application bankapp -n argocd
```

---

# Screenshots

Create a folder:

```
docs/screenshots/day-84/
```

Capture:

```
01-argocd-dashboard.png
02-application-resource-tree.png
03-self-healing-before.png
04-self-healing-after.png
05-running-pods.png
```

---

# Key Learnings

- Git should be the single source of truth.
- ArgoCD continuously watches Git repositories.
- Automated synchronization eliminates manual deployments.
- Self-healing restores configuration drift automatically.
- Helm integrates seamlessly with ArgoCD.
- GitOps provides reliable, repeatable, and auditable Kubernetes deployments.
- Amazon EKS works effectively with GitOps for production environments.

---

# Outcome

Successfully implemented a complete **GitOps deployment pipeline** using **ArgoCD**, **Helm**, and **Amazon EKS**. Verified automated synchronization, namespace creation, Helm-based application deployment, persistent storage provisioning, and ArgoCD self-healing by restoring the application to its desired state after an intentional configuration drift.
