---
description: Track usage, control costs, and add guardrails to your Cline coding agent
icon: square-code
---

# Cline

#### What is Cline?

[Cline](https://cline.bot/) is an AI coding assistant that integrates directly into your VS Code environment, providing autonomous coding capabilities—it can create and edit files, run commands, and complete multi-step tasks with your approval. It supports any OpenAI-compatible provider through its API settings.

By routing Cline through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—including coding models like [Grok Code Fast 1](https://fastrouter.ai/models/x-ai/grok-code-fast-1)
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers configuring the Cline VS Code extension to use FastRouter as an OpenAI-compatible provider.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* VS Code with the [Cline extension](https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev) installed

***

#### Quick Start

**Step 1: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

**Step 2: Configure the API Provider**

Open **Cline Settings** in VS Code and configure:

1. **API Provider:** select **OpenAI Compatible**
2. **Base URL:**

```
https://api.fastrouter.ai/api/v1
```

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

3. **API Key:** paste your FastRouter API key
4. **Model ID:** enter a FastRouter model slug, for example:

```
x-ai/grok-code-fast-1
```

**Step 3: Start a Task**

Give Cline a task. All requests now route through FastRouter, and every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

***

#### Use Cline with 100+ Models

FastRouter uses the `provider/model-name` format. Change the **Model ID** in Cline's settings to any catalog slug:

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

Let FastRouter pick the best model for each request based on query complexity, domain, and cost. Use this Model ID:

```
fastrouter/auto
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

***

#### FAQs

**Configuration & Setup**

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Change the Model ID in Cline's settings at any time—no key changes needed.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**My requests fail after setup. What should I check?**

Verify the Base URL is exactly `https://api.fastrouter.ai/api/v1`, the API key is correct, and the Model ID matches a FastRouter catalog slug exactly (including the provider prefix). Also make sure the Cline extension is updated to the latest version.

**Costs & Budgeting**

**Cline makes many requests per task. How do I keep costs under control?**

Set a budget and rate limit on the key you use with Cline. For team rollouts, create separate projects with project-scoped keys, and use Dynamic Tags for finer-grained attribution. The Dashboard breaks down costs by project, key, model, and tag.

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
