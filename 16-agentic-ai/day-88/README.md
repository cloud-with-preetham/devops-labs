# Day 88 – Multi-Tool Agents, MCP & CI/CD Failure Analyzer

> **90 Days of DevOps – Day 88**

Today I built a **Multi-Tool DevOps AI Agent** capable of troubleshooting Docker containers, Kubernetes workloads, GitHub Actions pipelines, and AWS EC2 resources using LangChain, Ollama, FastMCP, GitHub CLI, and AWS CLI.

---

## Project Overview

Modern DevOps engineers work with multiple platforms and tools every day. Instead of manually running different CLI commands, I built an AI-powered DevOps assistant that intelligently selects the appropriate tool based on the user's request.

Today's project demonstrates how Large Language Models can integrate with real DevOps tooling using the ReAct pattern and the Model Context Protocol (MCP).

---

## Objectives

- Build a Multi-Tool DevOps Agent
- Troubleshoot Docker and Kubernetes using AI
- Learn the Model Context Protocol (MCP)
- Build an MCP Server
- Build a GitHub Actions Failure Analyzer
- Create a custom AWS CLI Tool
- Understand AI Tool Calling patterns

---

# Tech Stack

| Category | Technologies |
|----------|--------------|
| Programming | Python |
| AI Framework | LangChain, LangGraph |
| LLM | Ollama (Qwen3:4b) |
| AI Protocol | FastMCP, LangChain MCP Adapters |
| Containers | Docker |
| Container Orchestration | Kubernetes, Kind |
| Cloud | AWS CLI |
| CI/CD | GitHub Actions, GitHub CLI |
| Operating System | Ubuntu 26.04 WSL |

---

# Project Architecture

```text
                     User Question
                           │
                           ▼
                 LangChain ReAct Agent
                           │
       ┌───────────────────┼────────────────────┐
       │                   │                    │
       ▼                   ▼                    ▼
 Docker Tools       Kubernetes Tools      GitHub Tools
       │                   │                    │
       ▼                   ▼                    ▼
 Docker CLI         kubectl Commands      GitHub CLI
                           │
                           ▼
                    AWS CLI Tool
                           │
                           ▼
                     Amazon EC2
```

The AI agent reasons about the user's question and automatically invokes the appropriate tool.

---

# Project Structure

```text
day-88/
│
├── README.md
├── day-88-multi-tool-mcp.md
│
└── screenshots/
    ├── day88-01-broken-environment.png
    ├── day88-02-multitool-agent-diagnosis.png
    ├── day88-03-mcp-server-running.png
    ├── day88-04-cicd-failure-analyzer.png
    └── day88-05-custom-aws-tool.png
```

---

# Tasks Completed

## Task 1 – Multi-Tool DevOps Agent

Built an AI agent capable of troubleshooting both Docker containers and Kubernetes resources.

### Docker Tools

- List Containers
- Get Container Logs
- Inspect Containers

### Kubernetes Tools

- List Pods
- Describe Pods
- View Kubernetes Events

### Environment Setup

Created a Kind Kubernetes cluster.

```bash
kind create cluster --name devops-demo
```

Created a broken Kubernetes Pod.

Created a broken Docker container.

The AI agent successfully diagnosed issues across both platforms.

---

## Task 2 – Understanding MCP

Learned the Model Context Protocol (MCP).

### What is MCP?

MCP is an open standard that enables AI applications to communicate with external tools.

Instead of embedding tools inside an AI application, they are exposed through an MCP Server and dynamically discovered by compatible clients.

### Benefits

- Write tools once
- Reuse across multiple AI clients
- Dynamic tool discovery
- Framework-independent integrations
- Better scalability

---

## Task 3 – MCP Server

Built an MCP Server using FastMCP.

Exposed Kubernetes tools:

- list_pods()
- describe_pod()
- get_events()

Started the server using:

```bash
python mcp_server.py
```

FastMCP successfully started and exposed Kubernetes tools using stdio transport.

### MCP Client

Attempted to connect using the provided MCP client implementation.

The MCP server started successfully.

The MCP client encountered a **Connection closed** initialization error with the current MCP SDK versions, highlighting a compatibility issue rather than a server implementation problem.

---

## Task 4 – CI/CD Failure Analyzer

Built an AI-powered GitHub Actions troubleshooting assistant.

Capabilities:

- List recent workflow runs
- Retrieve failed workflow logs
- Read workflow YAML files
- Explain CI/CD pipelines
- Diagnose workflow failures

Example prompts:

```
Show me the recent workflow runs

What failed in my last CI run?

Read the gitops-ci.yml workflow file and explain what it does
```

The agent successfully analyzed the GitOps workflow and explained each stage of the pipeline.

---

## Task 5 – Custom AWS Tool

Built a custom LangChain tool using AWS CLI.

```python
@tool
def list_ec2_instances():
```

The tool retrieves:

- Instance ID
- Instance State
- Instance Type
- Name Tag

The AI agent successfully answered:

```
List all EC2 instances

Which EC2 instances are running?
```

using live AWS CLI data.

---

# AI Tool Calling Workflow

Every tool follows the same workflow.

```text
User Question
      │
      ▼
LLM analyzes intent
      │
      ▼
Chooses Tool
      │
      ▼
Runs CLI Command
      │
      ▼
Reads Output
      │
      ▼
Generates Explanation
```

This design pattern can be reused with:

- Docker
- Kubernetes
- AWS CLI
- Terraform
- Ansible
- Helm
- GitHub CLI
- Azure CLI

---

# Screenshots

| Screenshot | Description |
|------------|-------------|
| day88-01-broken-environment.png | Broken Docker container and Kubernetes Pod |
| day88-02-multitool-agent-diagnosis.png | AI agent diagnosing Docker and Kubernetes issues |
| day88-03-mcp-server-running.png | FastMCP server exposing Kubernetes tools |
| day88-04-cicd-failure-analyzer.png | GitHub Actions workflow analysis |
| day88-05-custom-aws-tool.png | AI agent inspecting AWS EC2 instances |

---

# Key Learnings

- Built a multi-tool DevOps AI assistant.
- Learned how ReAct agents dynamically invoke tools.
- Explored the Model Context Protocol (MCP).
- Built and ran an MCP Server.
- Built an AI-powered GitHub Actions failure analyzer.
- Integrated AWS CLI with LangChain.
- Created a reusable AI tool pattern for DevOps automation.

---

# Skills Gained

- LangChain
- LangGraph
- Ollama
- FastMCP
- Model Context Protocol (MCP)
- AI Tool Calling
- Docker Troubleshooting
- Kubernetes Troubleshooting
- GitHub Actions Analysis
- AWS CLI Automation
- ReAct Pattern
- DevOps Automation
- Ubuntu WSL

---

# Conclusion

Day 88 demonstrated how AI agents can become powerful DevOps assistants by combining natural language reasoning with command-line tools. Instead of memorizing commands, the agent intelligently selected and executed Docker, Kubernetes, GitHub, and AWS operations based on user intent.

This project highlighted how the same AI tool-calling pattern can be extended to virtually any DevOps technology, making it a strong foundation for building production-ready AI-powered automation platforms.
