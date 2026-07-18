# Develop AI Agents on Azure

Hands-on labs from Microsoft's **Develop AI Agents on Azure** learning path, built
using Microsoft Foundry (formerly Azure AI Foundry), the Foundry Toolkit for
VS Code, and Python.

Each numbered folder is a self-contained lab with its own README describing what
was built, how it was tested, and how to run the code.

## Labs

| # | Lab | Focus |
|---|---|---|
| 01 | [Build AI Agents with Portal and VS Code](01-build-agent-portal-and-vscode/) | Creating a Foundry agent in the portal, grounding it with File Search + Code Interpreter, and driving it programmatically from a Python client in VS Code |
| 02 | [Build an Agent with Custom Tools](02-build-agent-with-custom-tools/) | Defining custom Python functions as agent tools, and handling multi-step function-calling conversations with the Responses API |
| 03 | [Connect MCP Tools to Azure AI Agents](03-connect-mcp-tools-to-azure-ai-agents/) | Connecting agents to a remote MCP server (Microsoft Learn Docs) and building a custom local MCP server/client with inventory tools |
| 04 | [Integrate an AI Agent with Foundry IQ](04-integrate-agent-with-foundry-iq/) | Grounding a portal-built agent in a Foundry IQ knowledge base (Azure AI Search + Blob Storage), with a human-in-the-loop approval flow for knowledge base lookups |
| 05a | [Deploy Agents to Teams and Copilot](05a-deploy-agents-to-teams-and-copilot/) | Publishing a portal-built agent to Microsoft Teams (Azure Bot Service) and Microsoft 365 Copilot, plus supporting scripts for Foundry IQ / Graph API concepts |
| 05b | [Work IQ: Workplace Intelligence](05b-work-iq-workplace-intelligence/) | Grounding an agent in live M365 Copilot data (emails, meetings, Teams) via the Work IQ MCP server, with a function-call tool loop over the Responses API |
| 06 | [Build a Workflow in Microsoft Foundry](06-build-workflow-ms-foundry/) | Building a multi-agent, multi-branch customer support triage workflow in the Foundry portal's workflow builder, and invoking it from Python as a streamed agent reference |

## Skills demonstrated across this repo

- Provisioning and configuring Microsoft Foundry projects and model deployments
- Designing agent instructions and grounding agents in domain documents (RAG /
  File Search)
- Using built-in agent tools (Code Interpreter) for data analysis and chart
  generation
- Developing agents with the Foundry Toolkit VS Code extension
- Building Python clients against the Azure AI Agents / Responses API
  (`azure-ai-projects`, `azure-identity`)
- Defining custom function tools with JSON-schema parameters and handling
  multi-step, agent-initiated function-calling loops
- Connecting agents to remote and custom local MCP (Model Context Protocol)
  servers, including the MCP tool-call approval flow
- Building Foundry IQ knowledge bases (Azure AI Search + Blob Storage) and
  grounding agents in them with human-in-the-loop approval for lookups
- Publishing agents to Microsoft Teams and Microsoft 365 Copilot from the
  Foundry portal, and understanding the resulting Azure Bot Service/app
  manifest artifacts
- Grounding agents in live Microsoft 365 data via the Work IQ MCP server, using
  the same function-call tool loop pattern as custom/remote MCP tools
- Building multi-agent, multi-branch workflows in the Foundry portal's
  workflow builder (loops, conditionals, JSON Schema-constrained agent
  outputs) and invoking them from code as streamed agent references
- Managing Azure resources responsibly (provisioning, testing, teardown)

## Environment

- Python 3.13 (3.12 for labs 03–06, due to dependency compatibility)
- VS Code + Foundry Toolkit extension
- Azure CLI (`az login`) for authentication — no API keys committed
