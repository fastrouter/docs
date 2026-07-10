---
description: >-
  Track usage, control costs, and add guardrails to your Xcode AI coding
  sessions.
icon: swift
---

# XCode

### What is Xcode?

[Xcode](https://developer.apple.com/xcode/) is Apple's IDE for building apps across Apple platforms. Its **Apple Intelligence** coding assistant (Xcode 26 and later) lets you add custom model providers under **Settings → Intelligence**, so you can chat with, and generate code from, models beyond the built-in options. Any provider that speaks an OpenAI-compatible API can be added—including FastRouter.

By routing Xcode through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—including coding models like [Grok Code Fast 1](https://fastrouter.ai/models/x-ai/grok-code-fast-1)
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers adding FastRouter as a model provider in Xcode's Intelligence settings.

#### Prerequisites

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* Xcode 26 or later on a Mac with **Apple Intelligence** enabled

***

### Quick Start

#### Step 1: Enable Apple Intelligence

Go to **macOS Settings → Apple Intelligence & Siri** and make sure **Apple Intelligence** is turned on. This is required before Xcode can use any AI model providers.

<figure><img src="../../.gitbook/assets/Screenshot 2026-07-03 at 7.03.12 PM.png" alt=""><figcaption></figcaption></figure>

#### Step 2: Get Your FastRouter API Key

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

#### Step 3: Add FastRouter as a Model Provider

1. Open **Xcode → Settings → Intelligence**.

<figure><img src="../../.gitbook/assets/Screenshot 2026-07-03 at 7.17.42 PM.png" alt=""><figcaption></figcaption></figure>

2. Click **Add a Model Provider** at the bottom of the page.

<figure><img src="../../.gitbook/assets/Screenshot 2026-07-03 at 7.18.07 PM.png" alt=""><figcaption></figcaption></figure>

3. In the dialog, enter the following details:
   * **URL:** `https://api.fastrouter.ai/api/`
   * **API Key Header:** `Authorization`
   * **API Key:** `Bearer sk-add-your-key-here` (include the literal `Bearer` prefix before your FastRouter key)
   * **Description:** `FastRouter` (or any name you prefer)

<figure><img src="../../.gitbook/assets/Screenshot 2026-07-03 at 7.30.24 PM.png" alt=""><figcaption></figcaption></figure>

5. Click Save to save the configuration.

> **Important:** Do not add `/v1` to the end of the endpoint. For Xcode provider setup, use `https://api.fastrouter.ai/api/`, not the direct API endpoint `https://api.fastrouter.ai/api/v1`.

> **Note:** Enter the API key with the `Bearer` prefix because the **API Key Header** is `Authorization`—Xcode sends the field's full value as the header, so it must be `Bearer <your-key>`.

<figure><img src="../../.gitbook/assets/Screenshot 2026-07-03 at 7.47.11 PM.png" alt=""><figcaption></figcaption></figure>

#### Step 4: Browse and Select Models

Once configured, select **FastRouter** in the provider list to see available models. If Xcode does not populate the list automatically, or you want a specific model, enter the FastRouter slug manually in `provider/model-name` format for example `x-ai/grok-code-fast-1`. Use **Favorites** to pin the models you use most; pinned models appear at the top for quick access.

<figure><img src="../../.gitbook/assets/Screenshot 2026-07-03 at 7.48.49 PM.png" alt=""><figcaption></figcaption></figure>

#### Step 5: Start Using AI in Xcode

Open the chat interface and start working with your selected model. All requests route through FastRouter, and every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption><p>XCode using Fable 5</p></figcaption></figure>

***

### Use Xcode with 100+ Models

FastRouter uses the `provider/model-name` format. Switch models from the chat model dropdown, or add FastRouter slugs to your Favorites:

```
# Anthropic Claude
anthropic/claude-4.5-sonnet

# OpenAI
openai/gpt-5.2
```

**Recommended models for Xcode:**

* `x-ai/grok-code-fast-1` — fast and inexpensive; a good default for routine edits
* `anthropic/claude-4.5-sonnet` — highest quality for complex, multi-file changes
* `openai/gpt-5.2` — strong all-round alternative

[Explore the full model catalog](https://fastrouter.ai/models)

#### Automatic Model Selection

Let FastRouter pick the best model for each request based on query complexity, domain, and cost by entering this slug as the model:

```
fastrouter/auto
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

***

### FAQs

#### Configuration & Setup

**No models appear after I add FastRouter. What should I check?**

Confirm the **URL** is exactly `https://api.fastrouter.ai/api/`, the **API Key Header** is `Authorization`, and the **API Key** field includes the `Bearer` prefix before your key. Do not add `/v1` to the Xcode provider URL. If the model list still doesn't populate, enter a known slug manually (for example `x-ai/grok-code-fast-1`) and verify your key has access to it.

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Switch models from the chat dropdown or add more Favorites—no key changes needed.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

#### Costs & Budgeting

**How do I keep spend predictable?**

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
