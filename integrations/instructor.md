---
description: >-
  Track usage, control costs, and add guardrails to your Instructor structured
  outputs
icon: square-code
---

# Instructor

### What is Instructor?

[Instructor](https://python.useinstructor.com/) is a library for getting structured, validated outputs from LLMs. It patches the OpenAI client so you can request a Pydantic model as the response type and get back a validated object, with automatic retries on validation failure.

By routing Instructor through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—compare which model produces the most reliable structured outputs
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting Instructor (Python) to FastRouter by patching an OpenAI client pointed at FastRouter.

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

**Step 2: Install Instructor**

```bash
pip install instructor
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

**Step 4: Patch an OpenAI Client Pointed at FastRouter**

Create a standard OpenAI client with FastRouter's base URL, then patch it with Instructor. Save this as `instructor_example.py`:

```python
import os
import instructor
from openai import OpenAI
from pydantic import BaseModel

client = instructor.from_openai(
    OpenAI(
        base_url="https://api.fastrouter.ai/api/v1",
        api_key=os.environ["FASTROUTER_API_KEY"],
    )
)


class CityInfo(BaseModel):
    city: str
    country: str
    population: int


result = client.chat.completions.create(
    model="openai/gpt-5.2",
    response_model=CityInfo,
    messages=[{"role": "user", "content": "Tell me about Tokyo."}],
)
print(result)
```

**Step 5: Run the Script**

```bash
python instructor_example.py
```

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

You get back a validated `CityInfo` object. The request appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/) with token usage and cost.

***

#### Use Instructor with 100+ Models

FastRouter uses the `provider/model-name` format. Switch providers by changing the `model` argument—useful for finding which model produces the most reliable structured outputs:

```python
result = client.chat.completions.create(
    model="anthropic/claude-4.5-sonnet",
    response_model=CityInfo,
    messages=[{"role": "user", "content": "Tell me about Tokyo."}],
)
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

Yes. The API key controls access and budget. Pass a different `model` on each call, all sharing one key.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Structured Outputs**

**My calls hit Instructor's retry limit. How do I fix it?**

Validation reliability varies by model. Try a more capable model, simplify the schema, or add field descriptions. Because FastRouter gives you every provider through one key, comparing models on your schema takes minutes.

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
