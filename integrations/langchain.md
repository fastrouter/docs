---
description: Track usage, control costs, and add guardrails to your LangChain applications
icon: link-horizontal
---

# LangChain

### What is LangChain?

[LangChain](https://www.langchain.com/) is the most widely adopted framework for developing applications powered by large language models. It provides composable building blocks for chains, agents, retrieval (RAG), and tool calling.

By routing LangChain through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint, no separate provider SDKs or keys inside your LangChain code
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting LangChain (Python) to FastRouter via `ChatOpenAI`, plus streaming, tool calling, and model selection. It does not cover LangChain.js.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai/))
* Python 3.10 or higher

***

#### Quick Start

**Step 1: Create a Project and Virtual Environment**

You'll only need to do this once:

{% code overflow="wrap" %}
```
mkdir my_projectcd my_projectpython -m venv .venv
```
{% endcode %}

Activate the virtual environment. Do this every time you start a new terminal session.

On macOS or Linux:

{% code overflow="wrap" %}
```
source .venv/bin/activate
```
{% endcode %}

On Windows:

{% code overflow="wrap" %}
```
.venv\Scripts\activate
```
{% endcode %}

**Step 2: Install LangChain**

{% code overflow="wrap" %}
```
pip install langchain langchain-openai
```
{% endcode %}

**Step 3: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai/)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

Export it in your terminal:

{% code overflow="wrap" %}
```mdc
export FASTROUTER_API_KEY=sk-add-your-key-here
```
{% endcode %}

**Step 4: Point `ChatOpenAI` at FastRouter**

FastRouter is fully OpenAI-compatible, so the standard `ChatOpenAI` class works out of the box—just override the base URL. Save this as `langchain_example.py`:

{% code overflow="wrap" %}
```python
import osfrom langchain_openai import ChatOpenAI
llm = ChatOpenAI(    base_url="https://api.fastrouter.ai/api/v1",    api_key=os.environ["FASTROUTER_API_KEY"],    model="openai/gpt-5.2",)
response = llm.invoke("Explain what an LLM gateway does in one sentence.")print(response.content)
```
{% endcode %}

**Step 5: Run the Script**

{% code overflow="wrap" %}
```python
python langchain_example.py
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1).png" alt="The request appears immediately in your FastRouter Dashboard, with token usage and cost attribution."><figcaption><p>The request appears immediately in your <a href="https://dashboard.fastrouter.ai/">FastRouter Dashboard</a>, with token usage and cost attribution.</p></figcaption></figure>

***

#### Streaming and Tool Calling

Because `ChatOpenAI` is LangChain's standard chat model interface, the FastRouter-backed `llm` object drops into any LangChain feature. Both of these run through FastRouter unchanged:

{% code overflow="wrap" %}
```python
# Streamingfor chunk in llm.stream("Count to 3, digits only"):    print(chunk.content, end="", flush=True)
# Tool callingfrom langchain_core.tools import tool
@tooldef get_weather(city: str) -> str:    """Get the current weather for a city."""    return f"The weather in {city} is 72°F and sunny."
response = llm.bind_tools([get_weather]).invoke("What is the weather in Paris?")print(response.tool_calls)# [{'name': 'get_weather', 'args': {'city': 'Paris'}, ...}]
```
{% endcode %}

***

#### Use LangChain with 100+ Models

FastRouter uses the `provider/model-name` format. Switch providers by changing the model slug—no new dependencies or credentials:

{% code overflow="wrap" %}
```md
claude = ChatOpenAI(base_url="https://api.fastrouter.ai/api/v1",    api_key=os.environ["FASTROUTER_API_KEY"],    model="anthropic/claude-4.5-sonnet",)
gemini = ChatOpenAI(    base_url="https://api.fastrouter.ai/api/v1",    api_key=os.environ["FASTROUTER_API_KEY"],    model="google/gemini-3.1-pro-preview",)
```
{% endcode %}

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

{% code overflow="wrap" %}
```md
llm = ChatOpenAI(base_url="https://api.fastrouter.ai/api/v1",    api_key=os.environ["FASTROUTER_API_KEY"], model="fastrouter/auto",)
```
{% endcode %}

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

**Flex Pricing for Batch Workloads**

For RAG indexing, evaluation runs, or other batch chains that tolerate higher latency, append `:flex` to any model to access up to 50% lower token costs:

{% code overflow="wrap" %}
```
model="openai/gpt-5.2:flex"
```
{% endcode %}

> **Note:** Flex is not recommended for interactive chains where you need low-latency responses.

[Compare flex pricing options](https://docs.fastrouter.ai/explore-features/flex-pricing)

***

#### FAQs

**Configuration & Setup**

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. The model is selected per `ChatOpenAI` instance, and you can run several models side by side in one application.

**What if `OPENAI_API_KEY` is already set in my environment?**

The explicit `api_key=` argument takes precedence. Pass your FastRouter key directly to avoid accidentally routing to OpenAI.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**`bind_tools` raises `ValidationError: invalid_tool_calls.0.args — Input should be a valid string`. What's wrong?**

Some model routes currently return tool-call `arguments` as a JSON object instead of the JSON-encoded string the OpenAI specification requires. Plain `invoke` and `stream` calls are unaffected, but tool calling fails. If you hit this error, switch to a model verified for tool calling through FastRouter, such as `openai/gpt-5.2` or `anthropic/claude-4.5-sonnet`.

**Costs & Budgeting**

**How do I track LangChain spending by team or app?**

Create separate projects for each team, issue project-scoped API keys, and use Dynamic Tags for finer-grained attribution. The Dashboard breaks down costs by project, key, model, and tag.

**Performance & Reliability**

**Does FastRouter add latency to LangChain calls?**

FastRouter adds near-zero gateway overhead. For most chains, this is negligible compared to model inference time.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
