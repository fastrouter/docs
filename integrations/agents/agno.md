---
description: Track usage, control costs, and add guardrails to your Agno agents
icon: bolt
---

# Agno

### What is Agno?

[Agno](https://www.agno.com/) (formerly Phidata) is a high-performance framework for building multi-agent systems with memory, knowledge, and tools. It is known for extremely fast agent instantiation and a clean, minimal API surface.

By routing Agno through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint, via Agno's built-in `OpenAILike` model class
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting Agno (Python) to FastRouter, running an agent, and adding tools.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* Python 3.9 or higher

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

**Step 2: Install Agno**

```bash
pip install agno openai
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

**Step 4: Use the `OpenAILike` Model Class**

Agno ships an `OpenAILike` model class made for OpenAI-compatible gateways like FastRouter. Save this as `agno_example.py`:

```python
import os
from agno.agent import Agent
from agno.models.openai.like import OpenAILike

agent = Agent(
    model=OpenAILike(
        id="openai/gpt-5.2",
        base_url="https://api.fastrouter.ai/api/v1",
        api_key=os.environ["FASTROUTER_API_KEY"],
    ),
    markdown=True,
)

agent.print_response("Explain what an LLM gateway does in one sentence.")
```

**Step 5: Run the Agent**

```bash
python agno_example.py
```

Agno prints a formatted response panel in your terminal:

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

The request appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/) with token usage and cost.

***

#### Adding Tools

The FastRouter-backed model works with Agno tools. Pass any Python function with a docstring and type hints:

```python
import os
from agno.agent import Agent
from agno.models.openai.like import OpenAILike


def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"The weather in {city} is 72°F and sunny."


agent = Agent(
    model=OpenAILike(
        id="openai/gpt-5.2",
        base_url="https://api.fastrouter.ai/api/v1",
        api_key=os.environ["FASTROUTER_API_KEY"],
    ),
    tools=[get_weather],
)

result = agent.run("What is the weather in Paris?")
print(result.content)
# The weather in Paris is currently 72°F and sunny.
```

***

#### Use Agno with 100+ Models

FastRouter uses the `provider/model-name` format. Change the `id` to any FastRouter model slug:

```python
model=OpenAILike(
    id="anthropic/claude-4.5-sonnet",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
)
```

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

```python
model=OpenAILike(
    id="fastrouter/auto",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
)
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

**Flex Pricing for Background Agents**

Agents running scheduled or background jobs tolerate higher latency. Append `:flex` to any model to access up to 50% lower token costs:

```python
id="openai/gpt-5.2:flex"
```

> **Note:** Flex is not recommended for interactive agents where you need low-latency responses.

[Compare flex pricing options](https://docs.fastrouter.ai/explore-features/flex-pricing)

***

#### FAQs

**Configuration & Setup**

**I get `ModuleNotFoundError: No module named 'openai'`. What's missing?**

Agno's OpenAI-compatible models require the OpenAI SDK: `pip install openai`.

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Each `OpenAILike` instance selects its own model, so different agents can use different providers with one key.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Tool runs fail with an `Invalid type ... arguments` error from the API. What's wrong?**

Some model routes currently return tool-call `arguments` as a JSON object instead of the JSON-encoded string the OpenAI specification requires, which breaks the tool-call round trip. Plain chat responses are unaffected. If you hit this error, switch to a model verified for tool calling through FastRouter, such as `openai/gpt-5.2` or `anthropic/claude-4.5-sonnet`.

**Costs & Budgeting**

**What happens when a key exceeds its budget?**

FastRouter blocks further requests until the budget resets (if a reset interval is configured) or an admin increases the limit. The agent receives an error response indicating the budget has been exceeded.

**Performance & Reliability**

**Does FastRouter add latency to agent runs?**

FastRouter adds near-zero gateway overhead. For most workloads, this is negligible compared to model inference time.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
