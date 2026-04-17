# Day 86 – GitOps CI/CD Pipeline with GitHub Actions, ArgoCD & Amazon EKS

## Project Overview

Today I implemented a complete GitOps workflow using GitHub Actions, DockerHub, ArgoCD, Helm, and Amazon EKS.

The objective was to automate application delivery from source code changes to Kubernetes deployment while demonstrating GitOps principles such as declarative deployments, continuous synchronization, and automatic self-healing.

---

# Objectives

- Understand GitOps workflow
- Explore GitHub Actions CI pipeline
- Configure personal DockerHub repository
- Configure GitOps pipeline for personal GitHub repository
- Build and push Docker images automatically
- Update Kubernetes manifests automatically
- Synchronize deployments using ArgoCD
- Demonstrate Kubernetes drift detection
- Validate ArgoCD self-healing

---

# Architecture

```
Developer
     │
     ▼
 Git Push
     │
     ▼
GitHub Actions
     │
     ├── Checkout Code
     ├── Build Spring Boot
     ├── Run Tests
     ├── Build Docker Image
     ├── Push DockerHub
     ├── Update Kubernetes Manifest
     └── Commit Updated Manifest
             │
             ▼
      Git Repository
             │
             ▼
          ArgoCD
             │
             ▼
        Amazon EKS
             │
             ▼
      Spring Boot Application
```

---

# Tools Used

- Git
- GitHub
- GitHub Actions
- Docker
- DockerHub
- Kubernetes
- Helm
- ArgoCD
- Amazon EKS
- kubectl
- AWS CLI

---

# Project Structure

```
AI-BankApp-DevOps/

├── .github/workflows/
│      gitops-ci.yml
│
├── helm-chart/
│      bankapp/
│
├── k8s/
│      bankapp-deployment.yml
│
└── src/
```

---

# Task 1 – Understanding GitOps Workflow

Studied the GitHub Actions workflow and understood:

- Workflow triggers
- Branch filters
- Path filters
- Manual workflow dispatch
- Build pipeline
- Docker image creation
- DockerHub push
- Manifest update
- Automated Git commit
- Continuous deployment using ArgoCD

---

# Task 2 – Configure GitOps Pipeline

Updated the workflow for my environment.

Configured:

- DockerHub username
- DockerHub repository
- GitHub Secrets
- ArgoCD repository

Verified:

- Git remote
- Feature branch
- Workflow configuration

---

# Task 3 – Execute GitOps Pipeline

Modified application source code.

Triggered GitHub Actions.

Pipeline automatically:

- Built application
- Executed Maven tests
- Built Docker image
- Pushed image to DockerHub
- Updated Kubernetes deployment manifest
- Created automated Git commit
- Triggered ArgoCD synchronization

Verified:

- Successful GitHub Actions execution
- Docker image push
- Bot commit
- ArgoCD synchronization

---

# Task 4 – Drift Detection & Self-Healing

## Scenario 1

Scaled deployment manually.

```bash
kubectl scale deployment bankapp \
-n bankapp \
--replicas=1
```

Observed:

- ArgoCD detected drift
- Deployment became OutOfSync
- Desired replica count restored automatically

Result:

Self-healing successful.

---

## Scenario 2

Attempted to modify deployment image.

```bash
kubectl set image deployment/bankapp \
bankapp=nginx:latest \
-n bankapp
```

Investigation showed:

The repository updates `k8s/bankapp-deployment.yml`, while ArgoCD deploys resources from the Helm chart (`helm-chart/bankapp`) using `values-dev.yaml`. Because the Helm values still referenced the original image repository, the running deployment continued using the image defined by the Helm chart.

Learning:

CI pipelines must update the same manifests or Helm values that ArgoCD consumes.

---

## Scenario 3

Deleted Kubernetes Service.

```bash
kubectl delete service bankapp-service -n bankapp
```

Observed:

- Service deleted
- ArgoCD detected missing resource
- Service recreated automatically
- Application returned to desired state

Result:

Self-healing successful.

---

# Important Commands

## Check ArgoCD

```bash
argocd app get bankapp
```

## Watch Pods

```bash
kubectl get pods -n bankapp -w
```

## Scale Deployment

```bash
kubectl scale deployment bankapp \
-n bankapp \
--replicas=1
```

## Delete Service

```bash
kubectl delete service bankapp-service \
-n bankapp
```

## View Deployment

```bash
kubectl get deployment bankapp -n bankapp
```

---

# Challenges Faced

## Docker Push Authentication

Issue:

GitHub Actions failed to push Docker image.

Resolution:

Verified DockerHub repository, regenerated DockerHub access token, and updated GitHub Secrets.

---

## Image Not Updating

Issue:

Deployment continued using the original Docker image.

Root Cause:

ArgoCD was deploying the Helm chart while the CI workflow updated a raw Kubernetes manifest.

Resolution:

Identified the configuration mismatch between the deployment source and the automated manifest update.

---

# Key Learnings

- GitOps uses Git as the single source of truth.
- ArgoCD continuously monitors the Git repository.
- Automated synchronization eliminates manual deployments.
- Kubernetes resources can recover automatically after manual changes.
- Helm charts and Kubernetes manifests must remain aligned with the CI pipeline.
- Self-healing improves application reliability and consistency.

---

# Screenshots

- GitOps Workflow
- GitHub Secrets
- Workflow Configuration
- Docker Image Configuration
- ArgoCD Repository
- GitHub Actions Success
- Bot Commit
- ArgoCD Sync
- Replica Scaling
- OutOfSync Status
- Self-Healing
- Service Deletion
- Service Recreation

---

# Outcome

Successfully implemented a production-style GitOps workflow integrating GitHub Actions, DockerHub, ArgoCD, Helm, and Amazon EKS.

Validated continuous deployment, automated synchronization, drift detection, and Kubernetes self-healing while identifying an important Helm configuration consideration in the CI/CD pipeline.
