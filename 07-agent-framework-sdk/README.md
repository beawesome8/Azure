# Lab 07 — Develop an Azure AI Chat Agent with the Microsoft Agent Framework SDK

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab builds
an `ExpenseClaimAgent` using the **Microsoft Agent Framework** SDK (`agent_framework`)
instead of the raw `azure-ai-projects` Responses API used in earlier labs — a
higher-level, tool-decorator-based abstraction over the same Foundry agent
backend. The agent reads expense line items from a local file and drafts a
submission email via a custom tool function.

## What this demonstrates

- Using the **Microsoft Agent Framework** SDK's `Agent` + `FoundryChatClient`
  classes instead of `AIProjectClient`/`get_openai_client()` directly
- Defining a custom tool with the **`@tool` decorator** (`agent_framework.tool`),
  using `pydantic.Field` + `typing.Annotated` for self-describing parameter
  schemas (`to`, `subject`, `body`) — a more declarative alternative to the
  hand-written JSON-schema `FunctionTool` definitions in labs 02/03
- `approval_mode="never_require"` — running a tool without a human-in-the-loop
  approval step (contrast with the MCP approval flows in labs 03–05)
- Authenticating the chat client with `AzureCliCredential` (vs.
  `DefaultAzureCredential` elsewhere in this repo)
- Using `async with Agent(...) as agent:` for scoped agent lifecycle management,
  and `await agent.run(prompt_messages)` to invoke it — a simpler call shape
  than manually managing conversations/responses and function-call loops
- Feeding unstructured local file data (`data.txt`, a CSV of expense rows) into
  the agent's prompt and letting it itemize, total, and draft an email body

## Architecture

```
                 ┌─────────────────────────────────────────┐
                 │   Microsoft Foundry Project (gpt-5)      │
                 │  ┌─────────────────────────────────────┐│
                 │  │ ExpenseClaimAgent                    ││
                 │  │  - instructions (draft + "send"      ││
                 │  │    expense claim email)               ││
                 │  │  - tool: submit_claim(to, subject,    ││
                 │  │    body) — @tool, never_require       ││
                 │  └───────────────┬───────────────────────┘│
                 └──────────────────┼───────────────────────┘
                                     │ agent_framework.foundry.FoundryChatClient
                                     │ (AzureCliCredential)
                        agent-framework.py (VS Code + Python, local)
                                     │
                                     ▼
                          data.txt (expense line items)
```

## Repo contents

| Path | Description |
|---|---|
| `python/agent-framework.py` | Loads `data.txt`, prompts the user, creates a `FoundryChatClient` + `Agent` with the `submit_claim` tool, and runs the agent against the expense data |
| `python/data.txt` | Sample expense line items (date, description, amount CSV) |
| `python/requirements.txt` | Python dependencies (`agent-framework==1.9.0`, `azure-identity==1.25.3`, `python-dotenv`) |
| `python/.env` | Environment variables (`PROJECT_ENDPOINT`, `MODEL_DEPLOYMENT_NAME`) — not committed with real values |

## Example interaction tested

Prompt: *"Submit an expense claim"* — the agent itemizes the taxi/dinner/hotel
rows from `data.txt`, computes the total, and calls `submit_claim` with a
subject of `Expense Claim` and an itemized body addressed to
`expenses@contoso.com`. The tool simulates sending the email by printing the
`To`/`Subject`/`body` to the console (no real SMTP call), and the agent then
confirms the submission back to the user.

## Running it locally

```bash
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint
# MODEL_DEPLOYMENT_NAME should match your deployed model (e.g. "gpt-5")

python agent-framework.py
```

Note: this lab was set up with a **Python 3.12** virtual environment for
consistency with the other labs in this repo. Authentication uses
`AzureCliCredential`, so `az login` must be run first — `DefaultAzureCredential`
is not used here.

## Stack

Microsoft Foundry · Microsoft Agent Framework SDK (`agent_framework`) ·
`FoundryChatClient` · Python · `azure-identity` · `pydantic`

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored.
- This lab intentionally uses a different (higher-level) SDK surface than labs
  01–06, which call `azure-ai-projects`'s Responses/Conversations API directly —
  useful for comparing the two approaches to the same underlying Foundry agent
  service.
