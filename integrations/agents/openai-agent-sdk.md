---
description: >-
  Track usage, control costs, and add guardrails to your OpenAI Agents SDK
  applications
icon: openai
---

# OpenAI Agent SDK

### What is the OpenAI Agents SDK?

The [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) is OpenAI's official framework for building agentic AI applications in Python. It provides a lightweight set of primitives—Agents, Tools, Handoffs, and Guardrails—for building multi-step, tool-using agents with minimal boilerplate.

By routing the Agents SDK through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—your agents are no longer locked to OpenAI models
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting the Python Agents SDK to FastRouter, running a tool-calling agent, and model selection. It does not cover the TypeScript SDK.

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

**Step 2: Install the OpenAI Agents SDK**

{% code overflow="wrap" %}
```
pip install openai-agents
```
{% endcode %}

**Step 3: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai/)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

Export it in your terminal:

{% code overflow="wrap" %}
```
export FASTROUTER_API_KEY=sk-add-your-key-here
```
{% endcode %}

**Step 4: Point the SDK at FastRouter**

The Agents SDK accepts any OpenAI-compatible client. Create an `AsyncOpenAI` client with FastRouter's base URL and pass it to your agent's model. Save this as `agent_example.py`:

{% code overflow="wrap" %}
```
import osimport asynciofrom openai import AsyncOpenAIfrom agents import Agent, OpenAIChatCompletionsModel, Runner, function_tool, set_tracing_disabled
# Point the OpenAI client at FastRouterclient = AsyncOpenAI(    base_url="https://api.fastrouter.ai/api/v1",    api_key=os.environ["FASTROUTER_API_KEY"],)set_tracing_disabled(True)  # tracing uploads to OpenAI; disable when not using an OpenAI key

@function_tooldef get_weather(city: str) -> str:    """Get the current weather for a city."""    return f"The weather in {city} is 72°F and sunny."

agent = Agent(    name="Weather Assistant",    instructions="You are a helpful assistant. Use tools when needed.",    model=OpenAIChatCompletionsModel(model="openai/gpt-5.2", openai_client=client),    tools=[get_weather],)

async def main():    result = await Runner.run(agent, "What's the weather in San Francisco?")    print(result.final_output)

if __name__ == "__main__":    asyncio.run(main())
```
{% endcode %}

> **Note:** `set_tracing_disabled(True)` is recommended because the SDK's built-in tracing exports to OpenAI's platform and requires a native OpenAI key. All FastRouter requests remain fully visible in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

**Step 5: Run the Agent**

{% code overflow="wrap" %}
```
python agent_example.py
```
{% endcode %}

The agent calls the `get_weather` tool and responds:

\<img src="../.gitbook/assets/fastrouter-openai-agents-sdk-terminal.png" alt="OpenAI Agents SDK running a tool-calling agent through FastRouter" height="214" width="608">

Every request, token count, and cost now appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

***

#### Use the Agents SDK with 100+ Models

FastRouter uses the `provider/model-name` format. Switching your agent to a different provider is a one-line change:

{% code overflow="wrap" %}
```
# Anthropic Claudemodel=OpenAIChatCompletionsModel(model="anthropic/claude-4.5-sonnet", openai_client=client)
# DeepSeekmodel=OpenAIChatCompletionsModel(model="deepseek/deepseek-v4-pro", openai_client=client)
```
{% endcode %}

You can give each agent in a multi-agent workflow a different model—a fast, inexpensive model for triage agents and a frontier model for reasoning-heavy agents—all through the same client and key.

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

{% code overflow="wrap" %}
```
model=OpenAIChatCompletionsModel(model="fastrouter/auto", openai_client=client)
```
{% endcode %}

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

**Flex Pricing for Batch Agent Workloads**

Agent pipelines that run unattended—evaluation sweeps, data extraction, report generation—tolerate higher latency. Append `:flex` to any model to access up to 50% lower token costs:

{% code overflow="wrap" %}
```
model=OpenAIChatCompletionsModel(model="openai/gpt-5.2:flex", openai_client=client)
```
{% endcode %}

> **Note:** Flex is not recommended for interactive sessions where you need low-latency responses.

[Compare flex pricing options](https://docs.fastrouter.ai/explore-features/flex-pricing)

***

#### FAQs

**Configuration & Setup**

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. The model is selected per agent in code, and you can switch models at any time without changing your key.

**Why do I see tracing errors or `OPENAI_API_KEY` warnings?**

The SDK's built-in tracing exports to OpenAI's platform, which requires a native OpenAI key. Call `set_tracing_disabled(True)` after creating your client. Your request telemetry is still captured in the FastRouter Dashboard.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Tool calls crash with `ValidationError: ... arguments — Input should be a valid string`. What's wrong?**

Some model routes currently return tool-call `arguments` as a JSON object instead of the JSON-encoded string the OpenAI specification requires, and the Agents SDK validates this field strictly. If you hit this error, switch to a model verified for tool calling through FastRouter, such as `openai/gpt-5.2` or `anthropic/claude-4.5-sonnet`.

**Costs & Budgeting**

**What happens when a key exceeds its budget?**

FastRouter blocks further requests until the budget resets (if a reset interval is configured) or an admin increases the limit. The agent receives an error response indicating the budget has been exceeded.

**Performance & Reliability**

**Does FastRouter add latency to agent runs?**

FastRouter adds near-zero gateway overhead. For multi-step agent runs, this is negligible compared to model inference time.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
