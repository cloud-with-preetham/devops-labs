# Day 88 – Multi-Tool Agents, MCP & CI/CD Failure Analyzer

**Duration:** Day 88 of #90DaysOfDevOps
**Focus Areas:** AI Agents, LangChain, Model Context Protocol (MCP), Kubernetes, Docker, GitHub Actions, AWS CLI

---

# Overview

On Day 88, I expanded my AI-powered DevOps assistant from a Docker-only troubleshooting agent into a **multi-tool DevOps agent** capable of diagnosing Docker containers, Kubernetes workloads, GitHub Actions pipelines, and AWS resources.

I also explored the **Model Context Protocol (MCP)**, an open standard that enables AI applications to discover and use external tools dynamically instead of hardcoding them into agent implementations.

Finally, I built a custom AWS CLI tool that allows the AI agent to inspect Amazon EC2 instances.

---

# Learning Objectives

- Build a multi-tool DevOps AI agent
- Understand the ReAct agent pattern
- Learn the Model Context Protocol (MCP)
- Expose Kubernetes tools through an MCP server
- Build a GitHub Actions failure analyzer
- Create a custom AWS CLI tool
- Understand how AI agents dynamically choose tools

---

# Technologies Used

- Python
- LangChain
- LangGraph
- Ollama
- Qwen3:4b
- FastMCP
- LangChain MCP Adapters
- Docker
- Kubernetes
- Kind
- GitHub CLI (gh)
- AWS CLI
- Ubuntu 26.04 WSL

---

# Project Architecture

```text
                    User Question
                          │
                          ▼
                  LangChain ReAct Agent
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
     Docker Tools    Kubernetes Tools   GitHub Tools
          │               │                │
          ▼               ▼                ▼
      Docker CLI      kubectl CLI        gh CLI
                          │
                          ▼
                    AWS CLI Tool
                          │
                          ▼
                  Amazon EC2 Resources
```

The agent automatically selects the appropriate tool based on the user's question without requiring manual intervention.

---

# Task 1 – Build the Multi-Tool DevOps Agent

## Objective

Extend the Docker troubleshooting agent from Day 87 by adding Kubernetes tools.

### Docker Tools

- List Docker containers
- Retrieve container logs
- Inspect Docker containers

### Kubernetes Tools

- List Pods
- Describe Pods
- View Kubernetes Events

---

## Test Environment

### Created a Kind Cluster

```bash
kind create cluster --name devops-demo
```

### Deployed a Broken Kubernetes Pod

The pod intentionally exited after a few seconds to simulate an application failure.

```text
broken-pod
Status: Error
```

### Created a Broken Docker Container

```bash
docker run -d --name broken-container nginx:alpine \
sh -c "echo 'container starting...' && sleep 2 && exit 1"
```

Status:

```text
Exited (1)
```

---

## ReAct Agent Workflow

The agent decides which tools to invoke based on the user's request.

Example questions:

```
What's broken across Docker and Kubernetes?

Why is broken-pod crashing?

Are there any unhealthy Docker containers?

Describe the recent events in the default namespace.
```

Instead of hardcoding logic, the LLM reasons about the question and invokes the most appropriate tool.

---

## Screenshots

### day88-01-broken-environment.png

Shows:

- Broken Kubernetes Pod
- Broken Docker Container

### day88-02-multitool-agent-diagnosis.png

Shows:

- Multi-tool DevOps Agent
- AI diagnosis
- Tool selection
- Docker and Kubernetes troubleshooting

---

# Task 2 – Understanding MCP

## What is MCP?

The **Model Context Protocol (MCP)** is an open standard that enables AI applications to communicate with external tools and services.

Instead of embedding tools directly inside an AI application, they are exposed through an MCP Server.

Multiple AI clients can then discover and use those tools dynamically.

---

## Without MCP

```text
AI Agent
   │
   ├── Docker Tool
   ├── Kubernetes Tool
   └── AWS Tool
```

Each framework must implement and maintain its own tools.

---

## With MCP

```text
               MCP Server
        ┌─────────────────────┐
        │ list_pods()         │
        │ describe_pod()      │
        │ get_events()        │
        └─────────┬───────────┘
                  │
      ┌───────────┼────────────┐
      │           │            │
 Claude      VS Code      Python Agent
 Desktop     Copilot      (LangChain)
```

---

## Why MCP Matters

Benefits include:

- Write tools once
- Reuse across multiple AI clients
- Dynamic tool discovery
- Reduced code duplication
- Framework-independent integrations

---

# Task 3 – MCP Server

Created an MCP Server using FastMCP.

Registered Kubernetes tools using:

```python
@mcp.tool
```

Available tools:

- list_pods()
- describe_pod()
- get_events()

Started the server using:

```bash
python mcp_server.py
```

FastMCP successfully started and exposed Kubernetes tools via the stdio transport.

---

## MCP Client

Attempted to connect using:

```
agent_with_mcp.py
```

The MCP server started successfully.

However, the MCP client encountered a **Connection closed** error during initialization using the latest MCP SDK/runtime.

The server implementation was successful, while the client requires updates for compatibility with the current MCP libraries.

---

## Screenshot

### day88-03-mcp-server-running.png

Shows:

- FastMCP Server
- Kubernetes Tools Server
- stdio Transport

---

# Task 4 – CI/CD Failure Analyzer

Built an AI-powered GitHub Actions troubleshooting agent.

The agent integrates with GitHub CLI (`gh`) and can:

- List recent workflow runs
- Retrieve failed logs
- Read GitHub Actions workflow files
- Explain CI/CD failures

---

## Example Questions

```
Show me the recent workflow runs

What failed in my last CI run?

Read the gitops-ci.yml workflow file and explain what it does
```

The analyzer successfully:

- Listed recent GitHub Actions workflow runs
- Identified the latest failed workflow
- Explained GitHub log retention (HTTP 410)
- Read and analyzed the GitOps workflow
- Explained every stage of the CI/CD pipeline

---

## Screenshot

### day88-04-cicd-failure-analyzer.png

Shows:

- Workflow analysis
- AI explanation
- GitHub Actions troubleshooting

---

# Task 5 – Custom AWS Tool

Created a custom LangChain tool that integrates with AWS CLI.

```python
@tool
def list_ec2_instances():
```

The tool retrieves:

- Instance ID
- Instance State
- Instance Type
- Name Tag

using:

```bash
aws ec2 describe-instances
```

---

## Example Questions

```
List all EC2 instances

Which EC2 instances are running?
```

The AI agent automatically invoked the AWS CLI tool and responded with:

- EC2 Instance ID
- State (Stopped)
- Instance Type (m7i-flex.large)
- Name Tag (my-server)

---

## Screenshot

### day88-05-custom-aws-tool.png

Shows:

- EC2 instance listing
- AWS CLI integration
- AI-generated explanation

---

# Tool Pattern Learned

Every AI tool follows the same workflow.

```text
User Question
      │
      ▼
LLM reasons
      │
      ▼
Chooses the appropriate Tool
      │
      ▼
Executes CLI Command
      │
      ▼
Collects Output
      │
      ▼
Explains the Result
```

This reusable pattern works with virtually any CLI utility, including:

- Docker
- Kubernetes
- AWS CLI
- Terraform
- Ansible
- GitHub CLI
- Helm
- Azure CLI

---

# Screenshots Structure

```text
screenshots/
├── day88-01-broken-environment.png
├── day88-02-multitool-agent-diagnosis.png
├── day88-03-mcp-server-running.png
├── day88-04-cicd-failure-analyzer.png
└── day88-05-custom-aws-tool.png
```

---

# Key Takeaways

- Built a multi-tool AI agent capable of troubleshooting Docker and Kubernetes.
- Learned how ReAct agents dynamically choose tools.
- Explored the Model Context Protocol (MCP) and its role in AI tool interoperability.
- Built and ran an MCP Server using FastMCP.
- Built an AI-powered GitHub Actions failure analyzer using GitHub CLI.
- Created a custom AWS CLI tool for inspecting EC2 instances.
- Established a reusable design pattern for integrating AI with DevOps command-line tools.

---

# Skills Gained

- AI Agents for DevOps
- LangChain
- LangGraph
- FastMCP
- Model Context Protocol (MCP)
- Docker Troubleshooting
- Kubernetes Troubleshooting
- GitHub Actions Analysis
- AWS CLI Automation
- ReAct Agent Pattern
- Tool Calling
- DevOps Automation
- Ubuntu WSL Development Environment
