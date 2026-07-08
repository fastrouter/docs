---
description: Track usage, control costs, and add guardrails to your Pydantic AI agents
icon: shield-check
---

# Pydantic AI

#### What is Pydantic AI?

[Pydantic AI](https://ai.pydantic.dev/) is an agent framework from the team behind Pydantic, designed to bring the "FastAPI feeling" to LLM development. Its signature strength is type-safe, validated structured outputs: you define a Pydantic model, and the agent guarantees the response matches it.

By routing Pydantic AI through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—run the same type-safe agent against any provider and compare structured-output reliability
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting Pydantic AI (Python) to FastRouter and running an agent with validated structured output.

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

**Step 2: Install Pydantic AI**

```bash
pip install "pydantic-ai-slim[openai]"
```

(Or install the full package with `pip install pydantic-ai`.)

**Step 3: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

Export it in your terminal:

```bash
export FASTROUTER_API_KEY=sk-add-your-key-here
```

**Step 4: Point the OpenAI Provider at FastRouter**

Pydantic AI's `OpenAIProvider` accepts a custom base URL, which is all FastRouter needs. Save this as `pydantic_ai_example.py`:

```python
import os
from pydantic import BaseModel
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIChatModel
from pydantic_ai.providers.openai import OpenAIProvider

model = OpenAIChatModel(
    "openai/gpt-5.2",
    provider=OpenAIProvider(
        base_url="https://api.fastrouter.ai/api/v1",
        api_key=os.environ["FASTROUTER_API_KEY"],
    ),
)


class CityInfo(BaseModel):
    city: str
    country: str
    population: int


agent = Agent(model, output_type=CityInfo)

result = agent.run_sync("Tell me about Tokyo.")
print(result.output)
```

**Step 5: Run the Agent**

```bash
python pydantic_ai_example.py
```

The agent returns a validated, typed `CityInfo` object:

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

The request appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/) with full token and cost details.

***

#### Use Pydantic AI with 100+ Models

FastRouter uses the `provider/model-name` format. Switching providers is a one-line change to the model slug—useful for testing which model produces the most reliable structured outputs:

```python
model = OpenAIChatModel(
    "anthropic/claude-4.5-sonnet",
    provider=OpenAIProvider(
        base_url="https://api.fastrouter.ai/api/v1",
        api_key=os.environ["FASTROUTER_API_KEY"],
    ),
)
```

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

```python
model = OpenAIChatModel(
    "fastrouter/auto",
    provider=OpenAIProvider(
        base_url="https://api.fastrouter.ai/api/v1",
        api_key=os.environ["FASTROUTER_API_KEY"],
    ),
)
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

**Flex Pricing for Batch Extraction**

Structured-output agents often run in batch—extracting records from documents, normalizing datasets, generating evaluations. Append `:flex` to any model to access up to 50% lower token costs on latency-tolerant jobs:

```python
"openai/gpt-5.2:flex"
```

> **Note:** Flex is not recommended for interactive agents where you need low-latency responses.

[Compare flex pricing options](https://docs.fastrouter.ai/explore-features/flex-pricing)

***

#### FAQs

**Configuration & Setup**

**I get an `ImportError` for `OpenAIChatModel` or `OpenAIProvider`. What's missing?**

Install the OpenAI extra: `pip install "pydantic-ai-slim[openai]"`.

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Each `OpenAIChatModel` instance selects its own model, and you can run several side by side.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Structured Outputs**

**My agent raises validation errors on structured output. How do I fix it?**

Try a more capable model—structured-output reliability varies by model. You can also simplify the output schema or add field descriptions to guide the model. Because FastRouter gives you every provider through one key, comparing models on your schema takes minutes.

**My agent crashes with `ValidationError: object — Input should be 'chat.completion'`. What's wrong?**

Some model routes currently return responses that deviate from the OpenAI chat-completions specification, and Pydantic AI validates responses strictly. If you hit this error, switch to a model verified to work with Pydantic AI through FastRouter, such as `openai/gpt-5.2` or `anthropic/claude-4.5-sonnet`.

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
