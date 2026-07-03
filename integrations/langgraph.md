---
description: Track usage, control costs, and add guardrails to your LangGraph agents
icon: brackets-curly
---

# Langgraph

### What is LangGraph?

[LangGraph](https://www.langchain.com/langgraph) is a framework for building stateful, multi-step agents as graphs. Built by the LangChain team, it adds durable state, cycles, branching, and human-in-the-loop control on top of LangChain's model interfaces.

By routing LangGraph through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting LangGraph (Python) to FastRouter using a `ChatOpenAI` model inside a prebuilt ReAct agent.

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

**Step 2: Install LangGraph**

```bash
pip install langchain langgraph langchain-openai
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

**Step 4: Point the Model at FastRouter**

LangGraph uses LangChain chat models, so the standard `ChatOpenAI` class works once you override the base URL. Save this as `graph_example.py`:

```python
import os
from langchain_openai import ChatOpenAI
from langchain.agents import create_agent

llm = ChatOpenAI(
    base_url="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
    model="openai/gpt-5.2",
)


def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"The weather in {city} is 72°F and sunny."


agent = create_agent(llm, tools=[get_weather])

result = agent.invoke({"messages": [{"role": "user", "content": "What's the weather in San Francisco?"}]})
print(result["messages"][-1].content)
```

**Step 5: Run the Agent**

```bash
python graph_example.py
```

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

The agent calls the `get_weather` tool and responds with the weather. Every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

***

#### Use LangGraph with 100+ Models

FastRouter uses the `provider/model-name` format. Switch providers by changing the model slug—no new dependencies or credentials:

```python
# Anthropic Claude
model="anthropic/claude-4.5-sonnet"

# Google Gemini
model="google/gemini-3.1-pro-preview"
```

You can give each node in your graph a different model—a fast model for routing nodes and a frontier model for reasoning nodes.

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

Yes. The API key controls access and budget. Each node can use its own `ChatOpenAI` instance with a different model, all sharing one key.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Costs & Budgeting**

**Graphs with cycles can make many model calls. How do I control cost?**

Set a budget and rate limit on the key, and use Dynamic Tags to attribute spend per graph or per run. The Dashboard breaks down costs by project, key, model, and tag.

**Performance & Reliability**

**Does FastRouter add latency to graph execution?**

FastRouter adds near-zero gateway overhead, negligible compared to model inference time.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
