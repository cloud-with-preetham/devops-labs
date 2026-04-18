# Day 87 – Introduction to Agentic AI for DevOps

> Building my first AI-powered DevOps Agent using **Python**, **Ollama**, **LangChain**, and **Docker**.

---

## Overview

On Day 87 of my **90 Days of DevOps** journey, I explored the fundamentals of **Agentic AI** by building a local AI-powered DevOps assistant.

Unlike a traditional LLM application that only generates text, this project demonstrates how an AI Agent can:

- Understand user requests
- Select the appropriate tool
- Execute Docker commands
- Analyze command output
- Return an intelligent response

This project serves as the foundation for building autonomous DevOps assistants capable of interacting with real infrastructure.

---

## Learning Objectives

- Understand the difference between an LLM application and an AI Agent
- Build a local AI application using Ollama
- Create LangChain tools for Docker
- Implement the ReAct (Reason → Act → Observe) pattern
- Integrate Docker CLI with AI
- Build an extensible AI agent architecture

---

# Tech Stack

| Category          | Technology  |
| ----------------- | ----------- |
| Language          | Python 3.13 |
| LLM Runtime       | Ollama      |
| Model             | Qwen 3      |
| Framework         | LangChain   |
| Agent Framework   | LangGraph   |
| Container Runtime | Docker      |
| OS                | Windows 11  |

---

# Project Structure

```text
agentic-ai-for-devops/
│
├── module-1/
│   └── explainer.py
│
├── module-2/
│   └── agent.py
│
├── screenshots/
│
├── requirements.txt
└── README.md
```

---

# Module 1 – Docker Error Explainer

A simple AI application that accepts Docker-related errors and generates beginner-friendly troubleshooting guidance.

### Features

- Accept Docker error messages
- Send prompts to Ollama
- Generate AI explanations
- Explain root cause
- Suggest troubleshooting steps
- Recommend Docker commands

---

## Example

### Input

```text
docker: Error response from daemon:
Conflict.
The container name "/myapp" is already in use.
```

### Output

- Root Cause
- Troubleshooting Steps
- Docker Commands
- Best Practices

---

# Module 2 – Docker Troubleshooter Agent

This module converts the application into an AI Agent capable of using external tools.

Instead of answering from memory, the LLM selects and executes Docker commands.

Implemented tools include:

- List Docker containers
- View container logs
- Inspect Docker containers
- List Docker images

---

# Agent Workflow

```text
                User Question
                      │
                      ▼
              ChatOllama (Qwen3)
                (Reasoning)
                      │
                      ▼
           LangChain ReAct Agent
                      │
      ┌───────────────┼───────────────┐
      ▼               ▼               ▼
List Containers    Get Logs     Inspect Container
      │               │               │
      ▼               ▼               ▼
 docker ps -a   docker logs   docker inspect
      │               │               │
      └───────────────┼───────────────┘
                      ▼
              Tool Output
                      │
                      ▼
             AI Final Response
```

---

# Docker Tools

## List Containers

Runs:

```bash
docker ps -a
```

---

## View Logs

Runs:

```bash
docker logs <container-name>
```

---

## Inspect Container

Runs:

```bash
docker inspect <container-name>
```

---

## List Images

Runs:

```bash
docker images
```

---

# ReAct Pattern

The AI Agent follows the ReAct workflow.

```text
Reason

↓

Act

↓

Observe

↓

Reason Again

↓

Answer
```

This enables the model to make informed decisions instead of relying solely on pre-trained knowledge.

---

# Prompt Engineering

A custom system prompt guides the model's behavior.

```python
SYSTEM_PROMPT = """
You are a Senior DevOps Engineer.

When given a Docker issue:

1. Explain the issue.
2. Identify the root cause.
3. Suggest troubleshooting steps.
4. Recommend Docker commands.

Keep the response concise.
"""
```

---

# Challenges Faced

## Python PATH

**Issue**

Python was installed but not detected in Git Bash.

**Resolution**

Updated the system PATH and restarted the terminal.

---

## Ollama Model Compatibility

**Issue**

Gemma 4 crashed due to runtime errors during inference.

**Resolution**

Switched to the Qwen 3 model for stable execution and tool-calling support.

---

## Docker CLI

**Issue**

Docker commands were initially unavailable.

**Resolution**

Started Docker Desktop and verified the Docker CLI installation.

---

## LangChain Tool Calling

**Issue**

Some Ollama models did not reliably support tool invocation.

**Resolution**

Used a compatible model and verified successful tool execution through the ReAct agent.

---

# Key Learnings

- AI Agents extend traditional LLM applications with external tools.
- LangChain enables seamless tool integration.
- Ollama allows running local LLMs without cloud APIs.
- ReAct combines reasoning with tool execution.
- Docker CLI can be integrated into AI workflows using Python.

---

# Screenshots

| Screenshot                    | Description                |
| ----------------------------- | -------------------------- |
| 01-ollama-setup.png           | Environment setup          |
| 02-docker-error-explainer.png | Docker Error Explainer     |
| 03-react-agent.png            | ReAct agent in action      |
| 04-agent-extension.png        | Extended Docker image tool |
| 05-project-structure.png      | Project structure          |
| 06-source-code.png            | Source code overview       |

---

# Skills Practiced

- Agentic AI
- LangChain
- LangGraph
- Ollama
- Prompt Engineering
- Python
- Docker
- Docker CLI Automation
- ReAct Pattern
- AI Tool Calling
- DevOps Automation

---

# Outcome

Successfully built a local AI-powered DevOps agent capable of interacting with Docker through LangChain tools and Ollama.

This project lays the foundation for future AI-powered DevOps assistants that can work with:

- Kubernetes
- Terraform
- AWS CLI
- Azure CLI
- Linux
- CI/CD Pipelines
- Monitoring & Observability

---

## Repository Highlights

- Local LLM execution using Ollama
- AI-powered Docker troubleshooting
- LangChain tool integration
- ReAct-based agent architecture
- Extensible design for future DevOps tools

---

## What's Next?

In the next phase, this agent will be enhanced to support:

- Kubernetes troubleshooting
- Linux command execution
- AWS resource inspection
- Terraform analysis
- Multi-tool autonomous DevOps workflows

---

### If you found this project helpful, feel free to star the repository and follow my **90 Days of DevOps** journey.
