---
description: Welcome to FastRouter.AI!
---

# Welcome

## Introduction to FastRouter

FastRouter.ai is an LLM gateway and LLMOps platform. It gives you one endpoint to reach 160+ models across 20+ providers — and a control plane that routes, governs, observes, and optimizes every call you send through it.

You change a base URL. FastRouter handles model selection, provider failover, credential management, caching, cost attribution, guardrails, and evaluations — with zero markup on token costs, whether you use your own provider keys (BYOK) or FastRouter credits.

## Start in 60 seconds

FastRouter is a drop-in replacement for the OpenAI SDK. Change two lines:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.fastrouter.ai/api/v1",
    api_key="YOUR_FASTROUTER_API_KEY",
)

response = client.chat.completions.create(
    model="openai/gpt-5.6-sol",
    messages=[{"role": "user", "content": "Hello from FastRouter"}],
)
```

Prefer a different format? FastRouter also speaks the [Anthropic Messages API](https://docs.fastrouter.ai/api-reference/anthropic-messages-format) and the [Gemini Interactions API](https://docs.fastrouter.ai/api-reference/gemini-interactions-api-format) natively — use whichever endpoint your stack already speaks.

Not sure which model to use? Send `fastrouter/auto` as the model ID and let FastRouter decide per request.

## What is an LLM Gateway?

An LLM gateway sits between your application and your model providers. It handles request routing, failover, observability, cost tracking, and credential management — so your code stops encoding provider differences and starts treating models as interchangeable infrastructure.

## Beyond The Gateway: Routing Intelligence

Most gateways stop at access. FastRouter continues past it.

Your production traffic is the benchmark — not a generic leaderboard. FastRouter analyzes how you actually use models, then tells you where to cache, compress, downgrade, or re-prompt, with projected savings and the evidence behind each recommendation. Optimization is a product surface here, not a consulting engagement.

[Watch more details about FastRouter](https://youtu.be/1Wb_DW2CHa8)

***

### Route

Reach every model through one endpoint, and control exactly how each request is served.

| Capability                                                                        | What it does                                                                                                                          |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| [Automatic Model Selection](explore-features/automatic-model-selection.md)        | Use `fastrouter/auto` and FastRouter picks the best-fit model per request, weighing complexity, domain, and cost.                     |
| [Provider Routing Strategies](explore-features/provider-routing-strategies.md)    | Choose who serves each model — lowest price, lowest latency, highest throughput, weighted shuffle, priority order, or AI Auto Router. |
| [Fallback Models](explore-features/fallback-models.md)                            | Automatically retry the next model or provider on rate limits, downtime, or errors. Configurable per key.                             |
| [Virtual Model Aliases](explore-features/virtual-model-aliases.md)                | Map one stable alias to many models and providers. Swap the model underneath without touching application code.                       |
| [FastRouter Blend](explore-features/fastrouter-blend-multi-model-deliberation.md) | Send one prompt to a panel of models in parallel, then have a judge model compare their answers into a structured analysis.           |
| [Free Model Router](explore-features/free-model-router.md)                        | Append `:free` to eligible models, or use `fastrouter/free`, for no-cost testing and low-stakes workloads.                            |

### Optimize

The routing intelligence layer — turning real usage into lower cost and better output.

| Capability                                                       | What it does                                                                                                                                     |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Insights](explore-features/insights.md)                         | Weekly, evidence-backed cost recommendations generated from your own traffic, each with projected savings and exactly where to apply the change. |
| [Custom Evaluations](explore-features/custom-evaluations.md)     | Benchmark and compare models on your own data with LLM-as-a-judge scoring and custom criteria.                                                   |
| [Video Evaluations](explore-features/video-evaluations.md)       | Score AI-generated video at scale across motion, sync, quality, and prompt adherence.                                                            |
| [Prompt Optimizations](explore-features/prompt-optimizations.md) | Evolve stronger system prompts automatically with GEPA reflective optimization, using LLM-judged feedback as gradients.                          |
| [Prompt Library](explore-features/prompt-library.md)             | Version, diff, deploy, and roll back prompts independently of your code — no redeploy required.                                                  |

### Save

Cut spend without changing what your application asks for.

| Capability                                                             | What it does                                                                                                                                    |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| [Response Caching](explore-features/response-caching.md)               | Cache full responses across providers, including generated images and video. Repeat requests return instantly at no additional generation cost. |
| [Prompt Caching](explore-features/response-caching.md)                 | Reuse repeated context — system prompts, RAG chunks, documents — at a fraction of the input price, with sticky routing to maximize hit rates.   |
| [Prompt Compression](explore-features/prompt-compression.md)           | Shrink prompts at the gateway to cut input tokens without degrading response quality. Opt-in per request; fails open.                           |
| [Flex Pricing](explore-features/flex-pricing.md)                       | Append `:flex` on eligible OpenAI and Gemini models for significantly lower cost on latency-tolerant work.                                      |
| [Batch Processing](explore-features/batch-processing.md)               | Submit high-volume, non-real-time workloads asynchronously at a steep discount.                                                                 |
| [Add External Keys (BYOK)](explore-features/add-external-keys-byok.md) | Route through your own provider accounts, with FastRouter's routing, observability, and governance still fully applied.                         |

### Govern

Hard limits, enforced at the gateway — before the call, not after the invoice.

| Capability                                                             | What it does                                                                                                                           |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| [Organization & Members](explore-features/organization-and-members.md) | Role-based access control across your organization.                                                                                    |
| [Projects](explore-features/projects.md)                               | Separate workspaces per team, product, or environment, with scoped keys, budgets, and cost attribution.                                |
| [Keys & Settings](explore-features/keys-and-settings.md)               | Granular per-key control over models, providers, budgets, and features.                                                                |
| [Provisioning Keys](explore-features/provisioning-keys.md)             | Programmatically create, rotate, scope, and revoke service account keys — built for platforms provisioning on behalf of end users.     |
| [Guardrails](explore-features/guardrails.md)                           | Deterministic and LLM-based checks for PII, toxicity, topic adherence, and regex patterns. Observe violations, or block them outright. |
| [Credits](explore-features/credits.md)                                 | Manage balance, budgets, and spend across the organization.                                                                            |

### Observe

Every call attributed, every anomaly flagged.

| Capability                                                   | What it does                                                                                                      |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- |
| [Dashboard & Activity Log](explore-features/quickstart.md)   | Inspect every request with full payloads, latency, tokens, and cost — plus one-click evals straight from the log. |
| [Tracing](explore-features/tracing.md)                       | Group related calls into a single trace with a `traceparent` header, including tool-call spans.                   |
| [Custom Alerts](explore-features/alerts.md)                  | Warning and Critical thresholds on latency, error rate, usage, and spend, routed to Slack, email, or webhook.     |
| [System Alerts](explore-features/system-alerts.md)           | Automatic notifications when credit balance or project and key budgets cross critical thresholds.                 |
| [Dynamic Tags](explore-features/dynamic-tags-per-request.md) | Tag requests per call with custom metadata, then slice usage and cost by feature, customer, environment, or team. |

### Build

Everything the modern stack expects, unified across providers.

| Capability                                                                                                                     | What it does                                                                                                                                                  |
| ------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Structured Outputs](explore-features/structured-outputs.md)                                                                   | JSON schema-constrained responses across all major providers.                                                                                                 |
| [Function Calling](explore-features/function-calling.md)                                                                       | Unified tool use, whichever provider serves the request.                                                                                                      |
| [Reasoning Tokens](explore-features/reasoning-tokens.md)                                                                       | Control and inspect thinking tokens on supported models.                                                                                                      |
| [Image](https://claude.ai/api-reference/image) & [Video](api-reference/video.md)                                               | Generation and editing across providers through one API.                                                                                                      |
| [Audio](api-reference/audio/) & [Embeddings](api-reference/embeddings.md)                                                      | Transcription, text-to-speech, and embeddings on the same endpoint.                                                                                           |
| [Image Processing](https://claude.ai/explore-features/image-processing) & [PDF Processing](explore-features/pdf-processing.md) | Send images and PDFs directly as input, handled natively at the gateway.                                                                                      |
| [Realtime API](https://claude.ai/explore-features/realtime-api-websocket)                                                      | Low-latency speech-to-speech and text streaming over WebSocket.                                                                                               |
| [MCP Gateway](https://claude.ai/explore-features/mcp-gateway)                                                                  | Register MCP tool servers once and expose their tools to any model. Credentials are vaulted and injected at the gateway — agents never see raw provider keys. |
| [Web Search](https://claude.ai/explore-features/web-search)                                                                    | Attach web search as a tool on any model request, with results injected into context automatically.                                                           |

### Integrate

Route the tools your team already uses through FastRouter for cost control, observability, and governance.

* [**Coding Assistants**](https://claude.ai/integrations/coding-assistants) — Claude Code, Cursor, Codex CLI, Cline, Aider, Roo Code, OpenCode, Kilo Code, Xcode
* [**Agents**](https://claude.ai/integrations/agents) — LangGraph, CrewAI, AutoGen, Agno, OpenAI Agents SDK, Pydantic AI, OpenClaw, Hermes
* [**Apps & Frameworks**](https://claude.ai/integrations/app) — LangChain, LlamaIndex, Instructor, n8n, Open WebUI, Scalekit

Building with an AI coding assistant? Point it at [skill.md](https://claude.ai/skill) to give it working knowledge of FastRouter's features.

***

### Jump Right In

<table data-view="cards"><thead><tr><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Explore Features</strong></td><td>Learn how FastRouter helps you scale and optimize your LLM usage</td><td></td><td></td><td><a href="explore-features/quickstart.md">quickstart.md</a></td></tr><tr><td><strong>API Reference</strong></td><td>Dive directly into request formats, supported models, error responses, and more</td><td></td><td></td><td><a href="/broken/pages/7aUFmnCMx9m4smGncsXL">Broken link</a></td></tr><tr><td><strong>Integrations</strong></td><td>Guides to integrate FastRouter with your favorite tools</td><td></td><td></td><td><a href="https://app.gitbook.com/s/zZfZz8wlCHOmP1FU2BsK/integrations">Integrations</a></td></tr></tbody></table>

Questions or feedback? Reach us at [contact@fastrouter.ai](mailto:contact@fastrouter.ai).

[Watch more details about FastRouter](https://youtu.be/1Wb_DW2CHa8)
