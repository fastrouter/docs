---
description: >-
  Track usage, control costs, and add guardrails to your AutoGen multi-agent
  apps
icon: message-captions
---

# AutoGen

### What is AutoGen?

[AutoGen](https://microsoft.github.io/autogen/) is a framework from Microsoft for building multi-agent AI applications, where multiple conversational agents collaborate to solve tasks. It supports any OpenAI-compatible model through its client configuration.

By routing AutoGen through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—assign different models to different agents with one key
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting AutoGen (Python) to FastRouter using the OpenAI chat completion client.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* Python 3.10 or higher

***

#### Quick Start

**Step 1: Create a Project and Virtual Environment**

You'll only need to do this once:

```bash
mkdir my_project
cd my_project
python -m venv .venv
```

Activate the virtual environment. Do this every time you start a new terminal session.

On macOS or Linux:

```bash
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scripts\activate
```

**Step 2: Install AutoGen**

```bash
pip install -U "autogen-agentchat" "autogen-ext[openai]"
```

**Step 3: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

Export it in your terminal:

```bash
export FASTROUTER_API_KEY=sk-add-your-key-here
```

**Step 4: Configure the Model Client for FastRouter**

AutoGen's `OpenAIChatCompletionClient` accepts a custom base URL. Save this as `autogen_example.py`:

```python
import os
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="openai/gpt-5.2",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
    model_info={
        "vision": False,
        "function_calling": True,
        "json_output": True,
        "family": "unknown",
        "structured_output": True,
    },
)
import os
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="openai/gpt-5.2",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
    model_info={
        "vision": False,
        "function_calling": True,
        "json_output": True,
        "family": "unknown",
        "structured_output": True,
    },
)

agent = AssistantAgent("assistant", model_client=model_client)


async def main():
    result = await agent.run(task="Explain what an LLM gateway does in one sentence.")
    print(result.messages[-1].content)


if __name__ == "__main__":
    asyncio.run(main())
agent = AssistantAgent("assistant", model_client=model_client)


async def main():
    result = await agent.run(task="Explain what an LLM gateway does in one sentence.")
    print(result.messages[-1].content)


if __name__ == "__main__":
    asyncio.run(main())
```

> **Note:** For non-OpenAI model slugs, AutoGen requires the `model_info` block shown above so it knows the model's capabilities.

**Step 5: Run the Agent**

```bash
python autogen_example.py
```

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

The agent responds, and the request appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/) with token usage and cost.

***

#### Use AutoGen with 100+ Models

FastRouter uses the `provider/model-name` format. Switch providers by changing the model slug. Give each agent in a multi-agent team its own client—a fast model for worker agents and a frontier model for the orchestrator:

```python
model="anthropic/claude-4.5-sonnet"
```

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

```python
model="fastrouter/auto"
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

***

#### FAQs

**Configuration & Setup**

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Each agent can have its own model client, all sharing one key.

**Why do I need the `model_info` block?**

AutoGen looks up model capabilities by name. Because FastRouter slugs aren't in AutoGen's built-in table, you declare capabilities explicitly with `model_info`.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Costs & Budgeting**

**Multi-agent conversations generate many calls. How do I control cost?**

Set budgets and rate limits on the key, and use Dynamic Tags to attribute spend per team or run. The Dashboard breaks down costs by project, key, model, and tag.

**Performance & Reliability**

**Does FastRouter add latency?**

FastRouter adds near-zero gateway overhead, negligible compared to model inference time.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
