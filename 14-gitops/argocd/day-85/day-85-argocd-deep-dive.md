# Day 85 – ArgoCD Advanced GitOps Workflows

## Overview

Day 85 focused on advanced GitOps concepts using ArgoCD. The lab covered synchronization strategies, Sync Waves, rollback mechanisms, the App of Apps pattern, Notifications, and Role-Based Access Control (RBAC). These features are commonly used in production Kubernetes environments to improve deployment reliability, automation, security, and application lifecycle management.

---

# Objectives

- Understand Manual vs Automated Sync
- Learn ArgoCD Sync Waves
- Perform application rollback
- Implement the App of Apps pattern
- Configure ArgoCD Notifications
- Configure ArgoCD RBAC
- Gain hands-on experience with production GitOps workflows

---

# Environment

| Component | Value |
|----------|-------|
| Kubernetes | Amazon EKS |
| GitOps Tool | ArgoCD |
| Application | AI-BankApp |
| Repository | AI-BankApp-DevOps |
| Namespace | bankapp |
| Git Branch | feat/gitops |

---

# Task 1 – Sync Strategies

## Manual Sync

Changed the application from Automated Sync to Manual Sync.

```bash
argocd app set bankapp --sync-policy none
```

Verified the application entered the **OutOfSync** state after making changes.

---

## Compare Differences

```bash
argocd app diff bankapp
```

Compared Git repository manifests against the live Kubernetes cluster.

---

## Dry Run

```bash
argocd app sync bankapp --dry-run
```

Validated changes before deployment.

---

## Manual Synchronization

```bash
argocd app sync bankapp
```

Successfully synchronized the application.

---

## Re-enable Automated Sync

```bash
argocd app set bankapp \
  --sync-policy automated \
  --self-heal \
  --auto-prune
```

Verified the application returned to:

- Synced
- Healthy

---

# Task 2 – Sync Waves

Implemented deployment ordering using annotations.

## PreSync Hook

```yaml
annotations:
  argocd.argoproj.io/hook: PreSync
  argocd.argoproj.io/sync-wave: "-1"
```

---

## ConfigMap

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "0"
```

---

## Secret

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "0"
```

---

## PersistentVolumeClaim

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "0"
```

---

## Service

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "1"
```

---

## Deployments

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "2"
```

This ensured resources were deployed in the proper sequence.

---

# Task 3 – Rollback

Viewed application history.

```bash
argocd app history bankapp
```

Disabled Auto Sync.

Rolled back to a previous revision.

```bash
argocd app rollback bankapp 1
```

Verified rollback.

Re-synchronized the application.

```bash
argocd app sync bankapp
```

Enabled Automated Sync again.

---

# Task 4 – App of Apps Pattern

Created the following structure.

```
argocd/
├── apps/
│   └── bankapp-app.yaml
└── root-app.yaml
```

Created the Root Application.

```bash
kubectl apply -f argocd/root-app.yaml
```

Initially encountered:

```
ComparisonError
```

Resolved by:

- Committing manifests
- Pushing to GitHub
- Refreshing Root Application
- Synchronizing Root Application

Successfully deployed child applications automatically.

---

# Task 5 – ArgoCD Notifications

Verified installation.

```bash
kubectl get pods -n argocd
```

Verified:

- Notifications Controller
- Notifications ConfigMap
- Notifications Secret

Configured:

- Webhook Service
- Notification Template
- Sync Trigger
- Application Subscription

Restarted controller.

```bash
kubectl rollout restart deployment argocd-notifications-controller -n argocd
```

Subscribed application.

```bash
kubectl annotate application bankapp \
-n argocd \
notifications.argoproj.io/subscribe.on-sync-succeeded.webhook=webhook
```

Triggered notification.

```bash
argocd app sync bankapp
```

Verified successful synchronization.

---

# Task 6 – RBAC

Inspected default RBAC.

```bash
kubectl get configmap argocd-rbac-cm -n argocd -o yaml
```

Created a custom read-only role.

```yaml
policy.csv: |
  p, role:readonly, applications, get, */*, allow
  p, role:readonly, projects, get, *, allow
  p, role:readonly, repositories, get, *, allow

  g, dev-team, role:readonly

policy.default: role:readonly
```

Restarted ArgoCD Server.

```bash
kubectl rollout restart deployment argocd-server -n argocd
```

Verified updated RBAC configuration.

---

# Commands Used

```bash
argocd app get
argocd app diff
argocd app sync
argocd app history
argocd app rollback
argocd app wait
argocd app set

kubectl get
kubectl apply
kubectl annotate
kubectl edit
kubectl describe
kubectl rollout restart
kubectl rollout status
```

---

# Project Structure

```
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
└── docs
    └── screenshots
        └── day-85
```

---

# Screenshots

Place all screenshots inside:

```
docs/screenshots/day-85/
```

Suggested screenshots:

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

# Key Learnings

- Manual vs Automated Sync
- GitOps reconciliation
- Sync Waves
- PreSync Hooks
- Rollback strategy
- Application History
- App of Apps architecture
- Notifications
- Webhooks
- RBAC
- Production GitOps workflows

---

# Outcome

Successfully implemented advanced ArgoCD GitOps features including deployment synchronization, ordered resource deployment, rollback strategies, App of Apps architecture, notifications, and RBAC. The application remained **Healthy** and **Synced** throughout validation, demonstrating a production-ready GitOps workflow on Amazon EKS.

---

# Status

**Day 85 Completed Successfully**
