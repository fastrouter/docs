---
description: Track usage, control costs, and add guardrails to your Open WebUI deployment
icon: window-maximize
---

# Open WebUI

### What is Open WebUI?

[Open WebUI](https://openwebui.com/) is a self-hosted, ChatGPT-style web interface for chatting with large language models. It runs entirely on your own infrastructure and connects to any server or provider that implements the OpenAI-compatible API.

By routing Open WebUI through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—your whole team picks from a shared catalog with one key
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting Open WebUI to FastRouter by adding it as an OpenAI-compatible connection.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* A running Open WebUI instance with admin access ([installation guide](https://docs.openwebui.com/getting-started/))

***

#### Quick Start

**Step 1: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

**Step 2: Add FastRouter as a Connection**

1. Open Open WebUI in your browser.
2. Go to ⚙️ **Admin Settings** → **Connections** → **OpenAI**.

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-27 at 2.24.08 PM.png" alt=""><figcaption></figcaption></figure>

3. Click ➕ **Add Connection**.

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-27 at 2.26.48 PM.png" alt=""><figcaption></figcaption></figure>

3. Fill in:
   * **URL:** `https://api.fastrouter.ai/api/v1`
   * **API Key:** your FastRouter API key
4. Click **Save**.

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-27 at 3.15.23 PM.png" alt=""><figcaption></figcaption></figure>

**Step 3: Add the Models You Want**

FastRouter exposes a large catalog, so rather than loading every model, add the specific slugs your team needs to the **Model IDs (Filter)** allowlist:

1. In the connection settings, find **Model IDs (Filter)**.
2. Type a FastRouter model slug—for example `openai/gpt-5.2`—and click the **+** icon.
3. Repeat for any other models (e.g., `anthropic/claude-4.5-sonnet`, `x-ai/grok-code-fast-1`).
4. Click **Save**.

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-27 at 3.15.23 PM.png" alt=""><figcaption></figcaption></figure>

> **Note:** Open WebUI verifies a connection by calling the provider's `/models` endpoint. Even if verification is slow or returns a warning, chat completions still work—the **Model IDs (Filter)** allowlist guarantees the models you listed appear in the selector.

**Step 4: Start Chatting**

Select a FastRouter model from the model dropdown in a new chat and send a message. The request routes through FastRouter, and every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

<figure><img src="../../.gitbook/assets/Screenshot 2026-06-27 at 3.17.47 PM.png" alt=""><figcaption></figcaption></figure>

***

#### Use Open WebUI with 100+ Models

FastRouter uses the `provider/model-name` format. Add any catalog slug to the connection's **Model IDs (Filter)** and it becomes selectable in the chat model dropdown:

```
anthropic/claude-4.5-sonnet
google/gemini-3.1-pro-preview
x-ai/grok-code-fast-1
```

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost. Add this slug to your **Model IDs (Filter)** and select it like any other model:

```
fastrouter/auto
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

***

#### FAQs

**Configuration & Setup**

**Connection verification shows an error, but is the connection broken?**

Not necessarily. Verification calls the `/models` endpoint; if it is slow or returns a non-200, you'll see a warning, but chat completions still work. Make sure your models are listed in the **Model IDs (Filter)** allowlist, and they will appear in the selector regardless.

**Do I need a trailing slash on the URL?**

No. Use exactly `https://api.fastrouter.ai/api/v1` with no trailing slash.

**Can I run multiple connections at once?**

Yes. Each connection has a toggle to enable or disable it without deleting it, so you can keep FastRouter alongside other providers and switch as needed.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Costs & Budgeting**

**How do I control spend across a team using one shared instance?**

Set a budget and rate limit on the FastRouter key, and use Dynamic Tags to attribute spend. The Dashboard breaks down costs by project, key, model, and tag.

**Privacy & Security**

**Is my chat content sent to FastRouter's servers?**

FastRouter acts as a pass-through gateway. Requests are routed to the model provider and responses are returned to your instance. Content logging can be disabled per key for sensitive workloads. See the **Disable Content Logging** option in key settings. For multi-user deployments, prefer a least-privilege key rather than an admin/master key.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
