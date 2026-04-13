# Day 82 — End-to-End GitOps Deployment on Amazon EKS

## Overview

Day 82 focused on implementing and validating a complete **GitOps deployment workflow on Amazon EKS**.

The AI BankApp was migrated from a manually managed Helm deployment to a declarative deployment model using **Helm and Argo CD**, with **Envoy Gateway** providing application routing and **cert-manager + Let's Encrypt** providing automated TLS.

By the end of the lab, Argo CD reported:

```text
Synced | Healthy
```

and the application was successfully accessible over HTTPS.

---

## What I Learned

- How to package Kubernetes workloads using Helm
- How to deploy Helm charts through Argo CD
- How Argo CD automated sync, pruning, and self-healing work
- How to validate Helm charts before deployment
- How to debug Kubernetes workloads using pod events
- How to configure Kubernetes Gateway API with Envoy Gateway
- How to route traffic to an Argo CD-managed service
- How cert-manager automates TLS certificate management
- How Let's Encrypt ACME certificates are issued
- How to migrate from manually managed Helm releases to GitOps
- How to validate an end-to-end production-style Kubernetes deployment

---

## Architecture

```text
Developer
    |
    | git push
    v
GitHub Repository
    |
    | feat/gitops
    v
Argo CD
    |
    | Detect + Reconcile
    v
Helm Chart
    |
    | values-dev.yaml
    v
Amazon EKS
    |
    +-----------------------------+
    |              |              |
    v              v              v
 BankApp         MySQL          Ollama
    |
    v
Envoy Gateway
    |
    | Gateway API
    v
cert-manager
    |
    | ACME
    v
Let's Encrypt
    |
    v
HTTPS BankApp
```

---

## Technology Stack

| Technology    | Purpose                          |
| ------------- | -------------------------------- |
| AWS           | Cloud platform                   |
| Amazon EKS    | Managed Kubernetes cluster       |
| Terraform     | Infrastructure provisioning      |
| Kubernetes    | Container orchestration          |
| Helm          | Kubernetes package management    |
| Argo CD       | GitOps continuous delivery       |
| Envoy Gateway | Gateway API implementation       |
| cert-manager  | Certificate lifecycle management |
| Let's Encrypt | TLS certificate authority        |
| MySQL         | Application database             |
| Ollama        | AI model runtime                 |
| GitHub        | Git repository and GitOps source |
| Git           | Version control                  |

---

# 1. Helm Chart

The BankApp stack is packaged as a reusable Helm chart.

```text
helm-chart/bankapp/
├── .helmignore
├── Chart.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── mysql-deployment.yaml
│   ├── ollama-deployment.yaml
│   ├── pvc.yaml
│   ├── secret.yaml
│   ├── service.yaml
│   ├── hooks/
│   │   └── pre-install-job.yaml
│   └── tests/
│       └── test-connection.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
└── values-prod.yaml
```

Environment-specific values allow the same chart to be reused across different deployment environments.

---

## Helm Validation

Before allowing Argo CD to deploy the chart, I validated it locally.

### Lint the Chart

```bash
helm lint ./helm-chart/bankapp \
  -f ./helm-chart/bankapp/values-dev.yaml
```

Result:

```text
1 chart(s) linted, 0 chart(s) failed
```

### Render Kubernetes Manifests

```bash
helm template bankapp-dev ./helm-chart/bankapp \
  -f ./helm-chart/bankapp/values-dev.yaml \
  -n bankapp > /tmp/bankapp-rendered.yaml
```

Inspect generated resources:

```bash
grep '^kind:' /tmp/bankapp-rendered.yaml
```

The chart rendered resources including:

```text
Secret
ConfigMap
PersistentVolumeClaim
Service
Deployment
Pod
Job
```

### Kubernetes Server-Side Validation

```bash
helm template bankapp-dev ./helm-chart/bankapp \
  -f ./helm-chart/bankapp/values-dev.yaml \
  -n bankapp | kubectl apply --dry-run=server -f -
```

This confirmed that the rendered manifests were accepted by the Kubernetes API server.

---

# 2. Argo CD GitOps Deployment

Argo CD uses Git as the source of truth for the BankApp deployment.

The application source configuration points to:

```text
Repository:
cloudwithpreetham/AI-BankApp-DevOps

Branch:
feat/gitops

Path:
helm-chart/bankapp

Values:
values-dev.yaml
```

The deployment enables:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true

  syncOptions:
    - CreateNamespace=true
    - ServerSideApply=true
```

This provides:

- Automatic synchronization
- Automatic resource pruning
- Self-healing
- Namespace creation
- Server-side apply

---

## Deploy the Argo CD Application

```bash
kubectl apply -f argocd/application.yml
```

Initially the application reported:

```text
OutOfSync | Missing
```

Argo CD automatically reconciled the desired state from Git.

Final verification:

```bash
kubectl get application bankapp -n argocd \
  -o jsonpath='{.status.sync.status}{" | "}{.status.health.status}{"\n"}'
```

Result:

```text
Synced | Healthy
```

---

## Argo CD — Synced & Healthy

![Argo CD Synced and Healthy](docs/screenshots/gitops/01-argocd-synced-healthy.png)

The Argo CD dashboard confirms that the application is synchronized with Git and all managed resources are healthy.

---

# 3. Amazon EKS Workloads

The GitOps deployment created three primary application workloads:

```text
BankApp
MySQL
Ollama
```

Verify pods:

```bash
kubectl get pods -n bankapp -o wide
```

Final state:

```text
bankapp          1/1   Running
bankapp-mysql    1/1   Running
bankapp-ollama   1/1   Running
```

Verify deployments:

```bash
kubectl get deployments -n bankapp
```

All deployments reported their expected replicas as available.

---

## EKS Workloads Running

![EKS BankApp Pods Running](docs/screenshots/gitops/02-eks-bankapp-pods-running.png)

---

# 4. Debugging Ollama Startup

During deployment, the Ollama pod initially remained in:

```text
ContainerCreating
```

I investigated the pod instead of immediately modifying the deployment.

```bash
kubectl describe pod <ollama-pod> -n bankapp
```

The Kubernetes events showed:

```text
Successfully assigned
AttachVolume.Attach succeeded
Pulling image "ollama/ollama:latest"
Successfully pulled image
Container created
Container started
```

The Ollama container image was large and required additional time to download.

After the image was pulled and the readiness probe succeeded:

```text
bankapp-ollama   1/1   Running
```

Argo CD then transitioned from:

```text
Synced | Progressing
```

to:

```text
Synced | Healthy
```

### Lesson

`Synced` and `Healthy` represent different states.

- **Synced** — Kubernetes manifests match the desired state in Git.
- **Healthy** — The deployed workloads have reached their expected operational state.

---

# 5. Envoy Gateway

Envoy Gateway provides external access to the BankApp using Kubernetes Gateway API.

Verify the Gateway:

```bash
kubectl get gateway -n bankapp
```

The final Gateway state showed:

```text
CLASS:       envoy-gateway
PROGRAMMED:  True
```

An AWS load balancer was automatically provisioned for the Gateway.

---

## Envoy Gateway Programmed

![Envoy Gateway Programmed](docs/screenshots/gitops/03-envoy-gateway-programmed.png)

---

# 6. HTTPRoute Backend Fix

During the GitOps migration, an important routing issue was discovered.

The existing HTTPRoute was targeting:

```text
bankapp-dev-service
```

However, the Argo CD-managed Helm deployment created:

```text
bankapp-service
```

The route was corrected:

```yaml
backendRefs:
  - group: ""
    kind: Service
    name: bankapp-service
    port: 8080
    weight: 1
```

Validate before applying:

```bash
kubectl apply --dry-run=server -f k8s/gateway.yml
```

Apply:

```bash
kubectl apply -f k8s/gateway.yml
```

Verify:

```bash
kubectl get httproute bankapp-route -n bankapp \
  -o jsonpath='{.spec.rules[*].backendRefs[*].name}{"\n"}'
```

Result:

```text
bankapp-service
```

The HTTPRoute reported:

```text
Accepted:     True
ResolvedRefs: True
```

---

# 7. Service Endpoint Verification

The BankApp service was verified using:

```bash
kubectl get svc bankapp-service -n bankapp
```

Endpoint discovery:

```bash
kubectl get endpointslice -n bankapp \
  -l kubernetes.io/service-name=bankapp-service
```

This confirmed that the service had a valid endpoint pointing to the running BankApp pod.

---

# 8. Automated TLS with Let's Encrypt

TLS certificate management is automated through cert-manager.

Verify the certificate:

```bash
kubectl get certificate -n bankapp
```

Final result:

```text
NAME          READY   SECRET
bankapp-tls   True    bankapp-tls
```

The CertificateRequest reported:

```text
APPROVED: True
READY:    True
ISSUER:   letsencrypt-prod
```

The ACME order reached:

```text
STATE: valid
```

---

## Let's Encrypt Certificate Ready

![Let's Encrypt Certificate Ready](docs/screenshots/gitops/04-letsencrypt-certificate-ready.png)

This confirms the complete certificate flow:

```text
Gateway
   |
   v
cert-manager
   |
   v
ACME Challenge
   |
   v
Let's Encrypt
   |
   v
TLS Secret
   |
   v
HTTPS Listener
```

---

# 9. HTTPS Application Verification

The application endpoint was tested with:

```bash
curl -I https://13.206.8.151.nip.io
```

The server returned:

```text
HTTP/2 302
location: https://13.206.8.151.nip.io/login
strict-transport-security: max-age=31536000 ; includeSubDomains
```

This confirms:

- DNS resolution works
- HTTPS works
- TLS termination works
- Envoy Gateway routing works
- BankApp is reachable
- Authentication redirects users to `/login`

---

## BankApp Accessible over HTTPS

![BankApp HTTPS Login](docs/screenshots/gitops/05-bankapp-https-login.png)

---

# 10. Removing the Old Manual Helm Release

Before completing the migration, two versions of the application existed:

```text
bankapp-dev
bankapp
```

`bankapp-dev` belonged to the previous manually installed Helm release.

`bankapp` was managed by Argo CD.

The old release was removed:

```bash
helm uninstall bankapp-dev -n bankapp
```

After cleanup:

```bash
kubectl get pods -n bankapp
```

Only the GitOps-managed workloads remained:

```text
bankapp
bankapp-mysql
bankapp-ollama
```

This established Argo CD as the deployment owner and Git as the single source of truth.

---

# 11. Git Repository and Branch Management

The project originally referenced the upstream repository.

The Git remote was updated to the personal fork:

```bash
git remote set-url origin \
  https://github.com/cloudwithpreetham/AI-BankApp-DevOps.git
```

The Argo CD Application was also updated to use the same repository.

During synchronization, the local and remote branches were found to have divergent histories.

Before modifying the remote history, the previous remote state was preserved:

```bash
git branch backup/old-remote-gitops 69f6f7b
```

This created a recovery reference for the previous commits.

### Lesson

Before performing destructive Git operations, preserve important history using a branch or tag.

---

# 12. GitOps Commit History

Important commits from the implementation:

```text
1055fb8 docs(gitops): document end-to-end EKS deployment verification
d28b6af fix(gateway): route traffic to Argo CD managed service
c608d29 fix(gitops): point Argo CD to fork repository
04ca364 feat(gitops): deploy BankApp to EKS with Helm and Argo CD
```

---

## GitOps Commit Evidence

![GitOps Commit History](docs/screenshots/gitops/06-gitops-commit-history.png)

The commit history follows a clear progression:

```text
feat(gitops)
     |
     v
fix(gitops)
     |
     v
fix(gateway)
     |
     v
docs(gitops)
```

---

# 13. Final Verification

The complete environment was verified using:

```bash
git fetch origin

git status -sb

git log -3 --oneline --decorate

kubectl get application bankapp -n argocd \
  -o jsonpath='{.status.sync.status}{" | "}{.status.health.status}{"\n"}'

kubectl get pods -n bankapp

kubectl get gateway,httproute -n bankapp

kubectl get certificate -n bankapp

curl -I https://13.206.8.151.nip.io
```

Final Argo CD status:

```text
Synced | Healthy
```

Final Git state:

```text
HEAD -> feat/gitops
origin/feat/gitops
```

Both local and remote branches point to the same commit.

---

# Final Deployment Status

| Component                 | Final Status |
| ------------------------- | ------------ |
| Amazon EKS                | Running      |
| Helm Chart                | Valid        |
| Argo CD Sync              | Synced       |
| Argo CD Health            | Healthy      |
| BankApp                   | Running      |
| MySQL                     | Running      |
| Ollama                    | Running      |
| Envoy Gateway             | Programmed   |
| HTTPRoute                 | Accepted     |
| HTTPRoute References      | Resolved     |
| cert-manager              | Operational  |
| Let's Encrypt Certificate | Ready        |
| HTTPS                     | Working      |
| GitOps Branch             | Synchronized |
| Deployment Documentation  | Complete     |

---

# Troubleshooting Summary

## Argo CD initially showed OutOfSync

Argo CD had just created the application and had not completed reconciliation.

After automated synchronization:

```text
Synced | Healthy
```

---

## GitHub push returned 403

The repository remote pointed to the upstream repository where the authenticated account did not have push permission.

Fixed by changing `origin` to the personal fork.

---

## Git branches diverged

The local branch contained new GitOps work while the remote branch contained additional commits.

The previous remote history was preserved using a backup branch before continuing.

---

## Ollama remained in ContainerCreating

The image was still being downloaded.

`kubectl describe pod` confirmed that scheduling, volume attachment, image pulling, container creation, and startup were proceeding normally.

No deployment change was necessary.

---

## HTTPRoute targeted the old service

The Gateway still routed traffic to:

```text
bankapp-dev-service
```

It was changed to:

```text
bankapp-service
```

After the fix:

```text
Accepted=True
ResolvedRefs=True
```

---

## Duplicate workloads existed

The previous manual Helm deployment and the new Argo CD deployment were both running.

The old release was removed:

```bash
helm uninstall bankapp-dev -n bankapp
```

Only the GitOps-managed workloads remained.

---

# Key DevOps Lessons

## Git Is the Source of Truth

In a GitOps workflow, production changes should originate from Git rather than manual cluster modifications.

---

## Validate Before Deploying

Useful pre-deployment checks include:

```bash
helm lint
helm template
kubectl apply --dry-run=server
```

These reduce deployment failures before Argo CD reconciliation begins.

---

## Synced Does Not Mean Healthy

Argo CD can report:

```text
Synced | Progressing
```

while Kubernetes is still starting workloads.

This is normal.

---

## Debug Using Evidence

For pod startup problems, inspect:

```bash
kubectl describe pod
kubectl get events
kubectl logs
```

before changing manifests.

---

## Avoid Multiple Deployment Owners

Running a manual Helm release and an Argo CD-managed release simultaneously creates configuration drift and duplicate workloads.

One deployment mechanism should own the application.

---

## Protect Git History

Before force-pushing or rewriting history, create a recovery reference:

```bash
git branch backup/<name> <commit>
```

This simple step can prevent accidental loss of work.

---

# Screenshot Evidence

```text
docs/screenshots/gitops/
├── 01-argocd-synced-healthy.png
├── 02-eks-bankapp-pods-running.png
├── 03-envoy-gateway-programmed.png
├── 04-letsencrypt-certificate-ready.png
├── 05-bankapp-https-login.png
└── 06-gitops-commit-history.png
```

These screenshots provide evidence for the complete deployment lifecycle from GitOps synchronization through HTTPS application access.

---

# Day 82 Outcome

Day 82 successfully completed an **end-to-end GitOps deployment on Amazon EKS**.

The final workflow is:

```text
GitHub
   |
   v
Argo CD
   |
   v
Helm
   |
   v
Amazon EKS
   |
   +--> BankApp
   +--> MySQL
   +--> Ollama
   |
   v
Envoy Gateway
   |
   v
cert-manager
   |
   v
Let's Encrypt
   |
   v
HTTPS BankApp
```

The application is now:

- Declaratively deployed
- Automatically synchronized
- Self-healing
- Managed through Helm
- Running on Amazon EKS
- Exposed through Envoy Gateway
- Secured with Let's Encrypt TLS
- Accessible over HTTPS
- Verified with deployment evidence

## Day 82 Status

**COMPLETE**
