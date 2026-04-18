# Day 87 – Introduction to Agentic AI for DevOps

> Building AI-powered DevOps agents using **Ollama**, **LangChain**, **Docker**, and the **ReAct** reasoning pattern.

---

## Overview

Day 87 marks the beginning of the **Agentic AI for DevOps** journey. Instead of using Large Language Models (LLMs) as traditional chatbots, the focus shifts toward building intelligent AI agents capable of interacting with real-world DevOps tools.

During this lab, I configured a local AI development environment using Ollama, built a Docker Error Explainer, and developed a Docker Troubleshooter Agent capable of autonomously diagnosing Docker containers using the ReAct (Reason + Act) pattern.

---

## Objectives

- Understand the fundamentals of Agentic AI.
- Learn the difference between AI agents and chatbots.
- Configure a local LLM environment using Ollama.
- Build a Docker Error Explainer using prompt engineering.
- Develop a Docker Troubleshooter Agent using LangChain.
- Understand the ReAct reasoning pattern.
- Extend the AI agent by adding a custom Docker tool.

---

# Tech Stack

| Category           | Technologies                    |
| ------------------ | ------------------------------- |
| AI Runtime         | Ollama                          |
| LLM                | Qwen 3:4B _(Gemma 4 attempted)_ |
| AI Framework       | LangChain                       |
| Agent Framework    | LangGraph                       |
| MCP                | FastMCP                         |
| Language           | Python                          |
| Container Platform | Docker                          |
| Environment        | Python Virtual Environment      |

---

# Project Structure

```text
day-87/
├── README.md
├── day-87-agentic-ai-intro.md
└── screenshots/
    ├── 01-project-setup.png
    ├── 02-python-venv.png
    ├── 03-dependencies-installed.png
    ├── 04-verify-setup.png
    ├── 05-docker-error-explainer.png
    ├── 06-system-prompt.png
    ├── 07-broken-container.png
    ├── 08-agent-diagnosis.png
    ├── 09-agent-tool-usage.png
    ├── 10-agent-architecture.png
    ├── 11-custom-tool-code.png
    └── 12-custom-tool-output.png
```

---

# Lab Tasks

## Task 1 – Understanding Agentic AI

Studied the core concepts of Agentic AI and how AI agents differ from traditional chatbots.

Covered topics:

- What is an AI Agent?
- AI Agent vs Chatbot
- ReAct Pattern
- Why Agentic AI is valuable for DevOps
- Key components of an AI agent

---

## Task 2 – Environment Setup

Configured the complete local development environment.

### Completed

- Installed Ollama
- Downloaded local LLMs
- Created Python virtual environment
- Installed project dependencies
- Verified Docker
- Verified kubectl
- Verified Kind
- Completed environment validation

Verification Output

```text
[PASS] Python 3.10+
[PASS] Docker
[PASS] kubectl
[PASS] Kind
[PASS] Ollama + gemma4

5/5 — you're ready!
```

---

## Task 3 – Docker Error Explainer

Built a lightweight AI application capable of translating Docker errors into beginner-friendly explanations.

### Features

- Accepts Docker error messages
- Explains the issue
- Identifies the root cause
- Suggests commands to resolve the problem

Example input

```text
docker: Error response from daemon:
Conflict.
The container name "/myapp" is already in use.
```

The AI explained:

- What happened
- Why it happened
- How to fix it

---

## Task 4 – Docker Troubleshooter Agent

Built an autonomous Docker troubleshooting agent capable of using Docker CLI tools.

Available tools:

- `list_containers()`
- `get_logs()`
- `inspect_container()`

The AI successfully diagnosed the intentionally failing `broken-app` container by inspecting its state, logs, and configuration.

---

## Task 5 – Agent Architecture

Learned how AI agents work internally using the ReAct reasoning loop.

Architecture:

```text
User
  │
  ▼
ChatOllama
  │
  ▼
create_react_agent()
  │
  ├── docker ps
  ├── docker logs
  └── docker inspect
          │
          ▼
 Tool Output
          │
          ▼
Reason Again
          │
          ▼
 Final Answer
```

---

## Task 6 – Extending the Agent

Added a custom tool to the AI agent.

```python
@tool
def list_images():
```

This allowed the agent to answer:

> What Docker images are available on this machine?

without any additional programming.

---

# ReAct Pattern

The Docker Troubleshooter Agent follows the ReAct (Reason + Act) pattern.

```text
User Question

↓

Reason

↓

Choose Tool

↓

Execute Tool

↓

Observe Output

↓

Reason Again

↓

Final Answer
```

This enables the AI agent to autonomously determine which tool to invoke based on the user's request.

---

# AI Agent Workflow

```text
                    User
                      │
                      ▼
      Why is broken-app crashing?
                      │
                      ▼
          ChatOllama (Qwen 3)
                      │
                      ▼
        create_react_agent()
                      │
      ┌────────┬──────────┬──────────┐
      ▼        ▼          ▼
docker ps  docker logs docker inspect
      │        │          │
      └────────┴──────────┘
               │
               ▼
         Tool Output
               │
               ▼
      LLM Reasons Again
               │
               ▼
        Final Diagnosis
```

---

# Challenges Faced

## Gemma 4 Compatibility Issue

While running the project on Windows, the `gemma4` model repeatedly crashed due to an Ollama runtime issue.

```
GGML_ASSERT(...)
```

To continue the lab successfully, the project was switched to **Qwen 3:4B**, which supported LangChain tool calling and allowed the Docker Troubleshooter Agent to function correctly.

This served as a valuable real-world troubleshooting exercise involving AI runtime compatibility.

---

# Key Learnings

- Understood the fundamentals of Agentic AI.
- Learned the difference between AI agents and chatbots.
- Configured a local LLM environment using Ollama.
- Built a Docker Error Explainer using prompt engineering.
- Learned how system prompts affect model behavior.
- Built an autonomous Docker Troubleshooter Agent.
- Learned the ReAct reasoning pattern.
- Wrapped Docker CLI commands as AI tools using LangChain.
- Added a custom Docker tool to extend the agent.
- Diagnosed and resolved a model compatibility issue by switching from Gemma 4 to Qwen 3:4B.

---

# Screenshots

| Screenshot | Description                     |
| ---------- | ------------------------------- |
| 01         | Project setup                   |
| 02         | Python virtual environment      |
| 03         | Dependency installation         |
| 04         | Environment verification        |
| 05         | Docker Error Explainer          |
| 06         | System prompt configuration     |
| 07         | Broken Docker container         |
| 08         | Docker Troubleshooter diagnosis |
| 09         | Agent tool usage                |
| 10         | Agent architecture              |
| 11         | Custom tool implementation      |
| 12         | Custom tool output              |

---

# Skills Gained

- Agentic AI Fundamentals
- Prompt Engineering
- Ollama
- LangChain
- LangGraph
- Docker Automation
- AI Tool Calling
- ReAct Agents
- Python Tool Development
- AI-assisted DevOps Troubleshooting

---

# Conclusion

Day 87 introduced the foundations of **Agentic AI for DevOps** by moving beyond traditional chatbot interactions and building AI agents capable of reasoning over real infrastructure. By combining Ollama, LangChain, Docker CLI tools, and the ReAct pattern, I created an autonomous troubleshooting agent that can inspect containers, analyze logs, and provide intelligent diagnostics. These concepts establish the foundation for extending AI agents to Kubernetes, Terraform, AWS, and other DevOps platforms in the upcoming days.

---

## Repository

**Module:** Agentic AI for DevOps

**Day:** 87 / 90

**Focus:** Introduction to Agentic AI, ReAct Pattern, Docker AI Agent

**Status:** Completed
