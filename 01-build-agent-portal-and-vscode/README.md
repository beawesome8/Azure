# Lab 01 — Build AI Agents with Portal and VS Code

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab builds an
IT support agent on **Microsoft Foundry** using both the no-code Foundry portal and
the Foundry Toolkit extension for VS Code, then drives that agent programmatically
from a Python client.

## What this demonstrates

- Provisioning a Microsoft Foundry project and resource, and deploying a model
- Authoring agent instructions (system prompt) and grounding the agent in a
  domain-specific knowledge document via **File Search**
- Enabling the **Code Interpreter** tool so the agent can run Python against an
  uploaded dataset (statistics, trend analysis, chart generation)
- Testing an agent conversationally, both in the Foundry portal playground and in
  the Foundry Toolkit extension inside VS Code
- Writing a Python client (`azure-ai-projects` + `azure-identity`) that:
  - Authenticates with `DefaultAzureCredential` (Azure CLI / `az login`)
  - Loads an existing agent by name from a Foundry project
  - Opens a multi-turn conversation via the OpenAI-compatible Responses API
  - Downloads and saves files/images the agent generates via Code Interpreter
    (e.g. matplotlib charts) to a local `agent_outputs/` folder

## Architecture

```
                 ┌─────────────────────────┐
                 │   Microsoft Foundry      │
                 │   Project                │
                 │                          │
                 │  ┌────────────────────┐  │
                 │  │ it-support-agent    │  │
                 │  │  - instructions     │  │
                 │  │  - File Search  ────┼──┼── IT_Policy.txt
                 │  │  - Code Interpreter ┼──┼── system_performance.csv
                 │  └────────────────────┘  │
                 └───────────┬──────────────┘
                              │ Responses API (conversations)
              ┌───────────────┴───────────────┐
              │                                │
   Foundry portal playground        agent_with_functions.py
   (browser)                        (VS Code + Python, local)
```

## Repo contents

| Path | Description |
|---|---|
| `python/agent_with_functions.py` | Console client that connects to the deployed agent and chats with it, saving any generated charts/files locally |
| `python/requirements.txt` | Python dependencies |
| `python/.env.example` | Template for the required environment variables (`PROJECT_ENDPOINT`, `AGENT_NAME`) |
| `sample-data/IT_Policy.txt` | Grounding document used for File Search — sample Contoso IT policy (password resets, software requests, hardware troubleshooting, etc.) |
| `sample-data/system_performance.csv` | Simulated CPU/memory/disk/network metrics used to exercise Code Interpreter |
| `screenshots/` | Portal + VS Code screenshots from the working session |

## Agent configuration

**Instructions given to the agent:**

```
You are an IT Support Agent for Contoso Corporation.
You help employees with technical issues and IT policy questions.

Guidelines:
- Always be professional and helpful
- Use the IT policy documentation to answer questions accurately
- If you don't know the answer, admit it and suggest contacting IT support directly
- When creating tickets, collect all necessary information before proceeding
```

**Tools enabled:** File Search (grounded on `IT_Policy.txt`), Code Interpreter
(grounded on `system_performance.csv`).

## Example interactions tested

- *"What's the policy for password resets?"* — answered via File Search grounding
- *"How do I request new software?"* — answered via File Search grounding
- *"Can you analyze the system performance data and tell me if there are any
  concerning trends?"* — Code Interpreter runs analysis over the CSV
- *"Create a chart showing CPU usage over time from the performance data"* —
  Code Interpreter generates a matplotlib chart, downloaded locally by the client
- *"Find any correlation between high CPU usage and memory usage in the
  performance data"* — Code Interpreter statistical analysis

## Running it locally

```bash
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

cp .env.example .env
# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint

python agent_with_functions.py
```

Requires an existing Microsoft Foundry project with an agent named
`it-support-agent` (or override via `AGENT_NAME`) already configured with File
Search + Code Interpreter, as described above. Authentication is via Azure CLI
credentials (`az login`) — no API keys are stored in this repo.

## Stack

Microsoft Foundry · Azure AI Agents (Responses API) · Foundry Toolkit for VS Code ·
Python · `azure-ai-projects` · `azure-identity` · Code Interpreter · File Search (RAG)

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored; only `.env.example` is tracked.
- Resources were deprovisioned after testing (`Settings > Delete project` /
  resource group deletion) to avoid ongoing Azure charges.
