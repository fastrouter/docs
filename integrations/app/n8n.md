---
description: Track usage, control costs, and add guardrails to your n8n AI workflows
icon: diagram-project
---

# n8n

### What is n8n?

[n8n](https://n8n.io/) is a workflow-automation platform that connects apps, APIs, and AI models into automated pipelines. With its visual node-based editor, you can build complex automations without writing code. The AI nodes are built on LangChain, and the OpenAI nodes accept a custom base URL—so they work with any OpenAI-compatible endpoint.

By routing n8n through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers pointing n8n's OpenAI credential at FastRouter, model selection, team governance, and production feature usage.

#### Prerequisites

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* An n8n instance (cloud or self-hosted)

***

### Quick Start

#### Step 1: Get Your FastRouter API Key

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

#### Step 2: Create an OpenAI Credential Pointing at FastRouter

1. In n8n, go to **Credentials** → **Add credential** and choose **OpenAI**.
2. Set **API Key** to your FastRouter API key.
3.  Expand the credential options and set **Base URL** to:

    ```
    https://api.fastrouter.ai/api/v1
    ```
4. Save the credential.



#### Step 3: Add an AI Node and Select the Credential

1. Add an **AI Agent** node (or any node that uses a **Chat Model**) to your workflow.
2. Add a **Chat OpenAI** model sub-node and select the FastRouter credential you just created.
3. In the model field, enter a FastRouter model slug, for example `openai/gpt-5.2`.

> **Note:** Because FastRouter slugs aren't in n8n's built-in OpenAI model list, type the slug directly (for example `anthropic/claude-4.5-sonnet`) rather than relying on the dropdown.

#### Step 4: Run the Workflow

Execute the workflow. The request routes through FastRouter, and every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-26 at 6.53.48 PM.png" alt=""><figcaption></figcaption></figure>

***

### FAQs

#### Configuration & Setup

**The node fails with an authentication or "model not found" error. What should I check?**

Confirm the credential's **Base URL** is exactly `https://api.fastrouter.ai/api/v1`, the API key is correct, and the model field contains a valid FastRouter slug including its provider prefix (for example `openai/gpt-5.2`, not `gpt-5.2`).

**Can I use the same credential across many workflows?**

Yes. One FastRouter credential works for every OpenAI/AI node in your instance. Switch models per node without changing the credential.

**Can I use n8n with my own provider keys (BYOK)?**

Yes. Set up an External Key integration in the FastRouter dashboard, then route n8n traffic through it. You retain your provider's pricing and rate limits while gaining FastRouter's routing and observability layer.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

#### Costs & Budgeting

**What happens when a key exceeds its budget?**

FastRouter blocks further requests until the budget resets (if a reset interval is configured) or an admin increases the limit. The workflow receives an error response indicating the budget has been exceeded.

**How do I track n8n spending by workflow or team?**

Create separate projects for each team, issue project-scoped API keys per workflow, and use Dynamic Tags (via Code nodes) for finer-grained attribution. The Dashboard breaks down costs by project, key, model, and tag.

#### Performance & Reliability

**Does FastRouter add latency to a workflow run?**

FastRouter adds near-zero gateway overhead. For most workflows, this is negligible compared to model inference time.

#### Privacy & Security

**Is my data sent to FastRouter's servers?**

FastRouter acts as a pass-through gateway. Requests are routed to the model provider and responses are returned to your client. Content logging can be disabled per key for sensitive workloads. See the **Disable Content Logging** option in key settings.

***

### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
