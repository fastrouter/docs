---
description: Track usage, control costs, and add guardrails to your LlamaIndex applications
icon: layer-group
---

# LlamaIndex

#### What is LlamaIndex?

[LlamaIndex](https://www.llamaindex.ai/) is a leading data framework for building LLM applications over your own data. It provides ingestion, indexing, retrieval, and query primitives for retrieval-augmented generation (RAG) and agentic workflows.

By routing LlamaIndex through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers connecting LlamaIndex (Python) to FastRouter using the `OpenAILike` LLM class.

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

**Step 2: Install LlamaIndex**

```bash
pip install llama-index-llms-openai-like
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

**Step 4: Point `OpenAILike` at FastRouter**

LlamaIndex's `OpenAILike` class targets any OpenAI-compatible endpoint. Save this as `llamaindex_example.py`:

```python
import os
from llama_index.llms.openai_like import OpenAILike

llm = OpenAILike(
    model="openai/gpt-5.2",
    api_base="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
    is_chat_model=True,
)

response = llm.complete("Explain what an LLM gateway does in one sentence.")
print(response)
```

> **Note:** Set `is_chat_model=True` so LlamaIndex uses the chat completions endpoint.

**Step 5: Run the Script**

```bash
python llamaindex_example.py
```

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

The response prints to your terminal, and the request appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/) with token usage and cost.

***

#### Use LlamaIndex with 100+ Models

FastRouter uses the `provider/model-name` format. Switch providers by changing the model slug:

```python
llm = OpenAILike(
    model="anthropic/claude-4.5-sonnet",
    api_base="https://api.fastrouter.ai/api/v1",
    api_key=os.environ["FASTROUTER_API_KEY"],
    is_chat_model=True,
)
```

The same `llm` object plugs into LlamaIndex query engines, chat engines, and agents—set it as the default with `Settings.llm = llm`.

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

Yes. The API key controls access and budget. Create multiple `OpenAILike` instances with different models, all sharing one key.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Do I also need a separate embeddings model?**

RAG pipelines need an embeddings model in addition to the chat model. FastRouter supports embeddings through the same endpoint—see the [Embeddings API reference](https://docs.fastrouter.ai/api-reference/embeddings).

**Costs & Budgeting**

**How do I track RAG spending?**

Set budgets and rate limits on the key, and use Dynamic Tags to attribute spend per pipeline. The Dashboard breaks down costs by project, key, model, and tag.

**Performance & Reliability**

**Does FastRouter add latency to queries?**

FastRouter adds near-zero gateway overhead, negligible compared to model inference and retrieval time.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)
