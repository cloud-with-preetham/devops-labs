# Day 82 — End-to-End GitOps Deployment on Amazon EKS

## 90 Days of DevOps

Day 82 focused on completing an end-to-end GitOps deployment of the AI BankApp on Amazon EKS using **Helm, Argo CD, Envoy Gateway, cert-manager, and Let's Encrypt**.

The goal was to move from manually managed Kubernetes resources to a declarative GitOps workflow where Git acts as the source of truth and Argo CD continuously reconciles the desired application state with the EKS cluster.

---

## Objectives

- Deploy BankApp through Argo CD
- Use Helm as the application packaging mechanism
- Configure environment-specific Helm values
- Run BankApp, MySQL, and Ollama on Amazon EKS
- Enable automated Argo CD synchronization
- Enable GitOps self-healing and pruning
- Configure Envoy Gateway
- Route traffic using Kubernetes Gateway API
- Configure HTTPS
- Provision TLS certificates using cert-manager and Let's Encrypt
- Remove the previous manually deployed Helm release
- Validate the complete GitOps deployment
- Document the deployment with screenshots

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
    | Detects Git changes
    v
Helm Chart
    |
    | values-dev.yaml
    v
Amazon EKS
    |
    +----------------------+
    |                      |
    v                      v
BankApp                  MySQL
    |
    +----------------------+
    |
    v
Ollama
    |
    v
Envoy Gateway
    |
    | Kubernetes Gateway API
    v
cert-manager
    |
    | Let's Encrypt TLS
    v
HTTPS BankApp
```

---

## Technologies Used

| Technology | Purpose |
|---|---|
| AWS | Cloud infrastructure |
| Amazon EKS | Managed Kubernetes cluster |
| Terraform | Infrastructure as Code |
| Kubernetes | Container orchestration |
| Helm | Kubernetes application packaging |
| Argo CD | GitOps continuous delivery |
| Envoy Gateway | Kubernetes Gateway API implementation |
| cert-manager | Kubernetes certificate management |
| Let's Encrypt | TLS certificate authority |
| MySQL | Application database |
| Ollama | AI model runtime |
| Git | Version control |
| GitHub | GitOps source repository |

---

# 1. Helm-Based BankApp Deployment

A Helm chart was created for the complete BankApp application stack.

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

This provides reusable environment-specific deployment configuration.

---

## Helm Validation

The Helm chart was linted before deploying it through GitOps.

```bash
helm lint ./helm-chart/bankapp \
  -f ./helm-chart/bankapp/values-dev.yaml
```

Result:

```text
1 chart(s) linted, 0 chart(s) failed
```

The rendered Kubernetes resources were also validated:

```bash
helm template bankapp-dev ./helm-chart/bankapp \
  -f ./helm-chart/bankapp/values-dev.yaml \
  -n bankapp
```

Server-side validation:

```bash
helm template bankapp-dev ./helm-chart/bankapp \
  -f ./helm-chart/bankapp/values-dev.yaml \
  -n bankapp | kubectl apply --dry-run=server -f -
```

The chart successfully generated the required resources.

---

# 2. Argo CD Application

The Argo CD Application was updated to deploy the Helm chart instead of directly managing the previous Kubernetes manifests.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bankapp
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/cloudwithpreetham/AI-BankApp-DevOps.git
    targetRevision: feat/gitops
    path: helm-chart/bankapp

    helm:
      valueFiles:
        - values-dev.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: bankapp

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
```

Important GitOps features:

- Automated synchronization
- Self-healing
- Resource pruning
- Automatic namespace creation
- Server-side apply
- Helm-based resource rendering

---

# 3. Git Repository Correction

Initially, the repository still pointed to the upstream project:

```text
TrainWithShubham/AI-BankApp-DevOps
```

The Git remote and Argo CD repository were updated to the personal fork:

```text
cloudwithpreetham/AI-BankApp-DevOps
```

Git remote:

```bash
git remote set-url origin \
  https://github.com/cloudwithpreetham/AI-BankApp-DevOps.git
```

Argo CD now monitors:

```text
Repository: cloudwithpreetham/AI-BankApp-DevOps
Branch:     feat/gitops
Path:       helm-chart/bankapp
Values:     values-dev.yaml
```

---

# 4. Handling Divergent Git History

The local and remote `feat/gitops` branches had diverged.

Verification:

```bash
git status -sb
```

The branch was:

```text
ahead 2, behind 14
```

The old remote history was preserved locally before continuing:

```bash
git branch backup/old-remote-gitops 69f6f7b
```

This preserved the previous remote work for recovery if required.

The final GitOps branch was then synchronized with the desired implementation.

---

# 5. Deploying Through Argo CD

The Argo CD Application was created using:

```bash
kubectl apply -f argocd/application.yml
```

Initial state:

```text
NAME      SYNC STATUS   HEALTH STATUS
bankapp   OutOfSync     Missing
```

Argo CD automatically detected the desired state and started synchronization.

Final state:

```text
Synced | Healthy
```

Verification:

```bash
kubectl get application bankapp -n argocd \
  -o jsonpath='{.status.sync.status}{" | "}{.status.health.status}{"\n"}'
```

Result:

```text
Synced | Healthy
```

![Argo CD Synced and Healthy](docs/screenshots/gitops/01-argocd-synced-healthy.png)

---

# 6. EKS Workload Verification

The GitOps-managed workloads were verified on Amazon EKS.

```bash
kubectl get pods -n bankapp -o wide
```

Final workloads:

```text
bankapp          1/1 Running
bankapp-mysql    1/1 Running
bankapp-ollama   1/1 Running
```

Deployments were also checked:

```bash
kubectl get deployments -n bankapp
```

All three deployments became available.

![EKS BankApp Pods Running](docs/screenshots/gitops/02-eks-bankapp-pods-running.png)

---

# 7. Debugging Ollama Startup

During deployment, the Ollama pod temporarily remained in:

```text
ContainerCreating
```

The pod was investigated using:

```bash
kubectl describe pod bankapp-ollama-6689dc77fd-8bt5m \
  -n bankapp
```

The events showed:

```text
Successfully assigned
AttachVolume.Attach succeeded
Pulling image "ollama/ollama:latest"
Successfully pulled image
Container created
Container started
```

The Ollama image was approximately 3.2 GB, so the initial image pull required additional time.

After the image was downloaded and the readiness checks passed:

```text
bankapp-ollama   1/1   Running
```

Argo CD subsequently changed from:

```text
Synced | Progressing
```

to:

```text
Synced | Healthy
```

---

# 8. Envoy Gateway

Envoy Gateway was used as the Kubernetes Gateway API implementation.

GatewayClass:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: envoy-gateway
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

Gateway validation:

```bash
kubectl get gateway -n bankapp
```

Result:

```text
NAME              CLASS           PROGRAMMED
bankapp-gateway   envoy-gateway   True
```

The Gateway received an AWS Load Balancer address.

![Envoy Gateway Programmed](docs/screenshots/gitops/03-envoy-gateway-programmed.png)

---

# 9. Fixing the HTTPRoute Backend

An important issue was discovered after moving from the manually installed Helm release to the Argo CD-managed release.

The HTTPRoute was still targeting:

```text
bankapp-dev-service
```

But the Argo CD-managed Helm deployment created:

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

Applied with:

```bash
kubectl apply -f k8s/gateway.yml
```

Verification:

```bash
kubectl get httproute bankapp-route -n bankapp \
  -o jsonpath='{.spec.rules[*].backendRefs[*].name}{"\n"}'
```

Result:

```text
bankapp-service
```

The route reported:

```text
Accepted:     True
ResolvedRefs: True
```

---

# 10. Service Endpoint Verification

The BankApp service was verified:

```bash
kubectl get svc bankapp-service -n bankapp
```

The EndpointSlice was also checked:

```bash
kubectl get endpointslice -n bankapp \
  -l kubernetes.io/service-name=bankapp-service
```

This confirmed that the service correctly resolved to the running BankApp pod.

---

# 11. TLS with cert-manager and Let's Encrypt

TLS certificate management was implemented using cert-manager.

Certificate verification:

```bash
kubectl get certificate -n bankapp
```

Result:

```text
NAME          READY   SECRET
bankapp-tls   True    bankapp-tls
```

The CertificateRequest was also successful:

```text
APPROVED: True
READY:    True
ISSUER:   letsencrypt-prod
```

The ACME order reached:

```text
STATE: valid
```

![Let's Encrypt Certificate Ready](docs/screenshots/gitops/04-letsencrypt-certificate-ready.png)

---

# 12. HTTPS Verification

The BankApp endpoint was tested using:

```bash
curl -I https://13.206.8.151.nip.io
```

Response:

```text
HTTP/2 302
location: https://13.206.8.151.nip.io/login
strict-transport-security: max-age=31536000 ; includeSubDomains
```

This confirmed:

- HTTPS connectivity works
- TLS termination works
- Envoy Gateway routing works
- BankApp is reachable
- Authentication redirects users to `/login`

![BankApp HTTPS Login](docs/screenshots/gitops/05-bankapp-https-login.png)

---

# 13. Removing the Old Manual Helm Deployment

Before completing the GitOps migration, both deployments existed:

```text
bankapp-dev
bankapp
```

The old manually installed Helm release was removed:

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

This is important because GitOps should have a single source of truth.

Argo CD now owns the active application deployment.

---

# 14. Final GitOps Verification

The complete deployment was verified with:

```bash
git status -sb

kubectl get application bankapp -n argocd \
  -o jsonpath='{.status.sync.status}{" | "}{.status.health.status}{"\n"}'

kubectl get pods -n bankapp

kubectl get gateway,httproute -n bankapp

kubectl get certificate -n bankapp

curl -I https://13.206.8.151.nip.io
```

Final result:

```text
Git Branch:          Synced with origin/feat/gitops
Argo CD Sync:        Synced
Argo CD Health:      Healthy
BankApp:             Running
MySQL:               Running
Ollama:              Running
Envoy Gateway:       Programmed
TLS Certificate:     Ready
HTTPS:               Working
```

---

# 15. Git Commit History

Important commits completed during the GitOps implementation:

```text
1055fb8 docs(gitops): document end-to-end EKS deployment verification
d28b6af fix(gateway): route traffic to Argo CD managed service
c608d29 fix(gitops): point Argo CD to fork repository
04ca364 feat(gitops): deploy BankApp to EKS with Helm and Argo CD
```

![GitOps Commit History](docs/screenshots/gitops/06-gitops-commit-history.png)

---

# Problems Encountered and Solutions

## Problem 1 — Argo CD Application Missing

Initially:

```bash
kubectl get applications -n argocd
```

returned:

```text
No resources found in argocd namespace.
```

### Solution

Located the existing Application manifest and updated it to use the Helm chart.

---

## Problem 2 — Wrong GitHub Repository

The project still referenced the upstream repository.

### Solution

Updated both the Git remote and Argo CD Application to:

```text
cloudwithpreetham/AI-BankApp-DevOps
```

---

## Problem 3 — Git Push Permission Denied

GitHub returned:

```text
Permission to TrainWithShubham/AI-BankApp-DevOps.git denied
```

### Solution

Authenticated with GitHub CLI and changed the remote to the personal fork.

---

## Problem 4 — Divergent Git Branches

The local branch was ahead while the remote contained additional commits.

### Solution

Fetched and inspected both histories before modifying the remote branch.

A backup branch was created to preserve the previous remote state.

---

## Problem 5 — Duplicate GatewayClass

`GatewayClass` existed both inside `gateway.yml` and as a separate `gatewayclass.yml`.

### Solution

Removed the duplicate definition from `gateway.yml` and kept:

```text
k8s/gatewayclass.yml
```

as the dedicated GatewayClass manifest.

---

## Problem 6 — Ollama ContainerCreating

The Ollama pod initially remained in `ContainerCreating`.

### Root Cause

The large `ollama/ollama:latest` image was still being downloaded.

### Solution

Inspected Kubernetes events instead of immediately changing the deployment.

The container started successfully after the image pull completed.

---

## Problem 7 — Gateway Routed to Old Service

The HTTPRoute still referenced:

```text
bankapp-dev-service
```

after the GitOps migration.

### Solution

Changed the backend to:

```text
bankapp-service
```

and validated `Accepted=True` and `ResolvedRefs=True`.

---

## Problem 8 — Duplicate Manual and GitOps Deployments

Both `bankapp-dev` and `bankapp` workloads existed simultaneously.

### Solution

Removed the manually installed release:

```bash
helm uninstall bankapp-dev -n bankapp
```

The cluster now contains only the Argo CD-managed application stack.

---

# Key Lessons Learned

### 1. Git Should Be the Source of Truth

A GitOps-managed environment should avoid unmanaged manual deployment changes.

Argo CD continuously reconciles the cluster with the desired state stored in Git.

### 2. Validate Helm Before GitOps Deployment

Commands such as:

```bash
helm lint
helm template
kubectl apply --dry-run=server
```

help catch configuration errors before Argo CD deploys them.

### 3. Synced Does Not Always Mean Healthy

During the Ollama startup:

```text
Synced | Progressing
```

was expected.

`Synced` means the desired manifests were applied.

`Healthy` means the resulting workloads reached their expected operational state.

### 4. Kubernetes Events Are Critical for Debugging

When a pod remains in `Pending` or `ContainerCreating`, always inspect:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

before modifying the deployment.

### 5. Service Names Matter During GitOps Migration

Moving from a manual Helm release to an Argo CD-managed release changed the generated service name.

Gateway routing therefore had to be updated from:

```text
bankapp-dev-service
```

to:

```text
bankapp-service
```

### 6. Preserve Git History Before Destructive Operations

Before rewriting divergent Git history, create a backup reference.

Example:

```bash
git branch backup/old-remote-gitops <commit>
```

This provides a recovery point if commits need to be restored.

### 7. Deployment Evidence Improves Portfolio Quality

The final implementation was documented with screenshots showing:

1. Argo CD Synced and Healthy
2. EKS workloads running
3. Envoy Gateway programmed
4. Let's Encrypt certificate ready
5. BankApp accessible over HTTPS
6. GitOps commit history

---

# Final Deployment Status

| Component | Status |
|---|---|
| Amazon EKS | Running |
| Helm Chart | Valid |
| Argo CD | Synced |
| Argo CD Health | Healthy |
| BankApp | Running |
| MySQL | Running |
| Ollama | Running |
| Envoy Gateway | Programmed |
| HTTPRoute | Accepted |
| cert-manager | Working |
| Let's Encrypt Certificate | Ready |
| HTTPS | Working |
| GitOps Repository | Synchronized |
| Documentation | Complete |

---

# Day 82 Result

Day 82 completed the transition from a manually managed Kubernetes deployment to an **end-to-end GitOps delivery architecture on Amazon EKS**.

The final deployment flow is:

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
cert-manager + Let's Encrypt
   |
   v
HTTPS BankApp
```

The application is now deployed declaratively, automatically reconciled by Argo CD, exposed through Envoy Gateway, secured with TLS, and fully documented with deployment evidence.

**Day 82 Status: COMPLETE**
