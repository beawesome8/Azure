# Lab 05a — Deploy Agents to Microsoft Teams and Copilot

Part of Microsoft's **Develop AI Agents on Azure** learning path. This lab is
primarily a **portal/publishing exercise**: building an `enterprise-knowledge-agent`
in the Foundry portal, grounding it in uploaded policy documents, and publishing
it to Microsoft Teams and Microsoft 365 Copilot — no custom agent code is
written for the publishing flow itself. The `python/` folder holds supporting
scripts (self-contained, no TODOs) that illustrate and validate the concepts
around Foundry IQ, Teams deployment, and Microsoft Graph integration.

## What this demonstrates

- Creating a Foundry project/agent (`enterprise-knowledge-agent`) in the **new
  Foundry portal UI** with instructions and uploaded knowledge documents
- Testing knowledge grounding in the playground (IT security + remote work
  policies)
- **Publishing an agent to Microsoft Teams**: the portal auto-creates an Azure
  Bot Service, generates a Teams app manifest, and packages icons/config into a
  downloadable app package
- Configuring Teams app metadata (name, descriptions, icons, developer info)
  and choosing a publish scope (Individual vs. Organization)
- **Publishing the same agent as a Microsoft 365 Copilot extension** (declarative
  agent/plugin), invokable via `@mentions` inside Copilot
- Understanding the difference between Shared and Organization publish scopes
  and their admin-approval requirements
- Supporting Python scripts that demonstrate the underlying patterns:
  Foundry IQ + Azure AI Search grounding, Teams Bot Framework/Adaptive Cards
  concepts, and Microsoft Graph API integration (SharePoint, Calendar, Mail)

## Architecture

```
                 ┌─────────────────────────────────────────┐
                 │   Microsoft Foundry Project (gpt-5)      │
                 │  ┌─────────────────────────────────────┐│
                 │  │ enterprise-knowledge-agent           ││
                 │  │  - instructions (Contoso policies)   ││
                 │  │  - uploaded docs (IT security,       ││
                 │  │    remote work policy)                ││
                 │  └───────────┬───────────────┬──────────┘│
                 └──────────────┼───────────────┼───────────┘
                                 │               │
                     Publish to Teams   Publish to M365 Copilot
                                 │               │
                        ┌────────▼──────┐ ┌──────▼─────────┐
                        │ Azure Bot      │ │ Copilot        │
                        │ Service +      │ │ extension /    │
                        │ Teams app      │ │ declarative    │
                        │ manifest       │ │ agent          │
                        └────────────────┘ └────────────────┘
```

## Repo contents

| Path | Description |
|---|---|
| `python/m365_teams_lab.py` | Self-contained interactive demo covering Foundry IQ, Teams deployment concepts, Graph API integration, and a live production-style agent demo (menu-driven, complete as provided) |
| `python/check_prerequisites.py` | Validates required CLI tools are installed before attempting deployment automation |
| `python/setup_search.py` | Automates creating an Azure AI Search resource and indexing the sample documents (optional path if not using the portal's Foundry IQ setup UI) |
| `python/deploy_helper.py` | Interactive wizard for deploying via Azure Developer CLI (`azd`) |
| `python/validate_deployment.py` | Checks that an `azd` deployment succeeded and prints next steps |
| `python/cleanup_all.py` | Removes Azure resources created by the deployment scripts, to avoid ongoing charges |
| `python/agent.yaml` | Declarative definition of `enterprise-knowledge-agent` (model, instructions, tools) mirroring what's configured in the portal |
| `python/sample_documents/` | Sample Contoso policy/reference docs used for knowledge grounding (IT security, remote work, company handbook, expense reporting) |
| `python/requirements.txt` | Python dependencies (`azure-ai-projects`, `azure-identity`, `python-dotenv`) |
| `python/.env` | Environment variables (`PROJECT_ENDPOINT`, `MODEL_DEPLOYMENT_NAME`) — not committed with real values |

## Portal setup (the actual lab — done in Foundry/Teams/Copilot, not code)

1. Create a Foundry project (`m365-lab`) and agent (`enterprise-knowledge-agent`)
   in the new Foundry portal experience
2. Set instructions and upload `it_security_policy.txt` /
   `remote_work_policy.txt` as knowledge grounding
3. Test grounding in the playground (password requirements, core hours,
   encryption requirements)
4. Publish → **Publish to Teams and Microsoft 365 Copilot**, fill in app name,
   descriptions, developer info, and 192×192 / 32×32 icons
5. Choose **Individual** scope for Teams (no admin approval needed for the lab)
   and test the agent inside Teams
6. Re-run the same publish flow with **Shared** scope for Microsoft 365 Copilot
   and test with `@Enterprise Knowledge Agent <question>`
7. Clean up: delete the agent in the portal (also removes the Azure Bot
   Service), uninstall the app from Teams, and note the Copilot extension
   becomes inactive automatically

## Running the supporting scripts locally

```bash
cd python
python -m venv labenv
./labenv/Scripts/Activate.ps1   # or `source labenv/bin/activate` on macOS/Linux
pip install -r requirements.txt
az login

# edit .env and set PROJECT_ENDPOINT to your Foundry project's endpoint
# MODEL_DEPLOYMENT_NAME should match your deployed model (e.g. "gpt-5")

python m365_teams_lab.py       # interactive concept walkthrough + live demo
python check_prerequisites.py  # optional: verify CLI tooling before azd-based deploy
```

Note: this lab was set up with a **Python 3.12** virtual environment for
consistency with the other labs in this repo. `setup_search.py` additionally
requires `azure-search-documents`, which isn't in `requirements.txt` — install
it separately (`pip install azure-search-documents`) only if you choose the
scripted Azure AI Search path instead of the portal's Foundry IQ setup UI.

## Stack

Microsoft Foundry · Azure Bot Service · Microsoft Teams · Microsoft 365 Copilot ·
Foundry IQ · Microsoft Graph API (concepts) · Python · `azure-ai-projects` ·
`azure-identity`

## Notes

- No secrets, endpoints, or subscription identifiers are committed to this repo —
  `.env` is git-ignored.
- The core deliverable of this lab is the portal-driven publishing workflow;
  the Python scripts are supporting/optional automation and concept demos, not
  a fill-in-the-blank coding exercise.
