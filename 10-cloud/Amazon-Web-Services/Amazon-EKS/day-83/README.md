# Day 83 --- Production-Ready GitOps Deployment on Amazon EKS

## Overview

Day 83 focused on stabilizing and validating the **AI BankApp** as a production-style Kubernetes workload on **Amazon EKS**.

The deployment combines GitOps, autoscaling, persistent storage, secure external routing, and an in-cluster AI assistant powered by Ollama and TinyLlama.

The main work was not simply deploying the application. It involved troubleshooting real distributed-system issues across DNS, TLS, Gateway API, session persistence, AI inference timeouts, HPA, and Argo CD reconciliation.

## Architecture

```text
GitHub (feat/gitops)
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
        +-- BankApp x2 <------ HPA (2–4)
        |
        +-- MySQL
        |     |
        |     +-- PVC / Amazon EBS
        |
        +-- Ollama
        |     |
        |     +-- TinyLlama
        |     +-- PVC / Amazon EBS
        |
        +-- Envoy Gateway
              |
              +-- Gateway API
              +-- HTTPRoute
              +-- HTTPS / TLS
              +-- Let's Encrypt
              +-- Cookie-based session affinity
```

## Tech Stack

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
- Spring Boot
- Ollama
- TinyLlama
- GitHub

## EKS Cluster Verification

Verify the worker nodes and control plane:

```bash
kubectl get nodes -o wide
kubectl cluster-info
```

The EKS cluster was running with three healthy worker nodes.

## Application Workloads

Verify the BankApp namespace:

```bash
kubectl get all -n bankapp
```

Core workloads:

```text
bankapp
bankapp-mysql
bankapp-ollama
```

All application components successfully reached the `Running` state.

## Persistent Storage

Verify persistent volumes:

```bash
kubectl get pv
kubectl get pvc -n bankapp
```

Storage configuration:

Component Capacity Access Mode Status

---

MySQL 2 Gi RWO Bound
Ollama 5 Gi RWO Bound

This provides persistent storage for database data and the local AI
model.

## Horizontal Pod Autoscaling

BankApp resource configuration:

```text
Requests:
CPU:    250m
Memory: 256Mi

Limits:
CPU:    500m
Memory: 512Mi
```

HPA configuration:

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

Apply and verify:

```bash
kubectl apply -f k8s/hpa.yml

kubectl get hpa -n bankapp
kubectl top nodes
kubectl top pods -n bankapp
```

Final state:

```text
CPU Target: 70%
Minimum replicas: 2
Maximum replicas: 4
Current replicas: 2
```

## Envoy Gateway

Envoy Gateway provides external traffic management using Kubernetes
Gateway API.

Verify:

```bash
kubectl get gatewayclass

kubectl get gateway -n bankapp

kubectl get httproute -n bankapp

kubectl get pods -n envoy-gateway-system
```

The Gateway was successfully programmed and exposed through an AWS load
balancer.

## DNS Routing Fix

The original `nip.io` hostname pointed to an outdated IP:

```text
13.206.8.151.nip.io
```

The active Envoy Gateway resolved to:

```text
13.235.189.104
```

The hostname was therefore updated to:

```text
13.235.189.104.nip.io
```

Verify:

```bash
dig +short 13.235.189.104.nip.io
```

Expected:

```text
13.235.189.104
```

This aligned DNS routing with the active Envoy Gateway endpoint.

## HTTPS with Let's Encrypt

cert-manager automatically requested a new certificate after the
hostname change.

Verify:

```bash
kubectl get certificate -n bankapp
```

Expected:

```text
READY=True
```

Inspect ACME resources:

```bash
kubectl get certificaterequest,order,challenge -n bankapp
```

Validate the certificate:

```bash
echo | openssl s_client \
  -connect ${BANKAPP_HOST}:443 \
  -servername ${BANKAPP_HOST} 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

The certificate was successfully issued by **Let's Encrypt**.

## HTTPS Application Health

Verify the deployed application through the Gateway:

```bash
curl -s https://$BANKAPP_HOST/actuator/health | python3 -m json.tool
```

Expected response:

```json
{
  "status": "UP",
  "groups": ["liveness", "readiness"]
}
```

## Ollama Connectivity

BankApp communicates with Ollama through the internal Kubernetes
service:

```text
http://bankapp-ollama:11434
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

Available model:

```text
tinyllama:latest
```

## AI Model Test

A direct generation request was used to verify inference:

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

TinyLlama successfully generated a response.

A second test verified Ollama's `/api/chat` endpoint, which is used by
BankApp.

## AI Gateway Timeout Troubleshooting

The AI service worked internally, but browser requests initially failed.

Envoy access logs revealed:

```text
POST /api/chat
response_code: 504
response_code_details: response_timeout
duration: 15010
```

The problem was therefore not Ollama itself.

The request path was:

```text
Browser
   |
   v
Envoy Gateway
   |
   X request timeout
   |
BankApp
   |
   v
Ollama
```

AI inference occasionally required more time than the Gateway allowed.

## HTTPRoute Timeout Fix

The BankApp HTTPRoute was updated:

```yaml
timeouts:
  request: 60s
```

Validate the manifest:

```bash
kubectl apply --dry-run=server -f k8s/gateway.yml
```

Verify the live route:

```bash
kubectl get httproute bankapp-route -n bankapp \
  -o jsonpath='{.spec.rules[0].timeouts}'
echo
```

Expected:

```json
{ "request": "60s" }
```

After the change, Envoy reported:

```text
POST /api/chat
response_code: 200
```

The AI response successfully reached the browser.

## Session Persistence

BankApp uses application sessions.

With multiple BankApp replicas, requests from the same browser could
otherwise reach different pods and break login or CSRF state.

Envoy Gateway was configured with a `BackendTrafficPolicy`:

```yaml
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
kubectl get backendtrafficpolicy bankapp-session \
  -n bankapp \
  -o yaml
```

The policy was accepted by the Envoy Gateway controller.

## Argo CD vs HPA Conflict

A second production-style issue appeared after enabling HPA.

The Deployment repeatedly moved between one and two replicas.

The reason:

```text
Helm / Argo CD
replicaCount = 1
      |
      v
Deployment = 1

HPA
minReplicas = 2
      |
      v
Deployment = 2
```

Because Argo CD self-healing was enabled, Argo CD continuously attempted
to restore the Git-defined replica count while HPA continuously restored
its minimum replica count.

## Separate Controller Ownership

Argo CD was configured to ignore the Deployment replica field:

```yaml
ignoreDifferences:
  - group: apps
    kind: Deployment
    name: bankapp
    namespace: bankapp
    jsonPointers:
      - /spec/replicas
```

The sync option was also added:

```yaml
- RespectIgnoreDifferences=true
```

This creates a clean ownership model:

```text
Argo CD
  |
  +-- image
  +-- configuration
  +-- probes
  +-- resources
  +-- deployment definition

HPA
  |
  +-- spec.replicas
```

After this change, the Deployment remained stable at two replicas.

## Final GitOps Verification

```bash
kubectl get application bankapp -n argocd
```

Final state:

```text
SYNC STATUS     Synced
HEALTH STATUS   Healthy
```

Verify the deployed Git revision:

```bash
kubectl get application bankapp -n argocd \
  -o jsonpath='{.status.sync.revision}'
echo
```

Revision:

```text
8ac0d953b4203ebd9f3a056dceb1bdf7345528b0
```

## Final Platform State

```bash
kubectl get pods -n bankapp -l app=bankapp -o wide

kubectl get hpa bankapp-hpa -n bankapp

kubectl get deployment bankapp -n bankapp
```

Final health:

Component State

---

Argo CD Synced / Healthy
BankApp Deployment 2/2 Ready
HPA 2 replicas, range 2--4
MySQL Running
Ollama Running
TinyLlama Responding
Envoy Gateway Programmed
HTTPS Working
Let's Encrypt Certificate Ready
AI Assistant Working

## End-to-End AI Flow

```text
Browser
   |
   | HTTPS
   v
Envoy Gateway
   |
   v
HTTPRoute
   |
   v
BankApp Service
   |
   v
BankApp Pod
   |
   v
ChatController
   |
   v
ChatService
   |
   v
Ollama Service
   |
   v
TinyLlama
   |
   v
AI Response
```

The BankApp AI Assistant successfully answered questions through the
production-style HTTPS endpoint.

## Troubleshooting Summary

---

Issue Root Cause Fix

---

Wrong external hostname `nip.io` pointed to an Updated hostname to
old IP active Gateway IP

TLS hostname changed Existing certificate cert-manager issued a
targeted old hostname new Let's Encrypt
certificate

`/api/chat` returned AI inference exceeded Increased HTTPRoute
504 Gateway timeout request timeout to 60s

Login/session Multiple replicas could Added cookie-based
instability receive the same user's consistent hashing
requests

Deployment oscillated 1 Argo CD and HPA both Ignored
↔ 2 managed replicas `/spec/replicas` in
Argo CD

Local `kubectl` failed Local context Used the EC2 management
referenced an obsolete host with the active
EKS cluster EKS context

---

## Important Git Commits

```text
d825e12 fix(gateway): stabilize BankApp routing and AI requests

8ac0d95 fix(gitops): prevent Argo CD from overriding HPA replicas
```

Branch:

```text
feat/gitops
```

## Useful Final Verification Commands

```bash
echo "--- Argo CD ---"
kubectl get application bankapp -n argocd

echo "--- Pods ---"
kubectl get pods -n bankapp

echo "--- HPA ---"
kubectl get hpa -n bankapp

echo "--- Gateway ---"
kubectl get gateway -n bankapp

echo "--- Certificate ---"
kubectl get certificate -n bankapp

echo "--- HTTPRoute ---"
kubectl get httproute -n bankapp
```

## Screenshots

Recommended evidence:

```text
docs/screenshots/day-83-01-eks-nodes.png
docs/screenshots/day-83-02-bankapp-workloads.png
docs/screenshots/day-83-03-persistent-storage.png
docs/screenshots/day-83-04-hpa-running.png
docs/screenshots/day-83-05-envoy-gateway.png
docs/screenshots/day-83-06-tls-certificate-ready.png
docs/screenshots/day-83-07-ollama-test.png
docs/screenshots/day-83-08-ai-chat-success.png
docs/screenshots/day-83-09-argocd-synced-healthy.png
docs/screenshots/day-83-10-final-production-health.png
```

## Key Learnings

1.  AI inference latency must be considered when configuring Gateway and
    proxy timeouts.
2.  Kubernetes applications using in-memory sessions need session-aware
    traffic distribution when scaled horizontally.
3.  GitOps controllers and autoscalers should not compete for ownership
    of the same field.
4.  Direct service testing helps isolate application problems from
    Gateway and networking problems.
5.  Envoy access logs provide valuable evidence for diagnosing HTTP
    routing and timeout failures.
6.  Production troubleshooting should validate each layer independently:

```text
DNS
 ↓
TLS
 ↓
Gateway
 ↓
HTTPRoute
 ↓
Service
 ↓
Pod
 ↓
Application
 ↓
AI dependency
```

## Outcome

Day 83 successfully delivered a production-style, GitOps-managed AI
application on Amazon EKS with:

- Automated GitOps reconciliation through Argo CD
- Helm-managed Kubernetes workloads
- Horizontal Pod Autoscaling
- Persistent MySQL and Ollama storage
- Envoy Gateway and Gateway API routing
- HTTPS using cert-manager and Let's Encrypt
- Session-aware load balancing
- AI-compatible request timeout configuration
- Ollama and TinyLlama integration
- End-to-end AI Assistant functionality

**Day 83 Status: COMPLETE**
