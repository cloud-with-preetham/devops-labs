# Day 87 - Introduction to Agentic AI for DevOps

## Objective

The goal of Day 87 was to understand the fundamentals of **Agentic AI** by building a simple AI-powered DevOps assistant. Unlike traditional LLM applications that only generate text, an AI agent can reason about a problem, choose the appropriate tool, execute commands, observe the results, and provide an informed response.

By the end of this lab, I built a Docker Troubleshooter Agent capable of interacting with the Docker CLI using LangChain tools and Ollama.

---

# Learning Objectives

- Understand the difference between an LLM application and an AI Agent.
- Set up a local Agentic AI development environment.
- Use Ollama as a local LLM provider.
- Build a Docker Error Explainer using Python and Ollama.
- Create LangChain tools for Docker operations.
- Build a ReAct Agent capable of invoking tools.
- Understand how reasoning and tool execution work together.
- Extend the agent with additional capabilities.

---

# Environment

| Component | Version |
|-----------|---------|
| OS | Windows 11 |
| Python | 3.13.15 |
| Ollama | 0.32.6 |
| Model | qwen3 |
| LangChain | Latest |
| LangGraph | Latest |
| Docker | 29.6.2 |

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
├── README.md
├── requirements.txt
└── screenshots/
```

---

# Task 1 – Understanding Agentic AI

Studied the core concepts of Agentic AI and how it differs from traditional LLM applications.

### LLM Application

```text
User
   │
   ▼
Prompt
   │
   ▼
LLM
   │
   ▼
Response
```

The model only generates text.

---

### AI Agent

```text
User
   │
   ▼
Reason
   │
   ▼
Choose Tool
   │
   ▼
Execute Tool
   │
   ▼
Observe Output
   │
   ▼
Reason Again
   │
   ▼
Final Answer
```

The model can interact with external systems and make decisions based on tool outputs.

---

# Task 2 – Environment Setup

Installed and configured:

- Python
- pip
- Virtual Environment
- Ollama
- Docker CLI
- LangChain
- LangGraph

Verified installation using:

```bash
python --version

pip --version

ollama --version

docker --version
```

---

# Task 3 – Docker Error Explainer

Built a simple AI application using Ollama.

### Features

- Accept Docker-related questions.
- Send prompts to Ollama.
- Receive AI-generated troubleshooting guidance.
- Use a custom system prompt.
- Configure model temperature.

### Example

Input

```text
docker: Error response from daemon:
Conflict.
The container name "/myapp" is already in use.
```

Output

- Root Cause
- Troubleshooting Steps
- Docker Commands
- Best Practices

---

# Prompt Engineering

Implemented a custom system prompt.

```python
SYSTEM_PROMPT = """
You are a Senior DevOps Engineer.

Explain Docker errors.

Provide:

1. Root Cause
2. Troubleshooting
3. Docker Commands
4. Best Practices

Keep responses concise.
"""
```

---

# Task 4 – Docker Troubleshooter Agent

Converted the application into a real AI Agent.

Instead of answering directly, the agent now chooses Docker tools automatically.

Implemented LangChain tools.

### Tool 1

```python
list_containers()
```

Runs

```bash
docker ps -a
```

---

### Tool 2

```python
get_logs(container_name)
```

Runs

```bash
docker logs
```

---

### Tool 3

```python
inspect_container(container_name)
```

Runs

```bash
docker inspect
```

---

# Agent Workflow

```text
User

↓

ChatOllama

↓

Reason

↓

Select Tool

↓

Execute Docker Command

↓

Read Output

↓

Generate Final Response
```

---

# Task 5 – Understanding ReAct Architecture

Learned the complete ReAct (Reason + Act) workflow.

```text
Question

↓

Reason

↓

Tool Selection

↓

Execute Tool

↓

Observe Result

↓

Reason Again

↓

Final Response
```

This architecture enables AI agents to interact with real-world systems instead of relying solely on model knowledge.

---

# Task 6 – Extending the Agent

Added a new capability.

### Docker Images Tool

```python
list_images()
```

Runs

```bash
docker images
```

Although the local model did not consistently invoke this tool automatically, the implementation and registration were completed successfully, demonstrating how an agent can be extended with additional capabilities.

---

# Challenges Faced

## Python PATH Issues

Problem

- Python was installed but not detected in Git Bash.

Solution

- Updated the system PATH.
- Restarted the terminal.

---

## Ollama Model Issues

Problem

Gemma 4 crashed during inference with internal runtime errors.

Solution

- Switched to the Qwen 3 model for better compatibility with tool calling.

---

## Docker CLI Issues

Problem

Docker CLI was initially unavailable in the terminal.

Solution

- Started Docker Desktop.
- Verified Docker installation and PATH.

---

## LangChain Tool Calling

Problem

Some local models did not fully support tool invocation.

Solution

- Switched to a compatible Ollama model and verified successful ReAct agent execution.

---

# Key Learnings

- AI Agents are more powerful than traditional LLM applications.
- LangChain tools allow LLMs to interact with external systems.
- Tool descriptions (docstrings) play a critical role in tool selection.
- ReAct enables reasoning before and after tool execution.
- Local LLMs can be used for DevOps automation without relying on cloud APIs.

---

# Screenshots

```
screenshots/
│
├── 01-ollama-setup.png
├── 02-docker-error-explainer.png
├── 03-react-agent.png
└── 04-list-images-tool.png
```

---

# Outcome

Successfully built a local Agentic AI prototype capable of:

- Explaining Docker errors.
- Listing Docker containers.
- Reading Docker logs.
- Inspecting Docker containers.
- Executing Docker commands through LangChain tools.
- Reasoning using the ReAct pattern.
- Extending the agent with additional Docker capabilities.

This project establishes the foundation for building more advanced AI-powered DevOps agents capable of working with Kubernetes, Terraform, AWS, and other cloud-native tools.

---

# Skills Practiced

- Agentic AI
- Prompt Engineering
- Ollama
- LangChain
- LangGraph
- ReAct Pattern
- Docker CLI
- Python
- AI Tool Calling
- DevOps Automation