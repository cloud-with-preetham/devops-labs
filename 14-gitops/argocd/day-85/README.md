# Day 85 – ArgoCD Advanced GitOps Workflows

> Mastering advanced GitOps concepts with ArgoCD on Amazon EKS by implementing Sync Strategies, Sync Waves, Rollback, App of Apps, Notifications, and RBAC.

![Kubernetes](https://img.shields.io/badge/Kubernetes-EKS-blue?logo=kubernetes)
![ArgoCD](https://img.shields.io/badge/GitOps-ArgoCD-red?logo=argo)
![Helm](https://img.shields.io/badge/Helm-Charts-0F1689?logo=helm)
![AWS](https://img.shields.io/badge/AWS-EKS-FF9900?logo=amazonaws)
![Status](https://img.shields.io/badge/Status-Completed-success)

---

# Project Overview

On **Day 85** of my **90 Days of DevOps** journey, I explored advanced ArgoCD features that are commonly used in production Kubernetes environments.

This hands-on lab focused on implementing advanced GitOps workflows, deployment orchestration, automated reconciliation, application rollback, notification systems, and access control.

---

# Architecture

```text
                    GitHub Repository
                           │
                           │
                    Git Push / Commit
                           │
                           ▼
                     Root Application
                   (App of Apps Pattern)
                           │
          ┌────────────────┴───────────────┐
          │                                │
          ▼                                ▼
   BankApp Application             Future Applications
          │
          ▼
       Helm Charts
          │
          ▼
      Amazon EKS Cluster
          │
          ▼
     Kubernetes Resources
          │
          ▼
 ArgoCD Sync • Notifications • RBAC
```

---

# Technologies Used

- Amazon EKS
- Kubernetes
- ArgoCD
- Helm
- GitHub
- GitOps
- YAML
- kubectl
- ArgoCD CLI

---

# Learning Objectives

- Understand Manual vs Automated Sync
- Configure Sync Waves
- Perform Rollback
- Learn GitOps Reconciliation
- Build App of Apps Architecture
- Configure Notifications
- Implement RBAC
- Practice Production GitOps Workflows

---

# Tasks Completed

## Task 1 – Sync Strategies

Implemented different synchronization strategies.

### Completed

- Manual Sync
- Automated Sync
- Diff
- Dry Run
- Manual Synchronization
- Self Heal
- Auto Prune

### Commands

```bash
argocd app set bankapp --sync-policy none

argocd app diff bankapp

argocd app sync bankapp --dry-run

argocd app sync bankapp

argocd app set bankapp \
--sync-policy automated \
--self-heal \
--auto-prune
```

---

## Task 2 – Sync Waves

Configured deployment ordering using Sync Wave annotations.

| Resource | Sync Wave |
|----------|----------:|
| PreSync Job | -1 |
| ConfigMap | 0 |
| Secret | 0 |
| PVC | 0 |
| Service | 1 |
| Deployment | 2 |

This ensured Kubernetes resources were deployed in the correct order.

---

## Task 3 – Rollback

Performed application rollback using ArgoCD.

### Commands

```bash
argocd app history bankapp

argocd app rollback bankapp 1

argocd app sync bankapp
```

### Learned

- Application History
- Rollback Strategy
- GitOps Reconciliation
- Auto Sync Behavior

---

## Task 4 – App of Apps Pattern

Created a parent application to manage child applications automatically.

### Directory

```text
argocd/
├── apps/
│   └── bankapp-app.yaml
└── root-app.yaml
```

Successfully synchronized:

- Root Application
- Child Application

---

## Task 5 – Notifications

Configured ArgoCD Notifications.

### Verified

- Notifications Controller
- Notifications ConfigMap
- Notifications Secret

Configured

- Webhook Service
- Notification Template
- Trigger
- Application Subscription

### Commands

```bash
kubectl rollout restart deployment argocd-notifications-controller -n argocd

kubectl annotate application bankapp \
-n argocd \
notifications.argoproj.io/subscribe.on-sync-succeeded.webhook=webhook

argocd app sync bankapp
```

---

## Task 6 – RBAC

Configured Role-Based Access Control.

Created a custom **ReadOnly** role.

```yaml
policy.csv: |
  p, role:readonly, applications, get, */*, allow
  p, role:readonly, projects, get, *, allow
  p, role:readonly, repositories, get, *, allow

  g, dev-team, role:readonly
```

Restarted ArgoCD Server.

```bash
kubectl rollout restart deployment argocd-server -n argocd
```

---

# Commands Practiced

```bash
argocd app get
argocd app diff
argocd app sync
argocd app history
argocd app rollback
argocd app wait
argocd app set

kubectl apply
kubectl annotate
kubectl edit
kubectl describe
kubectl rollout restart
kubectl rollout status
kubectl get
```

---

# Project Structure

```text
AI-BankApp-DevOps
│
├── argocd
│   ├── apps
│   │   └── bankapp-app.yaml
│   └── root-app.yaml
│
├── helm-chart
│   └── bankapp
│
├── docs
│   └── screenshots
│       └── day-85
│
└── README.md
```

---

# Screenshots

Create a folder:

```text
docs/screenshots/day-85/
```

Recommended screenshots:

- ArgoCD Dashboard
- Manual Sync
- Sync Waves
- Rollback
- App History
- Root App
- Child App
- Notifications Controller
- Notification Configuration
- Notification Sync
- RBAC Configuration
- Final Healthy State

---

# What I Learned

- GitOps Best Practices
- Manual vs Automated Synchronization
- Sync Waves
- Deployment Ordering
- PreSync Hooks
- Rollback Strategy
- GitOps Reconciliation
- Parent-Child Application Management
- Notifications
- Webhooks
- RBAC
- Production ArgoCD Workflows

---

# Outcome

Successfully implemented advanced GitOps capabilities using ArgoCD on Amazon EKS, including deployment synchronization, ordered resource deployment, rollback strategies, the App of Apps pattern, notifications, and role-based access control. All applications remained **Healthy** and **Synced**, demonstrating a production-ready GitOps workflow.

---

# Repository

```text
AI-BankApp-DevOps/
```

---

# Connect With Me

**GitHub:** https://github.com/cloudwithpreetham

**LinkedIn:** https://www.linkedin.com/in/preetham-pereira/

---

## Day 85 Status

**Completed Successfully**

**Next:** Day 86 – Progressive Delivery & Advanced Deployment Strategies
