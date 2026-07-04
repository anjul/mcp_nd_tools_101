---
title: MCP Tools 101
layout: default
permalink: /mcp-tools-101/
---

# 🚀 Ultimate Guide to Model Context Protocol (MCP) for SE & ML Engineers

## 📌 1. What is MCP? (The Core Concept)
**Model Context Protocol (MCP)** is an open-standard protocol developed by Anthropic. Think of it as a **Universal API Gateway** or a **Standardized Driver** for Large Language Models (LLMs).

### 💡 The Problem it Solves:
Previously, if you wanted an AI model to connect to a Database, GitHub, or Slack, you had to write custom, fragmented integration code for each tool. 
MCP standardizes this. The AI model acts as a universal plug, and every data source or API exposes itself via a standardized **MCP Server**.

### 🏠 Real-World Analogy: Smart Home Hub
* **LLM (The Brain):** The smart hub (like Google Home/Alexa) that understands your intent.
* **MCP (The Wire/Protocol):** The common language/protocol that allows the hub to talk to appliances.
* **MCP Tools (The Appliances):** The AC, Smart TV, or Smart Lights. Each has a "driver" telling the hub what it can do.

---

## 🔍 2. How "Discovery" Works (Under the Hood)
Discovery is the phase where the AI Client (e.g., Claude Desktop, Cursor) connects to the MCP Server and learns what it can do. It functions exactly like **REST API Swagger Documentation (`/openapi.js[...]

### The Handshake Steps:
1. **The Request:** The AI Client initiates a handshake via `STDIO` or `SSE` and asks: *"What tools do you have?"*
2. **The Response:** The MCP Server replies with a structured **JSON Schema** containing the **Name**, **Description**, and **Input Arguments** of all available tools.
3. **The Magic of Python FastMCP:** FastMCP automatically extracts metadata from your Python **Type Hints** and **Docstrings** to generate this JSON schema.

---

## 💻 3. Practical Code Example: ML Ops Monitor Server
Here is a production-style example of an MCP Server that exposes an ML metrics tracking tool with input arguments.

### Step 1: Installation
```bash
pip install mcp psutil
```

### Step 2: Python Code (`ml_mcp_server.py`)
```python
import random
from mcp.server.fastmcp import FastMCP

# 1. Initialize the MCP Server
mcp = FastMCP("ML Ops Monitor")

# 2. Define a tool with input arguments, type hints, and docstrings
@mcp.tool()
def fetch_training_metrics(run_id: str, last_n_epochs: int = 5) -> str:
    """
    Fetches historical loss and accuracy metrics for a specific ML training run.
    
    Args:
        run_id: The unique identifier string for the training run (e.g., 'run_042').
        last_n_epochs: Number of recent epochs to fetch data for. Defaults to 5.
    """
    # Simulated API call to MLflow or Weights & Biases (WandB)
    results = []
    for epoch in range(1, last_n_epochs + 1):
        loss = round(random.uniform(0.1, 0.5), 4)
        accuracy = round(random.uniform(0.75, 0.99), 4)
        results.append(f"Epoch {epoch}: Loss={loss}, Accuracy={accuracy}")
        
    metrics_summary = "\n".join(results)
    return f"Metrics for {run_id}:\n{metrics_summary}"

if __name__ == "__main__":
    # 3. Run the server
    mcp.run()
```

### Generated JSON Schema (What the AI actually "discovers"):
```json
{
  "name": "fetch_training_metrics",
  "description": "Fetches historical loss and accuracy metrics for a specific ML training run.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "run_id": {
        "type": "string",
        "description": "The unique identifier string for the training run (e.g., 'run_042')."
      },
      "last_n_epochs": {
        "type": "integer",
        "description": "Number of recent epochs to fetch data for. Defaults to 5.",
        "default": 5
      }
    },
    "required": ["run_id"]
  }
}
```

---

## 🛠️ 4. How Developers Think: Designing MCP Tools
Developers write tools based on the AI's **Blind Spots (What it can't see/do)**. Ask yourself three questions when designing a tool:

1. **Data Fetching:** Does the AI need real-time or internal company data? *(e.g., `check_gpu_memory_usage()`)*
2. **Action Execution:** Does the AI need to change the state of a system? *(e.g., `update_config_yaml()`, `trigger_jenkins_build()`)*
3. **Computation Efficiency:** Is the task heavy on math/string formatting? Let Python do it perfectly instead of LLM guessing. *(e.g., `calculate_f1_score()`)*

---

## 🧠 5. Generic System Prompt for Tool Usage & Discovery
*Copy and paste this prompt into any advanced LLM system prompt to make it act like a proper MCP Client/Agent.*

```text
# SYSTEM PROMPT: MCP-STYLE TOOL SELECTION & AGENTIC WORKFLOW

You are an expert AI Developer Assistant equipped with Tool-Use/Function-Calling capabilities inspired by the Model Context Protocol (MCP). 

## 1. TOOL DISCOVERY MECHANISM
- You have access to external tools exposed via JSON schemas.
- Every tool contains a precise `name`, a semantic `description`, and an `inputSchema` defining arguments.
- Do not guess or hallucinate tool names. Only use tools explicitly discovered in your context.

## 2. SEMANTIC MATCHING & EXECUTION
- Analyze the user's intent semantically. Match their request to the most specific tool description available.
- If a user request implies an action or data fetch that matches a tool description, you MUST generate a tool call instead of guessing the answer textually.
- Extract required parameters strictly from the user's prompt. 

## 3. PARAMETER & INPUT VALIDATION
- Identify which parameters are mandatory (`required`) and which are optional (`default`).
- If a required parameter is missing from the user's input, do NOT execute the tool. Instead, ask the user politely to provide the missing specific detail.
- Ensure type safety (e.g., do not pass a string to a parameter expecting an integer).

## 4. MULTI-STEP CHAINING
- If a task requires multiple tools, chain them logically (e.g., Tool A fetches the ID -> Use that ID in Tool B to fetch the metrics).
- Present the final execution results to the user in a clean, human-readable layout.
```
