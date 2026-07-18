# Lab 05b — Work IQ: Workplace Intelligence for AI Agents

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab builds
a `workplace-intelligence-agent` on Microsoft Foundry that uses **Work IQ** —
Microsoft's MCP server exposing contextual M365 Copilot data (emails, meetings,
Teams messages) — as agent tools, so the agent can answer real workplace
questions grounded in the user's own M365 data.

Note: this lab requires a **Microsoft 365 Copilot license**. Standard M365
accounts without Copilot cannot run the live scenarios; the code and README
here still document the patterns for reference.

## What this demonstrates

- Installing and authenticating **Work IQ** (`npm install -g @microsoft/workiq`,
  `workiq accept-eula`), including the IT-admin consent flow for organizational
  accounts
- Launching the Work IQ **MCP server as a subprocess per operation** via
  `StdioServerParameters(command="npx", args=["-y", "@microsoft/workiq", "mcp"])`
  rather than holding a persistent connection
- Converting MCP tool definitions (`list_tools()`) into `FunctionTool` objects
  and attaching them to a `PromptAgentDefinition`
- Driving the agent through the **Responses/Conversations API** (not the older
  Threads/Runs pattern), including the tool-call loop: detect `function_call`
  items, execute the matching Work IQ MCP tool, feed the result back as a
  `FunctionCallOutput`, and repeat until the agent returns a final answer
- Five workplace intelligence scenarios in a single interactive app:
  1. **Meeting Prep** — gather attendee/agenda/related-email context for a
     meeting
  2. **Project Status** — synthesize project updates across email, Teams, and
     meetings
  3. **Action Items** — extract and prioritize open tasks from multiple sources
  4. **Combined Intelligence** — cross-reference Work IQ (workplace data) with
     Foundry IQ (an indexed knowledge base) for the same topic
  5. **Custom Query** — free-form workplace questions
- Cleaning up by deleting the agent version on exit — no Azure resources are
  provisioned by this lab (Work IQ rides on the user's M365 Copilot license)

## Architecture

```
                 ┌─────────────────────────────────────────┐
                 │   Microsoft Foundry Project (gpt-5)      │
                 │  ┌─────────────────────────────────────┐│
                 │  │ workplace-intelligence-agent          ││
                 │  │  - instructions                       ││
                 │  │  - Work IQ tools (FunctionTool[])     ││
                 │  └───────────────┬───────────────────────┘│
                 └──────────────────┼───────────────────────┘
                                     │ Responses API (function_call loop)
                          workiq_lab.py (VS Code + Python, local)
                                     │ stdio (per call)
                                     ▼
                     npx @microsoft/workiq mcp  ──▶  Microsoft 365 Copilot
                                                       (emails, meetings, Teams,
                                                        via the user's license)
```

## Repo contents

| Path | Description |
|---|---|
| `python/workiq_lab.py` | Interactive application: validates Work IQ setup, connects to the Foundry project, discovers Work IQ MCP tools, creates the agent, and runs the 5-scenario menu (complete as provided, no code edits required) |
| `python/requirements.txt` | Python dependencies (`azure-ai-projects`, `azure-identity`, `python-dotenv`, `mcp`) |
| `python/.env` | Environment variables (`PROJECT_ENDPOINT`, `MODEL_DEPLOYMENT_NAME`) — not committed with real values |

## Example interactions tested (require M365 Copilot data)

- Meeting Prep: *"my 2pm meeting"* — pulls attendees/agenda plus related emails
  and prior meetings on the topic
- Project Status: *"Website redesign"* — status synthesized from emails, Teams
  messages, and meeting outcomes
- Action Items: *"this week"* — tasks pulled from meeting notes, emails, and
  Teams mentions, prioritized by urgency
- Combined Intelligence: *"remote work policy"* — compares informal Work IQ
  discussion with the official Foundry IQ-indexed policy document (requires a
  configured knowledge base, e.g. from [lab 04](../04-integrate-agent-with-foundry-iq/))
- Custom Query: *"What was decided in yesterday's standup?"*

## Running it locally

```bash
# 1. Install and verify Work IQ (Node.js 18+ required)
npm install -g @microsoft/workiq
workiq accept-eula
workiq ask -q "What meetings do I have today?"

# 2. Set up the Python app
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint
# MODEL_DEPLOYMENT_NAME should match your deployed model (e.g. "gpt-5")

python workiq_lab.py
```

Note: this lab was set up with a **Python 3.12** virtual environment for
consistency with the other labs in this repo. If `workiq ask` reports "Admin
consent required," an org admin must approve the consent URL it prints before
the app can query real M365 data; if it reports "No M365 Copilot license," the
live scenarios can't run — read the code/README to understand the pattern
instead.

## Stack

Microsoft Foundry · Work IQ (M365 Copilot MCP server) · Model Context Protocol
(MCP) · Azure AI Agents (Responses/Conversations API) · Python ·
`azure-ai-projects` · `azure-identity` · `mcp`

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored.
- No Azure resources are created by this lab — Work IQ operates against the
  user's existing M365 Copilot license, so cleanup is limited to deleting the
  Foundry agent version, which the app does automatically on exit.
