---
description: >-
  Track usage, control costs, and add guardrails to your CrewAI multi-agent
  crews
icon: users
---

# CrewAI

### What is CrewAI?

[CrewAI](https://www.crewai.com/) is a leading open-source framework for orchestrating role-playing, autonomous AI agents. You define agents with roles, goals, and backstories, assign them tasks, and CrewAI coordinates them as a "crew" to accomplish multi-step workflows.

Multi-agent crews generate a high volume of LLM calls, which makes cost visibility and control critical. By routing CrewAI through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—assign different providers to different agents with one key
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting CrewAI (1.x) to FastRouter, running a crew, and assigning different models to different agents.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* Python 3.10 or higher

***

#### Quick Start

**Step 1: Create a Project and Virtual Environment**

You'll only need to do this once. CrewAI requires Python 3.10–3.13:

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

**Step 2: Install CrewAI**

```bash
pip install crewai
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

**Step 4: Configure the CrewAI `LLM` to Use FastRouter**

Set `provider="openai"` so CrewAI uses its OpenAI-compatible client and passes the FastRouter model slug through verbatim. Save this as `crew_example.py`:

```python
import os
from crewai import Agent, Task, Crew, LLM

# provider="openai" makes CrewAI use the OpenAI-compatible protocol and
# pass the FastRouter model slug through verbatim
llm = LLM(
    model="openai/gpt-5.2",
    provider="openai",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
)

researcher = Agent(
    role="Tech Researcher",
    goal="Summarize technical topics clearly",
    backstory="You are an experienced technology analyst.",
    llm=llm,
    verbose=True,
)

task = Task(
    description="Write a two-sentence summary of what an LLM gateway does.",
    expected_output="A two-sentence summary.",
    agent=researcher,
)

crew = Crew(agents=[researcher], tasks=[task])
result = crew.kickoff()
print(result)
```

> **Note:** `provider="openai"` is required. Without it, CrewAI parses the model string itself and strips the provider prefix (sending `gpt-5.2` instead of `openai/gpt-5.2`), or—for non-OpenAI slugs—tries to use that provider's native SDK instead of FastRouter.

**Step 5: Run the Crew**

```bash
python crew_example.py
```

CrewAI prints the agent's progress (because `verbose=True`) followed by the final answer:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Every LLM call from every agent appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/) with token usage and cost.

***

#### Use CrewAI with 100+ Models

With `provider="openai"` set, any FastRouter slug works as-is—including non-OpenAI providers. A common pattern is to give cheap, fast models to high-volume worker agents and a frontier model to the planner, with no extra credentials:

```python
fast_llm = LLM(
    model="openai/gpt-4.1-nano",
    provider="openai",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
)

smart_llm = LLM(
    model="anthropic/claude-4.5-sonnet",
    provider="openai",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
)

worker = Agent(role="Researcher", goal="...", backstory="...", llm=fast_llm)
planner = Agent(role="Lead Strategist", goal="...", backstory="...", llm=smart_llm)
```

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

```python
llm = LLM(
    model="fastrouter/auto",
    provider="openai",
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
)
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

**Flex Pricing for Unattended Crews**

Crews that run unattended—research pipelines, content generation, scheduled jobs—tolerate higher latency. Append `:flex` to any model to access up to 50% lower token costs:

```python
model="openai/gpt-5.2:flex"
```

> **Note:** Flex is not recommended for crews where you are waiting on the output interactively.

[Compare flex pricing options](https://docs.fastrouter.ai/explore-features/flex-pricing)

***

#### FAQs

**Configuration & Setup**

**I get `ImportError: Unable to initialize LLM with model '...'` mentioning LiteLLM. Do I need LiteLLM?**

No. This error means the model string didn't match a provider CrewAI recognizes natively. Add `provider="openai"` to your `LLM(...)` call and pass the FastRouter slug in `model`—no LiteLLM install needed.

**I get `400 - There is no available model provider that meets your routing requirements`. What's wrong?**

The model slug reaching FastRouter is malformed. Ensure `provider="openai"` is set and `model` is an exact FastRouter catalog slug (for example, `openai/gpt-5.2`).

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Each agent can have its own `LLM` instance with a different model, all sharing one key.

**Costs & Budgeting**

**What happens when a key exceeds its budget?**

FastRouter blocks further requests until the budget resets (if a reset interval is configured) or an admin increases the limit. The crew receives an error response indicating the budget has been exceeded.

**How do I track spending per crew or per team?**

Create separate projects for each team, issue project-scoped API keys, and use Dynamic Tags for finer-grained attribution. The Dashboard breaks down costs by project, key, model, and tag.

**Performance & Reliability**

**Does FastRouter add latency to crew runs?**

FastRouter adds near-zero gateway overhead. For multi-step crew executions, this is negligible compared to model inference time.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
