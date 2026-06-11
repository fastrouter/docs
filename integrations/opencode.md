---
description: Track usage, control costs, and add guardrails to your OpenCode coding agent
icon: square-terminal
---

# OpenCode

### What is OpenCode?

[OpenCode](https://opencode.ai/) is an open-source coding agent that runs in your terminal, IDE, or desktop. It handles coding tasks, file operations, and multi-step workflows through natural language, and supports multiple model providers through its built-in provider settings.

By routing OpenCode through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers OpenCode desktop configuration with FastRouter, model selection, team governance, and production feature usage. It does not cover the OpenCode CLI or IDE plugins.

#### Prerequisites

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* OpenCode desktop app installed ([download](https://opencode.ai/download))

***

### Quick Start for OpenCode Desktop App

#### Step 1: Install OpenCode Desktop App

Download [OpenCode](https://opencode.ai/download) and install it on your system.

#### Step 2: Get Your FastRouter API Key

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

#### Step 3: Open OpenCode Settings

Launch the OpenCode desktop app and open **Settings**.

<img src="../.gitbook/assets/unknown (45).png" alt="" height="212" width="624">

#### Step 4: Go to Providers

In the Settings sidebar, select **Providers**.

<img src="../.gitbook/assets/unknown (46).png" alt="" height="252" width="624">

#### Step 5: Show More Providers

Click **Show more Providers** to expand the full provider list.

<img src="../.gitbook/assets/unknown (47).png" alt="" height="185" width="624">

#### Step 6: Select FastRouter

Search for or scroll to **FastRouter** in the provider list and select it.

<img src="../.gitbook/assets/unknown (48).png" alt="" height="141" width="624">

#### Step 7: Add Your API Key

Paste the FastRouter API key from Step 2 and save. You should see a **Successfully added** confirmation.

<img src="../.gitbook/assets/unknown (49).png" alt="" height="337" width="624">

#### Step 8: Enable FastRouter Models

To access FastRouter's full model catalog inside OpenCode:

1. Click the **model name** below the prompt box

<img src="../.gitbook/assets/unknown (50).png" alt="" height="193" width="624">

2.  Click **Manage Models**<br>

    <img src="../.gitbook/assets/Manage Models (4)" alt="Manage Models" height="311" width="624">
3.  Toggle on the FastRouter models you want to use<br>

    <img src="../.gitbook/assets/unknown (51).png" alt="" height="397" width="624">

&#x20; All enabled models are now available from the model picker. Requests route through FastRouter, and you can monitor usage in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

***

### Quick Start for OpenCode CLI

#### Step 1: Install OpenCode CLI

Download [OpenCode](https://opencode.ai/download) and install it on your system.

#### Step 2: Get Your FastRouter API Key

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

#### Step 3: Start OpenCode

```bash
cd /path/to/your/project
opencode
```

#### Step 4: Connect FastRouter

Inside OpenCode, run the `/connect` command and select **FastRouter** from the provider list:

```
/connect
```

<img src="../.gitbook/assets/unknown (52).png" alt="" height="87" width="624">

1.  Select **FastRouter**<br>

    <img src="../.gitbook/assets/unknown (54).png" alt="" height="245" width="624">


2.  Paste the API key from Step 2

    <img src="../.gitbook/assets/unknown (55).png" alt="" height="263" width="624">


3.  Pick a model and start prompting<br>

    <img src="../.gitbook/assets/unknown (56).png" alt="" height="573" width="624">

#### Step 4 (Alternative): Configure via `opencode.json`

Instead of running `/connect`, you can declare the FastRouter provider in your `opencode.json` config file:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "fastrouter": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "FastRouter",
      "options": {
        "baseURL": "https://api.fastrouter.ai/api/v1"
      },
      "models": {
        "anthropic/claude-sonnet-latest": {},
        "google/gemini-flash-latest": {}
      }
    }
  }
}
```

Set your API key via `/connect`, or by adding it to `~/.local/share/opencode/auth.json`:

```json
{
  "fastrouter": {
    "type": "api",
    "key": "sk-add-your-key-here"
  }
}
```

> **Note:** OpenCode stores credentials at `~/.local/share/opencode/auth.json` on Linux. macOS and Windows users should consult the OpenCode docs for their platform's auth path.

***

### Use OpenCode with 100+ Models

Switch between any of FastRouter's 100+ supported models from the OpenCode model picker (desktop) or by setting the `model` field / running `/model` (CLI). FastRouter uses the `provider/model-name` format:

```
openai/gpt-5.3-codex
```

#### Model Examples

Pick a different provider or model at any time:

```
# Anthropic Claude
anthropic/claude-sonnet-4.6

# Google Gemini
google/gemini-3-pro

# Mistral
mistralai/Mistral-Small-24B-Instruct-2501

# xAI Grok
x-ai/grok-4
```

No code changes or SDK swaps required. Select the model and OpenCode uses the new provider for the next request.

#### Automatic Model Selection

Let FastRouter pick the best model for each request based on query complexity, domain, and cost. Use this model identifier:

```
fastrouter/auto
```

FastRouter analyzes the input and routes to the most appropriate model from the available pool. This is the fastest way to get started without maintaining model preferences.

[Explore automatic provider selection](https://docs.fastrouter.ai/automatic-model-selection)

#### Cost-Optimized Routing with Sorting Slugs

Append a suffix to any model identifier to control provider selection:

```
# Route to the cheapest provider for this model
openai/gpt-5.3-codex:price

# Route to the fastest provider
openai/gpt-5.3-codex:throughput
```

[Configure cost and performance routing](https://docs.fastrouter.ai/provider-routing-strategies)

#### Flex Pricing

For batch-style coding tasks that tolerate higher latency, append `:flex` to any model to access up to 50% lower token costs:

```
openai/gpt-5.4-nano:flex
```

Flex routes your request to the provider's discounted inference tier. Same API key, same endpoint, same payload.

> **Note:** Flex is not recommended for interactive OpenCode sessions where you need low-latency responses. Use it for large refactoring jobs, codebase analysis, or documentation generation.

[Compare flex pricing options](https://docs.fastrouter.ai/explore-features/flex-pricing)

***

### FAQs

#### Configuration & Setup

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. The model is selected per request in OpenCode (model picker on desktop, `/model` or the `model` field in `opencode.json` on the CLI). You can switch models at any time without changing your key.

**Can I use OpenCode with my own provider keys (BYOK)?**

Yes. Set up an External Key integration in the FastRouter dashboard, then route OpenCode traffic through it. You retain your provider's pricing and rate limits while gaining FastRouter's routing and observability layer.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

#### Costs & Budgeting

**What happens when a key exceeds its budget?**

FastRouter blocks further requests until the budget resets (if a reset interval is configured) or an admin increases the limit. The developer receives an error response indicating the budget has been exceeded.

**How do I track OpenCode spending by team?**

Create separate projects for each team, issue project-scoped API keys, and use Dynamic Tags for finer-grained attribution. The Dashboard breaks down costs by project, key, model, and tag.

#### Performance & Reliability

**Does FastRouter add latency to OpenCode requests?**

FastRouter adds near-zero gateway overhead. For most workflows, this is negligible compared to model inference time.

#### Privacy & Security

**Is my code sent to FastRouter's servers?**

FastRouter acts as a pass-through gateway. Requests are routed to the model provider and responses are returned to your client. Content logging can be disabled per key for sensitive workloads. See the **Disable Content Logging** option in key settings.

***

### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
