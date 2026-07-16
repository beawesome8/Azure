# Lab 03 — Connect MCP Tools to Azure AI Agents

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab extends a
Microsoft Foundry agent with **Model Context Protocol (MCP)** tools, in two forms:
a remote, cloud-hosted MCP server (Microsoft Learn Docs) and a custom local MCP
server exposing inventory/sales tools to an "inventory-agent".

## What this demonstrates

- Connecting an Azure AI Agent directly to a **remote MCP server**
  (`https://learn.microsoft.com/api/mcp`) via `MCPTool`, with `require_approval`
  set to `"always"`
- Handling the `mcp_approval_request` / `mcp_approval_response` loop — an agent
  can issue several MCP tool calls per turn, each needing its own approval,
  auto-approved here in a loop until none remain
- Building a **custom local MCP server** (`server.py`) with `FastMCP`, exposing
  Python functions as discoverable tools via `@mcp.tool()`
- Implementing an **MCP client** (`client.py`) that starts the server over stdio
  transport (`stdio_client` + `ClientSession`), lists its tools, and wraps each
  one as an async callable
- Bridging MCP tools into the Responses API as `FunctionTool` definitions, so the
  agent can invoke local server-hosted tools like any other function tool
- Running a stateful, multi-turn chat loop where the agent reasons over live
  inventory/sales data to make restock/clearance recommendations
- Cleaning up by deleting the agent version when each session ends

## Architecture

```
Remote MCP (agent.py)
                 ┌─────────────────────────┐
                 │   Microsoft Foundry      │
                 │   Project (gpt-5)        │
                 │  ┌────────────────────┐  │        MCP (HTTP)
                 │  │ MyAgent             │──┼──▶ learn.microsoft.com/api/mcp
                 │  │  - MCPTool          │  │      (Microsoft Learn Docs)
                 │  └────────────────────┘  │
                 └───────────┬──────────────┘
                              │ Responses API + mcp_approval loop
                     agent.py (VS Code + Python, local)

Custom local MCP (server.py + client.py)
                 ┌─────────────────────────┐
                 │   Microsoft Foundry      │
                 │   Project (gpt-5)        │
                 │  ┌────────────────────┐  │
                 │  │ inventory-agent     │  │
                 │  │  - get_inventory_levels │
                 │  │  - get_weekly_sales │  │
                 │  └────────────────────┘  │
                 └───────────┬──────────────┘
                              │ Responses API (function_call / function_call_output)
                     client.py ──── stdio ────▶ server.py (FastMCP "Inventory")
```

## Repo contents

| Path | Description |
|---|---|
| `python/agent.py` | Connects to the remote Microsoft Learn Docs MCP server, creates an agent with an `MCPTool`, and auto-approves MCP tool-call requests |
| `python/server.py` | Custom local MCP server (`FastMCP`) exposing `get_inventory_levels` and `get_weekly_sales` as tools |
| `python/client.py` | MCP client that launches `server.py` over stdio, wraps its tools as `FunctionTool`s, creates `inventory-agent`, and runs an interactive chat loop |
| `python/requirements.txt` | Python dependencies (`mcp`, `fastmcp`, `azure-ai-projects`, `azure-identity`, `openai`, `python-dotenv`, `uvicorn`) |
| `python/.env` | Environment variables (`PROJECT_ENDPOINT`, `MODEL_DEPLOYMENT_NAME`) — not committed with real values |

## Example interactions tested

- *"Give me the Azure CLI commands to create an Azure Container App with a
  managed identity."* (`agent.py`) — agent calls the remote MCP tool to search
  Microsoft Learn docs
- *"Show me the current inventory levels for all products."* (`client.py`) —
  agent calls `get_inventory_levels`
- *"Are there any products that should be restocked?"* / *"Which products would
  you recommend for clearance?"* — agent calls both tools and applies the
  restock/clearance thresholds in its instructions

## Running it locally

```bash
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint
# MODEL_DEPLOYMENT_NAME should match your deployed model (e.g. "gpt-5")

python agent.py     # remote MCP server (Microsoft Learn Docs)
python client.py    # custom local MCP server (inventory tools)
```

Note: this lab was set up with a **Python 3.12** virtual environment rather than
3.13, since some dependencies (`fastmcp`/`mcp`) aren't yet compiled for 3.13 at
the time of writing.

Both agents (`MyAgent`, `inventory-agent`) are created and deleted automatically
on each run — no manual agent setup is needed in the portal. Authentication is
via Azure CLI credentials (`az login`) — no API keys are stored in this repo.

## Stack

Microsoft Foundry · Azure AI Agents (Responses API) · Model Context Protocol
(MCP) · `fastmcp` · `mcp` · Foundry Toolkit for VS Code · Python ·
`azure-ai-projects` · `azure-identity`

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored.
- The remote MCP tool requires per-call approval (`require_approval="always"`);
  `agent.py` auto-approves these in a loop for lab purposes.
