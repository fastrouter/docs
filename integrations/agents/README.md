---
description: >-
  Route any Python agent framework through FastRouter. One key, 100+ models,  
  and full cost visibility across every agent call.
icon: user-hat-tie
---

# Agents

Agent workloads fan out into many LLM calls per run, so cost and reliability matter more here than anywhere else. Every framework below accepts an OpenAI-compatible client, which is all FastRouter needs. Point it at FastRouter and each agent call routes through one endpoint with:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, and Mistral behind one key. Assign a cheap model to triage agents and a frontier model to reasoning-heavy ones without juggling provider credentials.
* **Observability** on every request: cost, tokens, latency, and model per call, visible in your [dashboard](https://dashboard.fastrouter.ai/).
* **Reliability** through automatic failover across providers, response caching, and intelligent routing.
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation.

The pattern is the same everywhere: create an OpenAI-compatible client with FastRouter's base URL, pass it a model slug in `provider/model-name` format.

<table data-view="cards"><thead><tr><th></th><th data-type="content-ref"></th></tr></thead><tbody><tr><td></td><td><a href="openai-agent-sdk.md">openai-agent-sdk.md</a></td></tr><tr><td></td><td><a href="openclaw.md">openclaw.md</a></td></tr><tr><td></td><td><a href="running-hermes-agent-with-fastrouter.md">running-hermes-agent-with-fastrouter.md</a></td></tr><tr><td></td><td><a href="langgraph.md">langgraph.md</a></td></tr><tr><td></td><td><a href="autogen.md">autogen.md</a></td></tr></tbody></table>
