# Day 83 --- Production-Ready GitOps Deployment on Amazon EKS

## Overview

Day 83 focused on validating and stabilizing the production-style
deployment of **AI BankApp** on Amazon EKS.

The application stack combines Kubernetes, Helm, Argo CD, Envoy Gateway,
cert-manager, Horizontal Pod Autoscaling, persistent storage, and an
Ollama-powered AI banking assistant.

This lab also involved troubleshooting several realistic
distributed-system issues, including DNS/TLS routing, AI request
timeouts, session persistence across replicas, and conflicts between
Argo CD reconciliation and HPA scaling.

---

## Architecture

```text
GitHub — feat/gitops
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
        +-- BankApp Pods x2
        |      |
        |      +-- HPA (2–4 replicas)
        |      +-- Spring Boot
        |
        +-- MySQL
        |      |
        |      +-- PersistentVolumeClaim
        |      +-- Amazon EBS
        |
        +-- Ollama
        |      |
        |      +-- TinyLlama
        |      +-- PersistentVolumeClaim
        |
        +-- Envoy Gateway
               |
               +-- Gateway API
               +-- HTTPRoute
               +-- HTTPS / TLS
               +-- Let's Encrypt
               +-- Cookie-based session affinity
```

---

## Technologies Used

- Amazon EKS
- Kubernetes
- Helm
- Argo CD
- Envoy Gateway
- Kubernetes Gateway API
- cert-manager
- Let's Encrypt
- Horizontal Pod Autoscaler
- Metrics Server
- Amazon EBS
- MySQL
- Ollama
- TinyLlama
- Spring Boot
- Git
- GitHub

---

## 1. Verify the EKS Cluster

The cluster was first validated to ensure all worker nodes were healthy.

```bash
kubectl get nodes -o wide
kubectl cluster-info
```

Three Amazon Linux worker nodes were successfully connected to the EKS
control plane.

---

## 2. Verify BankApp Workloads

```bash
kubectl get pods -n bankapp
kubectl get all -n bankapp
```

The namespace contained the primary application components:

- BankApp
- MySQL
- Ollama

All workloads reached the `Running` state.

---

## 3. Verify Persistent Storage

```bash
kubectl get pv
kubectl get pvc -n bankapp
```

Persistent storage was successfully provisioned for:

Workload Capacity Access Mode

---

MySQL 2 Gi RWO
Ollama 5 Gi RWO

Both PVCs reached the `Bound` state.

---

## 4. Configure Horizontal Pod Autoscaling

The BankApp container defines CPU and memory requests and limits:

```text
Requests:
  CPU:    250m
  Memory: 256Mi

Limits:
  CPU:    500m
  Memory: 512Mi
```

The HPA configuration uses CPU utilization:

```yaml
minReplicas: 2
maxReplicas: 4

metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

Apply the HPA:

```bash
kubectl apply -f k8s/hpa.yml
```

Verify:

```bash
kubectl get hpa -n bankapp
kubectl top nodes
kubectl top pods -n bankapp
```

Final state:

```text
TARGET        1% / 70%
MINPODS       2
MAXPODS       4
REPLICAS      2
```

---

## 5. Validate Envoy Gateway

Envoy Gateway was used instead of a traditional Kubernetes Ingress.

```bash
helm list -n envoy-gateway-system
kubectl get gatewayclass
kubectl get gateway -n bankapp
kubectl get httproute -n bankapp
kubectl get pods -n envoy-gateway-system
```

The GatewayClass was accepted and the BankApp Gateway reached:

```text
PROGRAMMED=True
```

---

## 6. Fix the nip.io Routing Problem

The original application hostname was:

```text
13.206.8.151.nip.io
```

However, the Envoy Gateway load balancer resolved to:

```text
13.235.189.104
```

DNS verification:

```bash
dig +short "$APP_URL"
dig +short 13.206.8.151.nip.io
```

The mismatch meant the hostname was not targeting the active Gateway
address.

The Gateway configuration was updated to:

```text
13.235.189.104.nip.io
```

Verification:

```bash
dig +short 13.235.189.104.nip.io
```

Result:

```text
13.235.189.104
```

---

## 7. Reissue the TLS Certificate

After changing the hostname, cert-manager automatically requested a new
certificate.

```bash
kubectl get certificate -n bankapp -w
```

The certificate transitioned to:

```text
READY=True
```

Additional verification:

```bash
kubectl get certificaterequest,order,challenge -n bankapp
```

The ACME order became:

```text
STATE=valid
```

TLS was then validated with:

```bash
echo | openssl s_client \
  -connect ${BANKAPP_HOST}:443 \
  -servername ${BANKAPP_HOST} 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

The certificate was successfully issued by Let's Encrypt.

---

## 8. Verify HTTPS

Application health:

```bash
curl -s https://$BANKAPP_HOST/actuator/health | python3 -m json.tool
```

Result:

```json
{
  "status": "UP",
  "groups": ["liveness", "readiness"]
}
```

HTTPS routing was therefore working successfully through Envoy Gateway.

---

## 9. Validate Ollama Connectivity

The BankApp ConfigMap provides the internal Ollama endpoint:

```yaml
OLLAMA_URL: http://bankapp-ollama:11434
```

Verify available models:

```bash
kubectl run ollama-test \
  --rm -it \
  --restart=Never \
  --image=curlimages/curl \
  -n bankapp \
  -- curl -s http://bankapp-ollama:11434/api/tags
```

The cluster returned:

```text
tinyllama:latest
```

This confirmed:

```text
BankApp Namespace
      |
      +--> bankapp-ollama:11434
                |
                +--> TinyLlama
```

---

## 10. Test Ollama Generation

A direct generation request was executed inside the cluster:

```bash
kubectl run ollama-generate-test \
  --rm -it \
  --restart=Never \
  --image=curlimages/curl \
  -n bankapp \
  -- curl -s \
  -X POST http://bankapp-ollama:11434/api/generate \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "tinyllama",
    "prompt": "What is compound interest? Answer in one sentence.",
    "stream": false
  }'
```

Ollama successfully generated a response.

This proved that the AI model and Kubernetes service networking were
functioning correctly.

---

## 11. Test the Ollama Chat API

BankApp uses Ollama's `/api/chat` endpoint.

A direct API test was performed:

```bash
kubectl run ollama-chat-test \
  --rm -it \
  --restart=Never \
  --image=curlimages/curl \
  -n bankapp \
  -- curl -s \
  -X POST http://bankapp-ollama:11434/api/chat \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "tinyllama",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful banking assistant. Keep answers short."
      },
      {
        "role": "user",
        "content": "What is compound interest?"
      }
    ],
    "stream": false
  }'
```

The model successfully returned an assistant response.

---

## 12. Diagnose the AI Gateway Timeout

Although Ollama worked internally, requests through the browser
initially failed.

Envoy access logs showed:

```text
POST /api/chat
response_code: 504
response_code_details: response_timeout
duration: 15010
```

This identified the real problem:

```text
Browser
   |
   v
Envoy Gateway
   |
   X 15-second timeout
   |
BankApp --> Ollama
```

The AI inference could take longer than Envoy's effective request
timeout.

---

## 13. Increase the HTTPRoute Request Timeout

The HTTPRoute was updated with:

```yaml
timeouts:
  request: 60s
```

Server-side validation:

```bash
kubectl apply --dry-run=server -f k8s/gateway.yml
```

Apply and verify:

```bash
kubectl get httproute bankapp-route -n bankapp \
  -o jsonpath='{.spec.rules[0].timeouts}'
echo
```

Result:

```json
{ "request": "60s" }
```

After the change, Envoy logged:

```text
POST /api/chat
response_code: 200
duration: 5940
```

The AI response successfully completed through the Gateway.

---

## 14. Configure Session Persistence

BankApp uses application sessions.

With multiple replicas, requests could reach different pods and break
login or CSRF state.

A `BackendTrafficPolicy` was configured using cookie-based consistent
hashing:

```yaml
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: BackendTrafficPolicy

spec:
  loadBalancer:
    type: ConsistentHash
    consistentHash:
      type: Cookie
      cookie:
        name: BANKAPP_AFFINITY
        ttl: 3600s
```

Verify:

```bash
kubectl get backendtrafficpolicy bankapp-session -n bankapp -o yaml
```

The policy was accepted by Envoy Gateway.

---

## 15. Diagnose the Argo CD vs HPA Conflict

The Deployment repeatedly scaled between one and two replicas.

Kubernetes events showed:

```text
Scaled down replica set from 2 to 1
Scaled up replica set from 1 to 2
```

The cause was competing controllers:

```text
Argo CD / Helm
replicaCount: 1
       |
       v
Deployment replicas = 1

HPA
minReplicas: 2
       |
       v
Deployment replicas = 2
```

Argo CD had:

```yaml
automated:
  prune: true
  selfHeal: true
```

Therefore Argo CD continuously restored the Helm-defined replica count
while HPA continuously restored its minimum replica count.

---

## 16. Separate Argo CD and HPA Ownership

The Argo CD Application was updated to ignore the Deployment replica
field:

```yaml
ignoreDifferences:
  - group: apps
    kind: Deployment
    name: bankapp
    namespace: bankapp
    jsonPointers:
      - /spec/replicas
```

The following sync option was also enabled:

```yaml
- RespectIgnoreDifferences=true
```

This establishes clear ownership:

```text
Argo CD
  |
  +--> Deployment configuration
  +--> image
  +--> environment
  +--> probes
  +--> resources

HPA
  |
  +--> spec.replicas
```

After this change, the Deployment remained stable at two replicas.

---

## 17. Final Argo CD Verification

```bash
kubectl get application bankapp -n argocd
```

Final result:

```text
SYNC STATUS     Synced
HEALTH STATUS   Healthy
```

Verify the Git revision:

```bash
kubectl get application bankapp -n argocd \
  -o jsonpath='{.status.sync.revision}'
echo
```

Final revision:

```text
8ac0d953b4203ebd9f3a056dceb1bdf7345528b0
```

---

## 18. Final Kubernetes Health

```bash
kubectl get pods -n bankapp -l app=bankapp -o wide
kubectl get hpa bankapp-hpa -n bankapp
kubectl get deployment bankapp -n bankapp
```

Final state:

```text
BankApp Pods   2/2 Running
Deployment     2/2 Ready
HPA            2 replicas
Argo CD        Synced / Healthy
```

---

## 19. Final AI Assistant Test

The deployed BankApp AI Assistant was tested through the browser.

Example question:

```text
What is compound interest?
```

The request successfully travelled through:

```text
Browser
   |
 HTTPS
   |
Envoy Gateway
   |
HTTPRoute
   |
BankApp
   |
ChatController
   |
ChatService
   |
Ollama
   |
TinyLlama
   |
BankApp
   |
Browser
```

The AI-generated response was successfully displayed in the application
UI.

---

## Troubleshooting Summary

---

Problem Root Cause Resolution

---

Incorrect application nip.io resolved to an Updated hostname to
hostname old IP active Gateway IP

TLS needed reissuance Gateway hostname cert-manager obtained
changed new Let's Encrypt
certificate

AI request returned 504 AI inference exceeded Increased HTTPRoute
Gateway timeout request timeout to 60
seconds

Session instability Requests could reach Added cookie-based
different BankApp consistent hashing
replicas

Replica count Argo CD and HPA both Configured Argo CD to
oscillated controlled replicas ignore `/spec/replicas`

Local kubectl failed Local kubeconfig Connected to the EC2
referenced an obsolete management host with
EKS cluster the correct cluster
context

---

---

## Git Commits

Gateway stabilization:

```text
d825e12 fix(gateway): stabilize BankApp routing and AI requests
```

GitOps/HPA ownership:

```text
8ac0d95 fix(gitops): prevent Argo CD from overriding HPA replicas
```

Both commits were pushed to:

```text
feat/gitops
```

---

## Recommended Screenshots

Store evidence under:

```text
docs/screenshots/
```

Recommended files:

```text
day-83-01-eks-nodes.png
day-83-02-bankapp-workloads.png
day-83-03-persistent-storage.png
day-83-04-hpa-running.png
day-83-05-envoy-gateway.png
day-83-06-tls-certificate-ready.png
day-83-07-ollama-tinyllama-test.png
day-83-08-ai-chat-success.png
day-83-09-argocd-synced-healthy.png
day-83-10-final-production-health.png
```

The most important portfolio screenshots are:

- AI Assistant successfully answering through the deployed HTTPS
  application.
- Argo CD showing `Synced` and `Healthy`.
- Two healthy BankApp replicas with the HPA active.
- Gateway and TLS certificate in ready state.

---

## Key Lessons

1.  A healthy AI model does not guarantee a healthy end-to-end AI
    application. Gateway and network timeouts must account for inference
    latency.

2.  Multiple application replicas require deliberate session-state
    handling when the application stores sessions in memory.

3.  GitOps controllers and Kubernetes autoscalers must have clearly
    separated field ownership.

4.  `kubectl events`, Envoy access logs, direct service testing, and
    incremental isolation are powerful tools for debugging distributed
    applications.

5.  Production-style Kubernetes troubleshooting requires validating
    every layer independently:

```text
DNS
 ↓
TLS
 ↓
Gateway
 ↓
Route
 ↓
Service
 ↓
Pod
 ↓
Application
 ↓
AI dependency
```

---

## Day 83 Result

**Status: COMPLETE**

Day 83 delivered a stable GitOps-managed AI BankApp deployment with:

- Amazon EKS
- Helm-based application deployment
- Argo CD continuous reconciliation
- Horizontal Pod Autoscaling
- Persistent MySQL and Ollama storage
- Envoy Gateway
- HTTPS with Let's Encrypt
- Session-aware load balancing
- AI-aware Gateway timeout configuration
- Ollama + TinyLlama integration
- End-to-end AI Assistant functionality

The final platform is healthy, autoscaling-aware, GitOps-managed,
HTTPS-enabled, and validated end-to-end.
