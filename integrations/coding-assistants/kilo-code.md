---
description: >-
  Track usage, control costs, and add guardrails to your Kilo Code coding
  sessions
icon: file-code
---

# Kilo Code

### What is Kilo Code?

[Kilo Code](https://kilocode.ai/) is an open-source AI coding agent for VS Code (with a CLI as well) that plans, writes, and fixes code across your project. It supports custom model providers, including any endpoint compatible with the OpenAI API standard—so it works with FastRouter out of the box.

By routing Kilo Code through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—including coding models like [Grok Code Fast 1](https://fastrouter.ai/models/x-ai/grok-code-fast-1)
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers adding FastRouter to Kilo Code as a custom OpenAI-compatible provider.

#### Prerequisites

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* VS Code with the [Kilo Code extension](https://kilocode.ai/) installed

***

### Quick Start

#### Step 1: Get Your FastRouter API Key

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

#### Step 2: Add FastRouter as a Custom Provider

1. In Kilo Code, open **Settings** (gear icon) and go to the **Providers** tab.

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-25 at 12.44.40 PM.png" alt="" width="375"><figcaption></figcaption></figure>



2. Scroll to the bottom and click **Custom provider**.



3. Fill in the custom provider dialog:
4. **Provider ID:** `fastrouter`

* **Display name:** `FastRouter`
* **Provider API:** **OpenAI Compatible**
* **Base URL:** `https://api.fastrouter.ai/api/v1`
* **API key:** your FastRouter API key

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-26 at 4.35.11 PM.png" alt=""><figcaption></figcaption></figure>

#### Step 3: Select Your Models

Once the **Base URL** and **API key** are entered, Kilo Code queries FastRouter's models endpoint and presents a searchable model picker with the full catalog. Search with fuzzy matching (typing `grok` finds `x-ai/grok-code-fast-1`), select the models you want, and click **Submit**. The provider's models then appear in Kilo Code's model picker.

> **Note:** If auto-detection doesn't populate, enter FastRouter slugs manually in `provider/model-name` format—for example `x-ai/grok-code-fast-1`.

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-26 at 4.37.12 PM.png" alt=""><figcaption></figcaption></figure>

#### Step 4: Start Coding

Pick a FastRouter model from the model picker and give Kilo Code a task. All requests route through FastRouter, and every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

***

### Use Kilo Code with 100+ Models

FastRouter uses the `provider/model-name` format. Add as many catalog models to the provider as you like and switch between them from the model picker:

```
x-ai/grok-code-fast-1
anthropic/claude-4.5-sonnet
openai/gpt-5.2
```

**Recommended models for Kilo Code:**

* `x-ai/grok-code-fast-1` — fast and inexpensive; a good default for routine edits
* `anthropic/claude-4.5-sonnet` — highest quality for complex, multi-file changes
* `openai/gpt-5.2` — strong all-round alternative

[Explore the full model catalog](https://fastrouter.ai/models)

#### Automatic Model Selection

Let FastRouter pick the best model for each request based on query complexity, domain, and cost by adding this slug to the provider's model list:

```
fastrouter/auto
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

***

### FAQs

#### Configuration & Setup

**The model picker doesn't auto-populate. What should I check?**

Confirm the **Base URL** is exactly `https://api.fastrouter.ai/api/v1` and the API key is correct—auto-detection queries FastRouter's models endpoint using both. If it still doesn't populate, add model IDs manually using full FastRouter slugs (including the provider prefix).

**I get "Model Not Found." What's wrong?**

The model ID doesn't match a FastRouter catalog slug. Use the full `provider/model-name` form—for example `x-ai/grok-code-fast-1`, not `grok-code-fast-1`.

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Add multiple models to the provider and switch from the model picker—no key changes needed.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

#### Costs & Budgeting

**Kilo Code can send large repo context. How do I control cost?**

Set a budget and rate limit on the key. The Dashboard breaks down costs by project, key, model, and tag.

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
