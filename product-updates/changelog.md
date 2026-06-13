---
description: New updates and improvements
icon: gem
---

# Changelog

{% updates format="full" %}
{% update date="2026-06-11" %}
## Added

Prompt Library — Write, store, version, and optimize prompts in one place and reference them by ID in API calls, so prompt changes ship without a code deploy. Mark any version as **Production** to serve it to all live requests, and roll back instantly by promoting an earlier version. Optimize — Refine a stored prompt with Prompt Optimizations and save the result as a tracked, optimized version, with **Compare** to diff versions before promoting. Variables — Insert `{{curly braces}}` placeholders in a prompt and fill them per request via the `variables` field.

[https://docs.fastrouter.ai/prompt-library](https://docs.fastrouter.ai/prompt-library)<br>
{% endupdate %}

{% update date="2026-05-28" %}
## Added

Free Models (`:free`) — Append `:free` to a supported model ID (e.g. `sarvam/sarvam-105b:free`) to route requests at no cost, with the suffix stripped transparently before reaching the provider. Available to all orgs regardless of billing status. Per-model daily quota — 10 requests per org per day, tracked independently per model and reset daily at UTC midnight; paid orgs consume free quota rather than billing credits.

[https://docs.fastrouter.ai/explore-features/free-models-free](https://docs.fastrouter.ai/explore-features/free-models-free)
{% endupdate %}

{% update date="2026-05-21" %}
## Added

**Support for non-Claude models via Anthropic Messages format** — Route Claude Code requests to OpenAI, DeepSeek, and other FastRouter-supported providers using the same Anthropic-compatible interface\
**Universal model access in Claude Code** — Launch Claude Code with any FastRouter-supported model using the `--model` flag, without changing tooling or workflows

[https://docs.fastrouter.ai/integrations/claude-code](https://docs.fastrouter.ai/integrations/claude-code)<br>
{% endupdate %}

{% update date="2026-05-14" %}
## Added

**Bring Your Own Keys (BYOK) for external providers** — Attach your own API credentials from supported LLM providers directly to FastRouter while preserving your negotiated pricing\
**Custom model provisioning** — Register fine-tuned or privately hosted models with custom endpoints, pricing metadata, and API compatibility mappings\
**Advanced endpoint configuration** — Override provider base URLs with OpenAI-, Anthropic-, or Gemini-compatible formats, plus support for custom authentication headers\
**Granular model enablement** — Enable or disable individual catalog models per integration and map custom models to provider-specific endpoints\
**Integrated routing visibility** — Reference integrations via Provider Slug across Virtual Models, Gateway Configs, and Activity Logs for full routing traceability\
[https://docs.fastrouter.ai/add-external-keys-byok](https://docs.fastrouter.ai/add-external-keys-byok)<br>
{% endupdate %}

{% update date="2026-05-07" %}
## Added

**Video Evaluations for AI-generated content** — Automatically assess video outputs at scale using LLM-based judges, with scoring across motion fidelity, audio-visual sync, cinematic quality, and prompt adherence

**Seamless log-based dataset creation** — Import video generation logs directly from FastRouter activity with filtering, sampling, and zero manual uploads

**Unified evaluation infrastructure** — Use the same Custom Evaluations setup as text and image evals, including shared judge configuration, scoring rubrics, and dashboards

**Multimodal LLM judging** — Leverage capable video-aware models to evaluate outputs with structured reasoning across multiple quality dimensions

**Deep-dive result analysis** — Access per-video judge reasoning, aggregated performance metrics, and cost/latency insights in a single view

[https://docs.fastrouter.ai/video-evaluations](https://docs.fastrouter.ai/video-evaluations)<br>
{% endupdate %}

{% update date="2026-04-30" %}
## Added

**Flex Pricing for Vertex AI and Google AI Studio models** — Access supported models at up to **50% lower cost** by using provider Flex inference tiers, ideal for batch jobs, background workloads, and latency-tolerant applications

**Zero code-change activation** — Append `:flex` to any supported model ID (for example `google/gemini-3.1-pro-preview:flex`) while keeping the same API key, endpoint, and request payload

**Provider-native Flex routing** — FastRouter automatically routes requests to the provider’s discounted Flex tier, with support for provider pinning to ensure correct execution paths

**Built for async and cost-sensitive workloads** — Recommended for summarisation pipelines, data extraction, classification, eval runs, scheduled jobs, and large-scale preprocessing tasks where response speed is less critical

**Model Catalog Flex visibility** — View supported Flex-enabled models and pricing directly in the model catalog, with per-model availability across providers

[https://docs.fastrouter.ai/flex-pricing](https://docs.fastrouter.ai/flex-pricing)
{% endupdate %}

{% update date="2026-04-23" %}
## Added

**Flex Pricing for OpenAI models** — Access supported models at up to **50% lower cost** by using provider Flex inference tiers, ideal for batch jobs, background workloads, and latency-tolerant applications

**Zero code-change activation** — Append `:flex` to any supported model ID (for example `openai/gpt-5.4-nano:flex`) while keeping the same API key, endpoint, and request payload

**Provider-native Flex routing** — FastRouter automatically routes requests to the provider’s discounted Flex tier, with support for provider pinning to ensure correct execution paths

**Built for async and cost-sensitive workloads** — Recommended for summarisation pipelines, data extraction, classification, eval runs, scheduled jobs, and large-scale preprocessing tasks where response speed is less critical

**Model Catalog Flex visibility** — View supported Flex-enabled models and pricing directly in the model catalog, with per-model availability across providers

[https://docs.fastrouter.ai/flex-pricing](https://docs.fastrouter.ai/flex-pricing)
{% endupdate %}

{% update date="2026-04-13" %}
## Added

**Added**

* **Prompt Optimizations (GEPA-powered)** — Automatically improve system prompts using FastRouter’s Genetic-Pareto optimization engine with iterative reflection, mutation, and scoring
* **Run prompt experiments from your own data** — Import datasets from files or Activity Logs, evaluate against custom metrics, and compare optimized prompts against baseline performance
* **LLM-as-a-Judge evaluations** — Score prompts across metrics like Accuracy, Helpfulness, Safety, Completeness, or your own custom criteria using a shared evaluator model
* **Optimization Insights** — Review improvement %, final scores, accepted iterations, and the full optimized prompt in a dedicated results view

[https://docs.fastrouter.ai/prompt-optimizations](https://docs.fastrouter.ai/prompt-optimizations)
{% endupdate %}

{% update date="2026-04-03" %}
## Added

**Added**

* **MCP Gateway** — Register any MCP-compatible server (GitHub, Linear, Gmail, or your own APIs) and expose its tools to any model routed through FastRouter, with centralized credential management, project-level scoping, and selective tool exposure
* **OAuth 2.0 & Static Header authentication for MCP servers** — Securely store and inject credentials server-side across all tool calls, with support for No Auth, Static Header, and full OAuth 2.0 authorization code flow
* **Auto-execution mode** — Set `auto_execute_tools: true` to let FastRouter handle the complete tool-call loop and return a final text response directly, with a configurable `max_tool_rounds` cap (maximum 5)

[https://docs.fastrouter.ai/mcp-gateway](https://docs.fastrouter.ai/mcp-gateway)
{% endupdate %}
{% endupdates %}

{% updates format="full" %}
{% update date="2026-03-27" %}
## Added

**Added**

* **Priority Routing** — Route requests through models in a fixed priority order, with automatic sequential fallback for deterministic, predictable routing
* **Category-Based Routing** — Direct requests to different model groups based on detected prompt category, with per-category sub-strategies and a configurable default fallback

[https://docs.fastrouter.ai/explore-features/virtual-model-aliases](https://docs.fastrouter.ai/explore-features/virtual-model-aliases)
{% endupdate %}

{% update date="2026-03-20" %}
## Added

* **Tracing (W3C `traceparent` support)** — Group multiple LLM API calls into a single trace with ordered spans
* **Traces view in Activity** — Visualize execution timelines, latency, tokens, and cost across spans

[https://docs.fastrouter.ai/tracing](https://docs.fastrouter.ai/tracing)
{% endupdate %}
{% endupdates %}
