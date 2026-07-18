# Lab 04 — Integrate an AI Agent with Foundry IQ

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab builds a
`product-expert-agent` in the Microsoft Foundry portal, grounds it in a Contoso
outdoor/camping product catalog via **Foundry IQ** (a knowledge base backed by
Azure AI Search + Blob Storage), and connects to it from a Python client with a
human-in-the-loop approval flow for knowledge base lookups.

## What this demonstrates

- Creating a Foundry project and agent in the **new Foundry portal UI**
- Building a **Foundry IQ knowledge base** from PDFs in Azure Blob Storage,
  connected through an Azure AI Search resource (embeddings + chat completions
  model, API-key authentication)
- Configuring the agent's Azure AI Search (Foundry IQ) tool to **require
  approval before every use**, via the Foundry Toolkit for VS Code extension
  (not exposed in the portal UI)
- Connecting to the agent from Python using `AIProjectClient` +
  `get_openai_client()`, and driving it through the **conversations API**
  (`conversations.create`, `conversations.items.create`, `responses.create`
  with an `agent_reference`)
- Handling the **MCP approval loop**: detecting an `mcp_approval_request` in
  the response output, prompting the user to approve/deny it, submitting an
  `mcp_approval_response`, and re-requesting the response
- Maintaining **client-side conversation history** and multi-turn context
  (e.g. a follow-up "how much do those cost?" resolves against the prior turn)

## Architecture

```
                 ┌───────────────────────────────────────┐
                 │   Microsoft Foundry Project (gpt-5)    │
                 │  ┌───────────────────────────────────┐│
                 │  │ product-expert-agent               ││
                 │  │  - instructions (Contoso outdoor/   ││
                 │  │    camping products, must cite)     ││
                 │  │  - Foundry IQ / Azure AI Search tool││
                 │  │    (require approval: always)       ││
                 │  └──────────────┬────────────────────┘│
                 └─────────────────┼──────────────────────┘
                                    │ Foundry IQ knowledge base
                                    ▼
                    Azure AI Search  ◀──── Azure Blob Storage
                    (ks-contosoproducts)    (contosoproducts container:
                                              3 Contoso product PDFs)
                                    ▲
                                    │ Responses API (conversations) +
                                    │ mcp_approval_request/response
                         agent_client.py (VS Code + Python, local)
```

## Repo contents

| Path | Description |
|---|---|
| `python/agent_client.py` | Connects to the Foundry project/agent, creates a conversation, sends user messages, and handles the MCP approval flow for Foundry IQ tool calls |
| `python/requirements.txt` | Python dependencies (`azure-ai-projects==2.0.0b4`, `azure-identity`, `python-dotenv`) |
| `python/.env` | Environment variables (`PROJECT_ENDPOINT`, `AGENT_NAME`) — not committed with real values |
| `data/` | Sample Contoso product documents (tents, backpacks, camping accessories) used to build the Foundry IQ knowledge base — provided as both `.md` and `.pdf` |

## Portal setup (not code — done in Foundry/Azure portal)

- Foundry project with a `gpt-5` model deployment and `product-expert-agent`
- Azure AI Search resource + Azure Storage account (`contosoproducts` blob
  container holding the 3 product PDFs)
- Foundry IQ knowledge base `ks-contosoproducts` (API-key auth, `text-embedding-3-small`
  embeddings, `gpt-5` chat completions)
- Agent's Azure AI Search tool set to **"Ask for approval for all tools"** via
  the Foundry Toolkit for VS Code extension's Agent Builder

## Example interactions tested

- *"What types of outdoor products does Contoso offer?"* — agent requests
  approval, then synthesizes an answer across multiple catalog documents
- *"Tell me about the weatherproof features of your tents."* — pulls specific
  details from the tents catalog
- *"What's the difference between your daypacks and expedition backpacks?"* —
  compares information from the backpacks guide
- *"What camping accessories would you recommend for a weekend hiking trip?"* —
  recommendation grounded in the accessories document
- *"How much do those items typically cost?"* — follow-up resolved using
  client-side conversation history/context from the prior turn

## Running it locally

```bash
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint
# AGENT_NAME should match the agent created in the portal (e.g. "product-expert-agent")

python agent_client.py
```

Note: this lab was set up with a **Python 3.12** virtual environment for
consistency with the other labs in this repo.

At each Foundry IQ lookup the app prints the tool name, server, and arguments,
then prompts `Approve this action? (yes/no)` before the agent proceeds. Type
`history` to view the full conversation, `quit` to exit.

Requires the agent (`product-expert-agent`) and its Foundry IQ knowledge base
to already exist in the portal — this app only connects to and drives an
existing agent, it does not create one. Authentication is via Azure CLI
credentials (`az login`) — no API keys are stored in this repo.

## Stack

Microsoft Foundry · Foundry IQ · Azure AI Search · Azure Blob Storage · Azure AI
Agents (Responses/Conversations API) · MCP approval flow · Foundry Toolkit for
VS Code · Python · `azure-ai-projects` · `azure-identity`

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored.
- The Foundry IQ tool is configured to require approval for every call, so the
  agent never searches the knowledge base without the user seeing and
  confirming the request first.
