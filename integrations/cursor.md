---
description: Track usage, control costs, and add guardrails to your Cursor AI editor
icon: square-code
---

# Cursor

#### What is Cursor?

[Cursor](https://cursor.com/) is a powerful AI-powered code editor built for pair-programming with AI. It provides chat, inline edits, and agentic coding workflows directly inside the editor, and supports custom OpenAI-compatible endpoints through its model settings.

By routing Cursor through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—including coding models like [Grok Code Fast 1](https://fastrouter.ai/models/x-ai/grok-code-fast-1)
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers configuring Cursor's OpenAI API settings to use FastRouter and adding models from the FastRouter catalog.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* Cursor installed ([download](https://cursor.com/))

***

#### Quick Start

**Step 1: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

**Step 2: Configure the OpenAI Base URL**

Open **Cursor Settings**, go to the **Models** section, and in the OpenAI API key settings:

1. Paste your FastRouter API key into the **OpenAI API Key** field
2. Enable **Override OpenAI Base URL** and set it to:

```
https://api.fastrouter.ai/api/v1 or https://go.fastrouter.ai/api/v1
```

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

**Step 3: Add a Model**

Click **Add Model** and enter a FastRouter model slug, for example:

```
x-ai/grok-code-fast-1
```

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

**Step 4: Start Coding**

Select the model you added from Cursor's model picker and start a chat or edit. Requests now route through FastRouter, and every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

***

#### Use Cursor with 100+ Models

FastRouter uses the `provider/model-name` format. Add any model from the catalog the same way—click **Add Model** and enter the slug:

```
# Anthropic Claude
anthropic/claude-4.5-sonnet

# OpenAI
openai/gpt-5.2

# xAI Grok
x-ai/grok-4
```

Check the [FastRouter Model Catalog](https://fastrouter.ai/models) for available models and input modalities (text, image, file).

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost. Add this model identifier:

```
fastrouter/auto
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

***

#### FAQs

**Configuration & Setup**

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Add as many models as you like in Cursor's settings and switch between them from the model picker—no key changes needed.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**My requests fail after setup. What should I check?**

Verify the Base URL is exactly `https://api.fastrouter.ai/api/v1`, the API key is correct, and the model name matches a FastRouter catalog slug exactly (including the provider prefix). Also make sure Cursor is updated to the latest version.

**Costs & Budgeting**

**What happens when a key exceeds its budget?**

FastRouter blocks further requests until the budget resets (if a reset interval is configured) or an admin increases the limit.

**Privacy & Security**

**Is my code sent to FastRouter's servers?**

FastRouter acts as a pass-through gateway. Requests are routed to the model provider and responses are returned to your client. Content logging can be disabled per key for sensitive workloads. See the **Disable Content Logging** option in key settings.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
