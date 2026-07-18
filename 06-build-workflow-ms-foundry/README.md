# Lab 06 — Build a Workflow in Microsoft Foundry

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab builds a
**sequential workflow** in the Microsoft Foundry portal's (preview) workflow
builder — a UI-based orchestration of multiple AI agents and control-flow nodes
— that triages and drafts responses to customer support tickets for a fictional
company, ContosoPay. It's then invoked programmatically from a Python client.

## What this demonstrates

- Building a **multi-node workflow** in the Foundry portal visualizer:
  - `Set variable` — seeds a `SupportTickets` array with sample ticket text
  - `For each` — iterates the array, exposing the current item as `CurrentTicket`
  - `Agent` (Triage-Agent) — classifies each ticket into `Billing` / `Technical`
    / `General` with a confidence score, constrained to a **JSON Schema**
    response format
  - `If/Else` — branches on confidence (`> 0.6`) and, on the high-confidence
    path, again on category (`= "Billing"`)
  - `Deliver a message` — sends fixed messages for the low-confidence and
    billing-escalation paths
  - `Agent` (Resolution-Agent) — drafts a category-appropriate support response
    for non-billing tickets
- Referencing workflow variables and node outputs with expressions
  (`Local.SupportTickets`, `Local.TriageOutputJson.confidence`, etc.)
- Previewing/testing the workflow interactively inside the Foundry portal
- **Invoking the published workflow from code** using `AIProjectClient` +
  `get_openai_client()`, treating the workflow itself as an `agent_reference`
  by name
- Streaming workflow execution (`responses.create(..., stream=True)`) and
  handling `response.completed` events to retrieve and print final output
- Parsing the workflow's combined JSON + text output per ticket with a small
  regex-based formatter (`print_workflow_output`)
- Cleaning up by deleting the conversation when done

## Architecture

```
                 ┌─────────────────────────────────────────────────────┐
                 │  Microsoft Foundry Workflow                         │
                 │  ContosoPay-Customer-Support-Triage                 │
                 │                                                     │
                 │  Set variable (SupportTickets[])                    │
                 │        │                                            │
                 │  For each ticket ──▶ CurrentTicket                  │
                 │        │                                            │
                 │  Agent: Triage-Agent  ──▶ TriageOutputJson/Text     │
                 │   (category + confidence, JSON Schema output)       │
                 │        │                                            │
                 │  If confidence > 0.6 ?                              │
                 │   ├─ No  ──▶ Deliver message: "request more info"   │
                 │   └─ Yes ──▶ If category = "Billing" ?              │
                 │               ├─ Yes ──▶ Deliver message: escalate  │
                 │               └─ No  ──▶ Agent: Resolution-Agent    │
                 │                           ──▶ ResolutionOutputText  │
                 └─────────────────────────┬───────────────────────────┘
                                            │ Responses API (agent_reference
                                            │ = workflow name), streamed
                                 workflow.py (VS Code + Python, local)
```

## Repo contents

| Path | Description |
|---|---|
| `python/workflow.py` | Connects to the Foundry project, creates a conversation, invokes the `ContosoPay-Customer-Support-Triage` workflow by name as an agent reference, streams the run, and prints formatted per-ticket output |
| `python/requirements.txt` | Python dependencies (`azure-ai-projects==2.0.0b4`, `azure-identity`, `python-dotenv`, `aiohttp`) |
| `python/.env` | Environment variables (`PROJECT_ENDPOINT`) — not committed with a real value |

## Portal setup (built in the Foundry workflow visualizer, not code)

- `Set variable`: `SupportTickets` — a hardcoded array of 3 sample ContosoPay
  support tickets (an API 403 error, a CSV export question, a duplicate charge)
- `For each`: loops `SupportTickets` into `CurrentTicket`
- `Triage-Agent`: classifies into Billing/Technical/General with a confidence
  score, constrained by a strict JSON Schema response format
- `If/Else` (confidence): routes low-confidence tickets to a "request more
  info" message
- `If/Else` (category): routes Billing tickets to a human-escalation message
- `Resolution-Agent`: drafts the customer-facing response for Technical/General
  tickets

## Example output tested

```
Ticket 1: Technical (100% confidence)
Issue: API returns a 403 error when creating invoices, API key unchanged.

Response:
Thank you for contacting us about the 403 error when creating invoices with
the API. This error typically relates to permission issues. Please ensure
your API key has the necessary permissions for invoice creation and that the
endpoint URL is correct...
```

- A CSV export question routes to `General` and gets a short how-to response
- A duplicate-charge ticket routes to `Billing` and is escalated to human
  support instead of receiving an automated draft

## Running it locally

```bash
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint
# (copy it from the workflow visualizer's Code > .env variables tab —
# AZURE_EXISTING_AIPROJECT_ENDPOINT)

python workflow.py
```

Note: this lab was set up with a **Python 3.12** virtual environment for
consistency with the other labs in this repo (the lab notes 3.13 dependency
compatibility gaps for some packages).

Requires the workflow (`ContosoPay-Customer-Support-Triage`) and its two agents
(`Triage-Agent`, `Resolution-Agent`) to already exist in the Foundry portal —
this app only invokes an existing published workflow, it does not build one.
Authentication is via Azure CLI credentials (`az login`) — no API keys are
stored in this repo.

## Stack

Microsoft Foundry · Foundry workflow builder (preview) · Azure AI Agents
(Responses/Conversations API, streamed) · Python · `azure-ai-projects` ·
`azure-identity`

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored.
- The workflow builder is in preview; the portal-side configuration (nodes,
  agents, conditions) isn't represented as code in this repo — only the client
  that invokes the published workflow is.
