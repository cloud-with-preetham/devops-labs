# AI-Powered Kubernetes Self-Healing with Temporal & Anthropic

> **Day 89 – 90 Days of DevOps Challenge**

An autonomous Kubernetes self-healing system that leverages **Anthropic Claude**, **Temporal Workflows**, and the **Kubernetes Python Client** to detect, diagnose, and remediate unhealthy workloads automatically.

---

## Project Overview

KubeHealer is an AI-powered Site Reliability Engineering (SRE) project that continuously monitors Kubernetes workloads, analyzes failures using **Anthropic Claude**, and performs safe remediation through **Temporal Durable Workflows**.

Instead of relying on static rules, KubeHealer uses an LLM to understand Kubernetes failures, determine their root cause, and decide whether an issue can be fixed automatically or requires human intervention.

This project demonstrates the future of **Agentic AI for DevOps**, where AI assists engineers in operating production infrastructure safely and intelligently.

---

## Features

- AI-powered Kubernetes troubleshooting
- Temporal Durable Workflows
- Anthropic Claude integration
- Automatic Deployment patching
- Kubernetes Python Client
- Image remediation
- Memory resource remediation
- Safe validation before applying fixes
- Human escalation for unsafe scenarios
- Environment-based model configuration

---

# Architecture

```text
                   +---------------------------+
                   |     Kubernetes Cluster    |
                   +-------------+-------------+
                                 |
                                 |
                    Detect unhealthy workloads
                                 |
                                 ▼
                    +-------------------------+
                    |  Temporal Workflow      |
                    +-----------+-------------+
                                |
          +---------------------+----------------------+
          |                                            |
          ▼                                            ▼
+-------------------------+                +--------------------------+
| Diagnose Activity       |                | Execute Fix Activity     |
| Anthropic Claude        |                | Kubernetes API           |
+------------+------------+                +------------+-------------+
             |                                          |
             ▼                                          ▼
     Root Cause Analysis                    Patch Deployment /
                                            Restart Pod /
                                            Skip Unsafe Fix
```

---

# Workflow

```text
Broken Kubernetes Workload
            │
            ▼
     Cluster Scan Activity
            │
            ▼
    Diagnose using Claude AI
            │
            ▼
      AI Recommendation
            │
     ┌──────┴────────┐
     │               │
 Safe Fix      Manual Intervention
     │               │
     ▼               ▼
Execute Fix      Skip & Escalate
     │
     ▼
 Workflow Complete
```

---

# Technologies Used

| Category | Technology |
|----------|------------|
| Language | Python |
| Container | Docker |
| Kubernetes | Kind |
| AI | Anthropic Claude |
| Workflow Engine | Temporal |
| API | Kubernetes Python Client |
| Version Control | Git |
| Operating System | Ubuntu |

---

# Project Structure

```text
kubehealer/
├── activities/
│   ├── k8s_activities.py
│   └── llm_activities.py
│
├── workflows/
│   └── healer_workflow.py
│
├── chaos/
│   ├── broken-image.yaml
│   ├── oom-pod.yaml
│   └── missing-config.yaml
│
├── worker.py
├── starter.py
├── models.py
├── setup.sh
├── requirements.txt
└── README.md
```

---

# Prerequisites

- Docker
- Kind
- kubectl
- Python 3.12+
- Temporal CLI
- Anthropic API Key

---

# Installation

## Clone Repository

```bash
git clone https://github.com/<your-username>/kubehealer.git
cd kubehealer
```

---

## Create Virtual Environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

---

## Install Dependencies

```bash
pip install -r requirements.txt
```

---

# Environment Variables

Create a `.env` file.

```env
ANTHROPIC_API_KEY=your-api-key
ANTHROPIC_MODEL=claude-sonnet-5
```

The project supports configurable Anthropic models through the `ANTHROPIC_MODEL` environment variable.

---

# Verify Installation

```bash
docker --version
kind version
kubectl version --client
python --version
temporal --version
```

---

# Create Kubernetes Cluster

```bash
kind create cluster --name kubehealer-demo
```

Verify

```bash
kubectl get nodes
```

---

# Deploy Broken Applications

```bash
chmod +x setup.sh
./setup.sh
```

This deploys intentionally broken Kubernetes workloads for testing.

---

# Verify Resources

```bash
kubectl get deployments
```

Example

```text
NAME
web-app
memory-hog
config-app
```

---

# Start Temporal Server

```bash
temporal server start-dev
```

---

# Start Worker

```bash
python worker.py
```

Expected output

```text
[OK] Anthropic API key
[OK] Kubernetes cluster

KubeHealer worker started. Waiting for tasks...
```

---

# Execute Workflow

Open another terminal.

```bash
python starter.py
```

Example

```text
Starting KubeHealer workflow...

Healed 3/5 pods

config-app ... skipped

memory-hog ... patch_resources

web-app ... fix_image
```

---

# AI Diagnosis Examples

| Kubernetes Issue | AI Diagnosis | Action |
|------------------|-------------|---------|
| Invalid Image | Incorrect image tag | Patch Deployment |
| OOMKilled | Low memory limit | Increase memory |
| Missing ConfigMap | Manual issue | Skip |

---

# Automatic Remediation

KubeHealer successfully performs:

- Fix incorrect container images
- Increase memory limits
- Restart unhealthy workloads
- Patch Deployments
- Skip unsafe operations
- Record workflow history

---

# Validation

Verify the cluster.

```bash
kubectl get deployments
kubectl get pods
```

Example

```text
web-app       Running

memory-hog    Running

config-app    CreateContainerConfigError
```

The ConfigMap issue remains unresolved intentionally because it requires manual intervention.

---

# Problems Solved

## Invalid Anthropic API Key

Resolved by generating an official Anthropic API key.

---

## Deprecated Claude Model

Updated the project from

```text
claude-sonnet-4-20250514
```

to

```text
claude-sonnet-5
```

and introduced configurable model selection.

---

## Temporal Connection Failure

Started the Temporal development server before launching the worker.

---

## Incorrect Kubernetes Cluster State

Recreated the Kind cluster and initialized workloads using the project's setup script.

---

# Key Learnings

- Temporal Durable Workflows
- AI-powered Kubernetes troubleshooting
- Anthropic API integration
- Kubernetes Python Client
- Autonomous remediation
- Safe AI validation
- Deployment patching
- Environment-driven configuration
- Production debugging

---

# Best Practices

- Environment variable configuration
- Secret management
- Configurable AI model
- Safe automated remediation
- Human approval for unsafe fixes
- Infrastructure cleanup
- Version-controlled configuration

---

# Screenshots

```text
docs/screenshots/

01-kind-cluster.png

02-temporal-ui.png

03-worker-running.png

04-workflow-success.png

05-kubectl-get-deployments.png

06-kubectl-get-pods.png

07-anthropic-api-success.png
```

---

# Cleanup

Delete workloads

```bash
kubectl delete -f chaos/
```

Delete Kind cluster

```bash
kind delete cluster --name kubehealer-demo
```

---

# Future Enhancements

- Slack notifications
- Microsoft Teams integration
- Grafana integration
- Prometheus metrics
- Multi-cluster support
- GitOps integration with ArgoCD
- AI confidence scoring
- Root cause history
- Auto-generated incident reports
- Approval workflow through Slack

---

# Skills Demonstrated

- Kubernetes
- Python
- Docker
- Temporal
- Anthropic Claude API
- AI Agents
- Kubernetes Automation
- DevOps
- Site Reliability Engineering
- Cloud Native Development

---

# Conclusion

KubeHealer demonstrates how Large Language Models can be integrated into Kubernetes operations to automate incident diagnosis and remediation using Temporal Durable Workflows.

By combining AI with cloud-native technologies, the project showcases a practical implementation of autonomous operations while maintaining safety through validation and controlled execution. It highlights the potential of Agentic AI in modern DevOps and SRE workflows by reducing manual intervention for repetitive operational tasks and allowing engineers to focus on higher-value work.

---

## Author

**Preetham**

**90 Days of DevOps Challenge**

Building practical, production-ready DevOps projects with Kubernetes, Cloud, Automation, Observability, GitOps, and AI.
