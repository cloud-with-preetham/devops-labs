# Day 86 – GitOps CI/CD Pipeline with GitHub Actions, ArgoCD & Amazon EKS

![GitOps](https://img.shields.io/badge/GitOps-ArgoCD-red)
![GitHub Actions](https://img.shields.io/badge/CI-GitHub_Actions-blue)
![Kubernetes](https://img.shields.io/badge/Platform-Kubernetes-326CE5)
![Amazon EKS](https://img.shields.io/badge/Cloud-Amazon_EKS-FF9900)
![Docker](https://img.shields.io/badge/Container-Docker-2496ED)
![Helm](https://img.shields.io/badge/Package-Helm-0F1689)

---

# Project Overview

This project demonstrates a complete **GitOps Continuous Delivery pipeline** using:

- GitHub Actions
- DockerHub
- Helm
- ArgoCD
- Amazon EKS
- Kubernetes

A source code change automatically triggers the CI pipeline, builds the application, creates a Docker image, updates deployment manifests, and synchronizes the Kubernetes cluster through ArgoCD.

---

# Architecture

```
                  Developer
                      │
               Git Push (feat/gitops)
                      │
                      ▼
              GitHub Repository
                      │
                      ▼
             GitHub Actions CI
                      │
      ┌───────────────┼────────────────┐
      │               │                │
      ▼               ▼                ▼
 Build Project    Run Tests     Build Docker Image
                                      │
                                      ▼
                               Push to DockerHub
                                      │
                                      ▼
                      Update Kubernetes Manifest
                                      │
                                      ▼
                          Commit Updated Manifest
                                      │
                                      ▼
                              Git Repository
                                      │
                                      ▼
                                  ArgoCD
                                      │
                                      ▼
                              Amazon EKS Cluster
                                      │
                                      ▼
                           Spring Boot Application
```

---

# Technology Stack

| Category | Tools |
|----------|-------|
| Version Control | Git, GitHub |
| CI | GitHub Actions |
| CD | ArgoCD |
| Container | Docker |
| Registry | DockerHub |
| Orchestration | Kubernetes |
| Package Manager | Helm |
| Cloud | Amazon EKS |
| Build Tool | Maven |
| Language | Java 21 |
| Framework | Spring Boot |

---

# Project Structure

```
AI-BankApp-DevOps/

├── .github/
│   └── workflows/
│       └── gitops-ci.yml
│
├── helm-chart/
│   └── bankapp/
│
├── k8s/
│   └── bankapp-deployment.yml
│
├── src/
│
└── README.md
```

---

# GitOps Workflow

## Step 1

Developer pushes code to:

```
feat/gitops
```

---

## Step 2

GitHub Actions automatically:

- Checks out repository
- Sets up Java
- Builds the application
- Executes Maven tests
- Builds Docker image
- Pushes image to DockerHub
- Updates Kubernetes manifest
- Creates automated Git commit

---

## Step 3

ArgoCD continuously watches Git.

Whenever a commit is detected:

- Synchronizes Kubernetes resources
- Performs rolling updates
- Maintains desired state

---

## Step 4

Amazon EKS deploys the latest application automatically.

---

# GitOps Features Demonstrated

- Automated CI pipeline
- Continuous Deployment
- Declarative Infrastructure
- Git as Single Source of Truth
- Automated Synchronization
- Kubernetes Self-Healing
- Drift Detection
- Rolling Deployment

---

# Drift Detection Demonstration

## Scenario 1

Manual Deployment Scaling

```bash
kubectl scale deployment bankapp \
-n bankapp \
--replicas=1
```

Result:

- ArgoCD detected drift
- Deployment became OutOfSync
- Replica count restored automatically

---

## Scenario 2

Manual Image Update

```bash
kubectl set image deployment/bankapp \
bankapp=nginx:latest \
-n bankapp
```

Investigation identified that ArgoCD was deploying from the Helm chart while the CI workflow updated a raw Kubernetes manifest. This highlighted the importance of ensuring CI updates the same deployment source that ArgoCD consumes.

---

## Scenario 3

Delete Kubernetes Service

```bash
kubectl delete service bankapp-service \
-n bankapp
```

Result:

- Service removed
- ArgoCD detected missing resource
- Service recreated automatically

---

# Useful Commands

## View Application

```bash
argocd app get bankapp
```

---

## Watch Pods

```bash
kubectl get pods -n bankapp -w
```

---

## Scale Deployment

```bash
kubectl scale deployment bankapp \
-n bankapp \
--replicas=1
```

---

## Delete Service

```bash
kubectl delete service bankapp-service \
-n bankapp
```

---

## View Deployment

```bash
kubectl get deployment bankapp -n bankapp
```

---

# Challenges & Solutions

## DockerHub Authentication

**Issue**

GitHub Actions failed while pushing the Docker image.

**Solution**

- Verified DockerHub repository
- Regenerated DockerHub access token
- Updated GitHub Secrets

---

## Image Not Updating

**Issue**

Application continued using the original image.

**Root Cause**

ArgoCD was deploying from the Helm chart while the GitHub Actions workflow updated a Kubernetes manifest outside the Helm chart.

**Resolution**

Identified the deployment source mismatch and documented the correct GitOps approach.

---

# Key Learnings

- Git is the single source of truth.
- GitHub Actions automates the CI workflow.
- ArgoCD continuously reconciles Kubernetes with Git.
- Drift detection prevents configuration drift.
- Self-healing restores deleted or modified resources automatically.
- Helm-based GitOps requires CI pipelines to update Helm values rather than unrelated manifests.

---

# Screenshots

## Workflow

- GitOps Workflow Overview
- Pipeline Steps
- GitHub Secrets
- Workflow Configuration

## Deployment

- Docker Image Configuration
- ArgoCD Repository
- GitHub Actions Success
- Automated Bot Commit
- ArgoCD Synchronization

## Self-Healing

- Manual Scaling
- OutOfSync Status
- Automatic Recovery
- Service Deletion
- Service Recreation

---

# Learning Outcomes

After completing this project, I gained practical experience with:

- GitOps principles
- GitHub Actions CI
- Docker image automation
- DockerHub integration
- Helm deployments
- ArgoCD synchronization
- Amazon EKS
- Kubernetes self-healing
- Drift detection
- Production-style Continuous Delivery

---

# Conclusion

This project demonstrates a complete GitOps workflow using GitHub Actions, DockerHub, ArgoCD, Helm, and Amazon EKS. It showcases automated application delivery, declarative deployments, Kubernetes drift detection, and self-healing, providing hands-on experience with modern cloud-native CI/CD practices.
