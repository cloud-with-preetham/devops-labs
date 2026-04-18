# Day 87 – Introduction to Agentic AI for DevOps

## Objective

The objective of Day 87 was to begin the **Agentic AI for DevOps** journey by understanding how Large Language Models (LLMs) can move beyond answering questions and become intelligent agents capable of interacting with real-world tools. During this lab, I configured a local AI environment using Ollama, explored prompt engineering, built a Docker Error Explainer, and developed a Docker Troubleshooter Agent capable of autonomously diagnosing Docker containers using the ReAct reasoning pattern.

---

# What is an AI Agent?

An AI Agent is an application powered by a Large Language Model (LLM) that can reason, make decisions, and interact with external tools to accomplish tasks.

Unlike traditional chatbots that only generate text, AI agents can:

- Execute CLI commands
- Read files
- Call APIs
- Inspect infrastructure
- Analyze outputs
- Decide the next action based on observations

For DevOps engineers, this means AI can perform infrastructure troubleshooting in the same way an engineer would by interacting with Docker, Kubernetes, Terraform, AWS CLI, GitHub CLI, and many other tools.

---

# AI Agent vs Chatbot

| Chatbot                    | AI Agent                                             |
| -------------------------- | ---------------------------------------------------- |
| Generates text responses   | Uses tools to solve problems                         |
| Cannot execute commands    | Can execute CLI commands                             |
| Answers based on knowledge | Reasons using real infrastructure data               |
| Passive assistant          | Autonomous problem solver                            |
| No environment awareness   | Reads logs, container status, and system information |

---

# Why Agentic AI for DevOps?

Modern DevOps work revolves around command-line tools.

Instead of manually executing multiple commands, an AI agent can perform the workflow autonomously.

Example:

```
User:
Why is broken-app crashing?

↓

Agent decides to inspect containers

↓

docker ps -a

↓

Container found

↓

docker logs broken-app

↓

Reads logs

↓

docker inspect broken-app

↓

Analyzes exit code

↓

Provides root cause
```

This significantly reduces troubleshooting time and provides consistent diagnostics.

---

# The ReAct Pattern

ReAct stands for:

- **Reason**
- **Act**
- **Observe**

Instead of immediately answering a question, the AI continuously reasons about which tool to use next.

Example:

```
User:
Why is broken-app crashing?

↓

Reason

I should inspect running containers.

↓

Action

docker ps -a

↓

Observation

broken-app is exited.

↓

Reason

I should inspect the logs.

↓

Action

docker logs broken-app

↓

Observation

Application exits after 2 seconds.

↓

Reason

Inspect container configuration.

↓

Action

docker inspect broken-app

↓

Observation

ExitCode = 1

↓

Final Answer

The container intentionally exits after executing its startup command.
```

This reasoning loop continues until the model has enough information to produce an accurate response.

---

# Key Components of an Agent

## 1. LLM (Brain)

- Ollama
- Qwen 3:4B (used due to Gemma 4 compatibility issues)
- Responsible for reasoning and decision-making.

---

## 2. Tools (Hands)

Python functions wrapped with the `@tool` decorator.

Examples:

- list_containers()
- get_logs()
- inspect_container()
- list_images()

Each tool executes an actual Docker CLI command.

---

## 3. Agent Framework

LangChain's

```python
create_react_agent()
```

creates the reasoning loop that allows the AI to:

- Think
- Choose a tool
- Read the output
- Think again
- Produce an answer

---

## 4. CLI Commands

The tools internally execute commands such as:

```bash
docker ps -a
docker logs
docker inspect
docker images
```

The LLM never executes commands directly—it calls Python tools that wrap these commands.

---

# Environment Setup

The following environment was configured successfully.

## Local LLM

- Ollama
- Qwen 3:4B
- Gemma 4 downloaded (Windows compatibility issue encountered)

## Python Environment

```bash
python -m venv .venv
source .venv/Scripts/activate
```

Installed dependencies:

- langchain
- langchain-ollama
- langgraph
- ollama
- fastmcp
- langchain-mcp-adapters

---

## Pre-flight Verification

Successfully passed all setup checks.

```
[PASS] Python 3.10+
[PASS] Docker
[PASS] kubectl
[PASS] Kind
[PASS] Ollama + gemma4

5/5 — you're ready!
```

---

# Docker Error Explainer

The first AI application was a simple Docker Error Explainer.

Instead of manually searching online, Docker error messages were sent directly to the local LLM.

Example error:

```
docker: Error response from daemon:
Conflict.
The container name "/myapp" is already in use.
```

The model explained:

- What happened
- Root cause
- Commands to fix the issue

---

## Prompt Engineering

A system prompt instructed the model to behave as a Docker expert.

Example:

```python
SYSTEM_PROMPT = """
You are a Docker expert.

Explain:

1. What happened
2. Root cause
3. How to fix it

Keep responses short.
"""
```

This demonstrated how prompt engineering significantly affects response quality.

---

## Temperature

The model used:

```python
temperature = 0.3
```

A lower temperature produces:

- Deterministic responses
- Consistent troubleshooting
- Reliable technical explanations

---

# Docker Troubleshooter Agent

The next application introduced AI agents.

Instead of simply generating text, the model was capable of using tools autonomously.

Available tools included:

- list_containers()
- get_logs()
- inspect_container()

The agent successfully diagnosed the intentionally failing Docker container named:

```
broken-app
```

The diagnosis concluded that:

- The startup command intentionally exited after two seconds.
- Exit code was 1.
- The container was behaving exactly as configured.

---

## Additional Tool

A new custom tool was added.

```python
@tool
def list_images():
```

This allowed the AI agent to answer questions such as:

```
What Docker images are available on this machine?
```

without any additional prompting.

---

# Agent Architecture

```
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
       Tool Output (Text)
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

While using Windows with Ollama, the Gemma 4 model repeatedly crashed with the following error:

```
GGML_ASSERT(...)
```

This issue occurred during tool-calling operations.

To continue the lab successfully, the model was switched to:

```
Qwen 3:4B
```

which supported LangChain tool calling correctly and allowed the agent to function as expected.

This troubleshooting experience highlighted the importance of validating model compatibility with the chosen runtime environment.

---

# Screenshots

- Project setup
- Python virtual environment
- Dependency installation
- Setup verification
- Docker Error Explainer
- System prompt
- Broken Docker container
- Docker Troubleshooter Agent diagnosis
- Agent tool usage
- Agent architecture
- Custom tool implementation
- Custom tool output

---

# Key Learnings

- Understood the difference between AI agents and traditional chatbots.
- Learned the ReAct (Reason, Act, Observe) reasoning pattern.
- Configured a local LLM environment using Ollama.
- Built a Docker Error Explainer using prompt engineering.
- Learned how system prompts influence model behavior.
- Built a Docker Troubleshooter Agent using LangChain.
- Integrated Docker CLI commands as AI tools.
- Extended the agent with a custom tool.
- Understood how AI agents autonomously reason over tool outputs.
- Experienced and resolved a real-world model compatibility issue by switching from Gemma 4 to Qwen 3:4B.

---

# Conclusion

Day 87 marked the beginning of Agentic AI for DevOps. Instead of simply interacting with a language model, I built an autonomous AI agent capable of using Docker tools, reasoning over real infrastructure data, and diagnosing container issues. This architecture can be extended to Kubernetes, Terraform, AWS CLI, and other DevOps tools, laying the foundation for intelligent infrastructure automation.
