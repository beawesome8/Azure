# Lab 02 — Build an Agent with Custom Tools

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab builds an
astronomy observations agent on **Microsoft Foundry** that uses custom Python
functions as tools, giving the agent the ability to look up astronomical events,
calculate telescope rental costs, and generate observation reports.

## What this demonstrates

- Defining custom Python functions and exposing them to an agent as **function
  tools**, each described with a JSON schema (name, parameters, types,
  descriptions)
- Connecting to a Microsoft Foundry project and OpenAI-compatible client with
  `azure-ai-projects` + `azure-identity`
- Creating an agent (`PromptAgentDefinition`) with multiple function tools attached
- Running a multi-turn conversation loop that:
  - Sends user prompts to the agent via the Responses API
  - Detects `function_call` items in the agent's response
  - Executes the corresponding local Python function with the agent-supplied
    arguments
  - Sends the function output back to the agent as a `function_call_output`
  - Lets the agent chain multiple tool calls in a single turn (e.g. look up an
    event *and* calculate a cost) before producing a final answer
- Generating and saving a text file report as a side effect of a tool call
- Cleaning up by deleting the agent version when the session ends

## Architecture

```
                 ┌─────────────────────────┐
                 │   Microsoft Foundry      │
                 │   Project (gpt-5)        │
                 │                          │
                 │  ┌────────────────────┐  │
                 │  │ astronomy-agent     │  │
                 │  │  - instructions     │  │
                 │  │  - next_visible_event      │
                 │  │  - calculate_observation_cost │
                 │  │  - generate_observation_report │
                 │  └────────────────────┘  │
                 └───────────┬──────────────┘
                              │ Responses API (conversations)
                              │
                     agent.py (VS Code + Python, local)
                              │
                     functions.py — local execution of
                     function_call requests from the agent
```

## Repo contents

| Path | Description |
|---|---|
| `python/agent.py` | Console client that connects to the Foundry project, defines the function tools, creates the agent, and runs the chat loop |
| `python/functions.py` | The actual Python implementations of the three tools: `next_visible_event`, `calculate_observation_cost`, `generate_observation_report` |
| `python/data/` | Sample data files (astronomical events, telescope rates, priority multipliers) used by the functions |
| `python/requirements.txt` | Python dependencies |
| `python/.env` | Environment variables (`PROJECT_ENDPOINT`, `MODEL_DEPLOYMENT_NAME`) — git-ignored, not committed |

## Function tools

| Tool | Purpose |
|---|---|
| `next_visible_event` | Looks up the next astronomical event visible from a given continent/location |
| `calculate_observation_cost` | Calculates telescope rental cost from tier, hours, and priority level |
| `generate_observation_report` | Combines the above into a formatted `.txt` report saved locally |

## Example interactions tested

- *"Find me the next event I can see from South America and give me the cost for
  5 hours of premium telescope time at normal priority."* — agent calls
  `next_visible_event` and `calculate_observation_cost` in the same turn
- *"Generate that information in a report for Bellows College."* — agent calls
  `generate_observation_report`, which writes `report_<event>_<timestamp>.txt`
  locally

## Running it locally

```bash
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint
# MODEL_DEPLOYMENT_NAME should match your deployed model (e.g. "gpt-5")

python agent.py
```

Requires an existing Microsoft Foundry project with a `gpt-5` (or equivalent)
model deployment. The agent (`astronomy-agent`) is created and deleted
automatically by `agent.py` on each run — no manual agent setup is needed in the
portal. Authentication is via Azure CLI credentials (`az login`) — no API keys
are stored in this repo.

## Stack

Microsoft Foundry · Azure AI Agents (Responses API) · Foundry Toolkit for VS Code ·
Python · `azure-ai-projects` · `azure-identity` · Custom function tools

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored.
- Generated report files (`report_*.txt`) are written to the `python/` folder at
  runtime and are not committed.
