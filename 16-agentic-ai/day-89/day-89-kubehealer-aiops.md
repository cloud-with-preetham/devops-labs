# Day 89 – AI-Powered Kubernetes Self-Healing with Temporal & Anthropic

## Project Overview

On **Day 89** of my **90 Days of DevOps** journey, I built and tested an **AI-powered Kubernetes Self-Healing system (KubeHealer)** that automatically diagnoses unhealthy Kubernetes workloads using **Anthropic Claude** and performs safe remediation through **Temporal Workflows**.

The project continuously analyzes Kubernetes pod failures, identifies root causes using AI, and executes automated recovery actions while safely skipping issues that require manual intervention.

---

## Objectives

- Learn Temporal durable workflows
- Integrate Anthropic Claude API
- Diagnose Kubernetes pod failures using AI
- Automatically remediate safe failures
- Skip risky operations requiring human intervention
- Build an AI-powered Kubernetes SRE workflow

---

# Architecture

```
                   +----------------------+
                   |   Kubernetes Cluster |
                   +----------+-----------+
                              |
                              |
                    Detect unhealthy pods
                              |
                              ▼
                  +-----------------------+
                  |   Temporal Workflow   |
                  +-----------+-----------+
                              |
          +-------------------+-------------------+
          |                                       |
          ▼                                       ▼
+--------------------+                +----------------------+
| Diagnose Activity  |                | Execute Fix Activity |
| (Anthropic Claude) |                | Kubernetes API       |
+----------+---------+                +----------+-----------+
           |                                       |
           ▼                                       ▼
   Root Cause Analysis                 Patch Deployment /
                                       Restart Pod /
                                       Skip Unsafe Fix
```

---

# Project Structure

```
kubehealer/
├── activities/
│   ├── k8s_activities.py
│   └── llm_activities.py
├── workflows/
│   └── healer_workflow.py
├── chaos/
│   ├── broken-image.yaml
│   ├── oom-pod.yaml
│   └── missing-config.yaml
├── worker.py
├── starter.py
├── models.py
├── requirements.txt
├── setup.sh
└── README.md
```

---

# Technologies Used

- Python
- Kubernetes
- Kind
- Docker
- Temporal
- Anthropic Claude API
- Kubernetes Python Client
- Temporal Python SDK

---

# Prerequisites

- Docker
- Kind
- kubectl
- Python 3.12+
- Temporal CLI
- Anthropic API Key

---

# Environment Setup

Create a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

# Configure Environment Variables

Create a `.env` file:

```env
ANTHROPIC_API_KEY=your-api-key
ANTHROPIC_MODEL=claude-sonnet-5
```

> **Note:** The project now supports configurable Anthropic model selection using the `ANTHROPIC_MODEL` environment variable.

---

# Verify Tools

```bash
docker --version
kind version
kubectl version --client
python3 --version
temporal --version
```

---

# Create Kubernetes Cluster

```bash
kind create cluster --name kubehealer-demo
```

Verify:

```bash
kubectl get nodes
```

---

# Deploy Broken Applications

Run:

```bash
chmod +x setup.sh
./setup.sh
```

This deploys intentionally broken applications:

- Invalid Docker image
- Out-of-memory application
- Missing ConfigMap

---

# Verify Resources

```bash
kubectl get deployments
kubectl get pods
```

Example:

```
NAME         READY
web-app      0/1
memory-hog   0/1
config-app   0/1
```

---

# Start Temporal

```bash
temporal server start-dev
```

---

# Start Worker

```bash
python worker.py
```

Expected:

```
[OK] Anthropic API key
[OK] Kubernetes cluster

KubeHealer worker started. Waiting for tasks...
```

---

# Execute Workflow

In another terminal:

```bash
python starter.py
```

Example output:

```
Starting KubeHealer workflow...

Healed 3/5 pods

config-app ... skipped
memory-hog ... patch_resources
web-app ... fix_image
```

---

# AI Diagnosis Results

| Workload | Issue | AI Action |
|----------|------|-----------|
| web-app | Invalid Image | Fix Image |
| memory-hog | OOM | Increase Memory |
| config-app | Missing ConfigMap | Skip |

---

# Automated Remediation

Successfully performed:

- Corrected invalid container image
- Increased memory limits
- Patched Kubernetes Deployment
- Skipped unsafe remediation
- Logged workflow execution

---

# Validation

Verify cluster state:

```bash
kubectl get deployments
kubectl get pods
```

Expected:

```
web-app        Running
memory-hog     Running
config-app     CreateContainerConfigError
```

The missing ConfigMap remains unresolved intentionally because it requires manual intervention.

---

# Problems Encountered

## 1. Invalid Anthropic API Key

**Issue**

```
401 AuthenticationError
```

**Solution**

Generated an official Anthropic API key and configured it in the local environment.

---

## 2. Deprecated Claude Model

**Issue**

```
404 model not found
```

**Solution**

Updated the project from:

```
claude-sonnet-4-20250514
```

to

```
claude-sonnet-5
```

and introduced configurable model selection using:

```python
MODEL = os.getenv("ANTHROPIC_MODEL", "claude-sonnet-5")
```

---

## 3. Temporal Connection Refused

**Issue**

```
Connection refused localhost:7233
```

**Solution**

Started the Temporal development server before launching the worker.

---

## 4. Incorrect Kubernetes State

**Issue**

Standalone Pods existed instead of Deployments.

**Solution**

Recreated the Kind cluster and initialized the environment using:

```bash
./setup.sh
```

---

# Key Learnings

- Temporal durable workflows
- AI-assisted Kubernetes troubleshooting
- Anthropic API integration
- Kubernetes Python client
- Safe automated remediation
- Production debugging techniques
- Environment-driven configuration
- Kubernetes Deployment patching

---

# Best Practices Implemented

- Environment variable configuration
- Secret management using `.env`
- Configurable AI model selection
- Safe validation before remediation
- Human approval for non-remediable issues
- Version-controlled infrastructure

---

# Screenshots

```
docs/screenshots/
├── 01-kind-cluster.png
├── 02-temporal-ui.png
├── 03-worker-running.png
├── 04-workflow-success.png
├── 05-kubectl-get-pods.png
├── 06-kubectl-get-deployments.png
└── 07-anthropic-success.png
```

---

# Cleanup

Delete resources:

```bash
kubectl delete -f chaos/
```

Delete cluster:

```bash
kind delete cluster --name kubehealer-demo
```

---

# Conclusion

This project demonstrates how AI can be integrated into Kubernetes operations to automate incident diagnosis and safe remediation using Temporal workflows and Anthropic Claude. By combining durable orchestration with intelligent analysis, KubeHealer showcases a practical implementation of AI-assisted Site Reliability Engineering (SRE) principles while maintaining safety through controlled remediation and manual escalation for high-risk scenarios.

---

## Repository

```
projects/kubehealer
```

---

**Day 89/90 Completed**
