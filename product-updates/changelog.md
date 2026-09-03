---
description: New updates and improvements
icon: gem
---

# Changelog

{% updates format="full" %}
{% update date="2026-09-02" %}
## Added Image Evaluations for AI-generated content

**Image Evaluations** — Assess image generation outputs at scale using multimodal LLM judges, scoring across prompt adherence, visual quality, composition, and reference-image fidelity — the same Custom Evaluations workflow you already use for text and video.

**Log-based dataset creation** — Import image generation requests directly from your Activity Log with filtering and sampling, no manual uploads.

[https://docs.fastrouter.ai/explore-features/image-evaluations](https://docs.fastrouter.ai/explore-features/image-evaluations)
{% endupdate %}

{% update date="2026-08-26" %}
## Added Insights

**Insights** — Weekly, evidence-backed recommendations that cut LLM spend without giving up output quality. Each recommendation is scoped to a real project, key, and model, comes with a projected weekly saving, and shows the full calculation and raw request evidence behind the number. Nothing is applied automatically — you review and decide.

**Three recommendation types** — _Caching_ detects repeated system-prompt prefixes and models the saving from prompt caching; _Flex Tier_ reprices high-volume standard-tier traffic at provider Flex rates (same model, same output); _Model Switch_ replays sampled requests against cheaper alternatives and scores both with an LLM judge, linking out to the full evaluation run.

**Ranked, filterable view** — Summary bar with projected total and per-category savings, sortable cards, and a detail panel covering scope, methodology, warnings, evidence, and how to apply. Model Switch is opt-in under Settings since it consumes billable inference.

[https://docs.fastrouter.ai/explore-features/insights](https://docs.fastrouter.ai/explore-features/insights)
{% endupdate %}

{% update date="2026-08-19" %}
## Added Prompt Compression

**Prompt Compression** — Opt-in, per-request compression that shrinks messages before they reach the provider, cutting input tokens without changing the response. Add a single `optimize.compress` block to any Chat Completions or Anthropic Messages request; everything else stays as-is.

**Three engines** — `headroom` (lossless structural compression for system prompts, rules, and tool schemas), `caveman` (rule-based prose compaction that preserves negations, numbers, code, URLs, and quoted strings), and `llmlingua` (ML token pruning for RAG chunks, docs, and transcripts). Combine them with `"engine": "all"` for maximum savings.

**Audit mode & fail-open** — Set `"mode": "audit"` to measure what you would save without changing a byte. If compression can't run, your original messages are sent unchanged. Savings are reported via `X-FastRouter-Compression-*` response headers.

**Compression stats in the Dashboard & Activity Log** — See original vs compressed prompts and tokens saved per request in the Activity Log flyout, plus a Dashboard tab tracking compression outcomes across your traffic.

[https://docs.fastrouter.ai/explore-features/prompt-compression](https://docs.fastrouter.ai/explore-features/prompt-compression)
{% endupdate %}

{% update date="2026-08-12" %}
## Explicit prompt caching for OpenAI GPT-5.6+

Control exactly where the cached prefix ends with `prompt_cache_breakpoint` and `prompt_cache_options` on both Chat Completions and Responses APIs, alongside FastRouter's automatic sticky routing that keeps follow-up requests on the provider holding a warm cache.

[https://docs.fastrouter.ai/explore-features/prompt-caching](https://docs.fastrouter.ai/explore-features/prompt-caching)
{% endupdate %}

{% update date="2026-08-05" %}
## Added Free Model Router

Use `fastrouter/free` as the model ID to have FastRouter automatically select a free model. The **FastRouter Free Model Router** automatically selects a free model from the currently available pool for each request. This lets you use a single model ID without maintaining a list of individual free models as availability changes.
{% endupdate %}

{% update date="2026-07-29" %}
## Added Artifact Builder in the Model Playground

**Artifact Builder In Text Playground** — Build interactive **webpages and apps directly in FastRouter’s Text Playground** with the new Artifact Builder. Turn your prompts into functional, editable artifacts and iterate on them in real time—all within the playground.
{% endupdate %}

{% update date="2026-07-22" %}
## Added MCP Server Templates

**MCP Server Templates** — Added pre-configured templates for popular MCP servers, eliminating the need to manually enter server configuration values. Connect common tools in just a few clicks while retaining the flexibility to customize settings when needed.

[https://docs.fastrouter.ai/mcp-gateway](https://docs.fastrouter.ai/mcp-gateway)
{% endupdate %}

{% update date="2026-07-15" %}
## Added Cache Analytics in the Dashboard

**Cache Analytics** — A dedicated caching view in the Dashboard showing cached vs uncached prompt tokens, cache-write vs cache-read volume, hit rate, and the resulting cost savings over time, filterable by key, model, and provider.

**Per-request cache breakdown** — The Activity Log flyout now surfaces cache-read and cache-write tokens for every request, including Anthropic cache usage, so you can confirm caching is working on a specific key or conversation.

[https://docs.fastrouter.ai/explore-features/prompt-caching](https://docs.fastrouter.ai/explore-features/prompt-caching)
{% endupdate %}

{% update date="2026-07-08" %}
## Added FastRouter Blend: Multi-Model Deliberation

**FastRouter Blend** — Ask a panel of models the same prompt in parallel, then have a judge model compare their answers into a structured analysis: agreements, disagreements, coverage gaps, standout insights, missing considerations, and confidence notes. The judge evaluates rather than merges, so you see exactly where models agree, conflict, and what each uniquely contributed.

**Two ways to use it** — Set `"model": "fastrouter/blend"` to always run Blend and get a readable summary as the completion, or attach the `fastrouter:blend` server tool to any normal request and let the outer model decide when to call it.

**Configurable panel & judge** — Supply up to 5 `analysis_models` or let the router auto-select the top 3 candidates; override the judge with `model`. Sub-calls reuse your existing routing, BYOK, and credits, with aggregated panel + judge usage and cost returned on every response.

[https://docs.fastrouter.ai/explore-features/fastrouter-blend-multi-model-deliberation](https://docs.fastrouter.ai/explore-features/fastrouter-blend-multi-model-deliberation)
{% endupdate %}

{% update date="2026-07-01" %}
## Added Image & Video Model Playgrounds

**Model Playground: Image & Video** — Added dedicated Image and Video Playgrounds to experiment with multimodal models directly from the browser. Generate images and videos, compare model outputs, and iterate without writing any code.
{% endupdate %}

{% update date="2026-06-25" %}
## Improved Prompt Library&#x20;

**Prompt Library** — Optimize any saved prompt version directly from Prompt Library. Prompt Optimizations now creates a new version automatically, preserving the original while making it easy to review and promote improvements.

**Prompt Comparison with Samples** — Compare the original and optimized prompt side-by-side using sample inputs before promoting a new version. Quickly validate improvements and understand how prompt changes affect model outputs.

[https://docs.fastrouter.ai/prompt-library](https://docs.fastrouter.ai/prompt-library)
{% endupdate %}

{% update date="2026-06-18" %}
## Added New Video & Multimodal Models

**BytePlus Provider** — Added support for BytePlus-hosted models. FastRouter automatically handles BytePlus' custom pricing flow, including providers that return pricing information asynchronously, ensuring accurate cost tracking and billing.

**New Video & Multimodal Models** — Added support for the latest image, video, and reasoning models, including x-ai/grok-imagine-video, GLM 5.2, Kimi Code 2.7, and Minimax M3.

[https://fastrouter.ai/models?order=newest](https://fastrouter.ai/models?order=newest)<br>
{% endupdate %}

{% update date="2026-06-11" %}
## Added MCP Server Templates

**MCP Server Templates** — Added pre-configured templates for popular MCP servers, eliminating the need to manually enter server configuration values. Connect common tools in just a few clicks while retaining the flexibility to customize settings when needed.

[https://docs.fastrouter.ai/mcp-gateway](https://docs.fastrouter.ai/mcp-gateway)
{% endupdate %}

{% update date="2026-06-04" %}
## Added Prompt Library

**Prompt Library** — Write, store, version, and optimize prompts in one place and reference them by ID in API calls, so prompt changes ship without a code deploy. Mark any version as **Production** to serve it to all live requests, and roll back instantly by promoting an earlier version. Optimize — Refine a stored prompt with Prompt Optimizations and save the result as a tracked, optimized version, with **Compare** to diff versions before promoting. Variables — Insert `{{curly braces}}` placeholders in a prompt and fill them per request via the `variables` field.

[https://docs.fastrouter.ai/prompt-library](https://docs.fastrouter.ai/prompt-library)<br>
{% endupdate %}

{% update date="2026-05-28" %}
## Added Free Models

**Free Models** (`:free`) — Append `:free` to a supported model ID (e.g. `sarvam/sarvam-105b:free`) to route requests at no cost, with the suffix stripped transparently before reaching the provider. Available to all orgs regardless of billing status. Per-model daily quota — 10 requests per org per day, tracked independently per model and reset daily at UTC midnight; paid orgs consume free quota rather than billing credits.

[https://docs.fastrouter.ai/explore-features/free-models-free](https://docs.fastrouter.ai/explore-features/free-models-free)
{% endupdate %}

{% update date="2026-05-21" %}
## Added Support for non-Claude models via Anthropic Messages format

**Support for non-Claude models via Anthropic Messages format** — Route Claude Code requests to OpenAI, DeepSeek, and other FastRouter-supported providers using the same Anthropic-compatible interface\
**Universal model access in Claude Code** — Launch Claude Code with any FastRouter-supported model using the `--model` flag, without changing tooling or workflows

[https://docs.fastrouter.ai/integrations/claude-code](https://docs.fastrouter.ai/integrations/claude-code)<br>
{% endupdate %}

{% update date="2026-05-14" %}
## Added Bring Your Own Keys (BYOK) for external providers

**Bring Your Own Keys (BYOK) for external providers** — Attach your own API credentials from supported LLM providers directly to FastRouter while preserving your negotiated pricing\
**Custom model provisioning** — Register fine-tuned or privately hosted models with custom endpoints, pricing metadata, and API compatibility mappings\
**Advanced endpoint configuration** — Override provider base URLs with OpenAI-, Anthropic-, or Gemini-compatible formats, plus support for custom authentication headers\
**Granular model enablement** — Enable or disable individual catalog models per integration and map custom models to provider-specific endpoints\
**Integrated routing visibility** — Reference integrations via Provider Slug across Virtual Models, Gateway Configs, and Activity Logs for full routing traceability\
[https://docs.fastrouter.ai/add-external-keys-byok](https://docs.fastrouter.ai/add-external-keys-byok)<br>
{% endupdate %}

{% update date="2026-05-07" %}
## Added Video Evaluations for AI-generated content&#x20;

**Video Evaluations for AI-generated content** — Automatically assess video outputs at scale using LLM-based judges, with scoring across motion fidelity, audio-visual sync, cinematic quality, and prompt adherence

**Seamless log-based dataset creation** — Import video generation logs directly from FastRouter activity with filtering, sampling, and zero manual uploads

**Unified evaluation infrastructure** — Use the same Custom Evaluations setup as text and image evals, including shared judge configuration, scoring rubrics, and dashboards

**Multimodal LLM judging** — Leverage capable video-aware models to evaluate outputs with structured reasoning across multiple quality dimensions

**Deep-dive result analysis** — Access per-video judge reasoning, aggregated performance metrics, and cost/latency insights in a single view

[https://docs.fastrouter.ai/video-evaluations](https://docs.fastrouter.ai/video-evaluations)<br>
{% endupdate %}

{% update date="2026-04-30" %}
## Added Flex Pricing for Vertex AI and Google AI Studio models&#x20;

**Flex Pricing for Vertex AI and Google AI Studio models** — Access supported models at up to **50% lower cost** by using provider Flex inference tiers, ideal for batch jobs, background workloads, and latency-tolerant applications

**Zero code-change activation** — Append `:flex` to any supported model ID (for example `google/gemini-3.1-pro-preview:flex`) while keeping the same API key, endpoint, and request payload

**Provider-native Flex routing** — FastRouter automatically routes requests to the provider’s discounted Flex tier, with support for provider pinning to ensure correct execution paths

**Built for async and cost-sensitive workloads** — Recommended for summarisation pipelines, data extraction, classification, eval runs, scheduled jobs, and large-scale preprocessing tasks where response speed is less critical

**Model Catalog Flex visibility** — View supported Flex-enabled models and pricing directly in the model catalog, with per-model availability across providers

[https://docs.fastrouter.ai/flex-pricing](https://docs.fastrouter.ai/flex-pricing)
{% endupdate %}

{% update date="2026-04-23" %}
## Added Flex Pricing for OpenAI models

**Flex Pricing for OpenAI models** — Access supported models at up to **50% lower cost** by using provider Flex inference tiers, ideal for batch jobs, background workloads, and latency-tolerant applications

**Zero code-change activation** — Append `:flex` to any supported model ID (for example `openai/gpt-5.4-nano:flex`) while keeping the same API key, endpoint, and request payload

**Provider-native Flex routing** — FastRouter automatically routes requests to the provider’s discounted Flex tier, with support for provider pinning to ensure correct execution paths

**Built for async and cost-sensitive workloads** — Recommended for summarisation pipelines, data extraction, classification, eval runs, scheduled jobs, and large-scale preprocessing tasks where response speed is less critical

**Model Catalog Flex visibility** — View supported Flex-enabled models and pricing directly in the model catalog, with per-model availability across providers

[https://docs.fastrouter.ai/flex-pricing](https://docs.fastrouter.ai/flex-pricing)
{% endupdate %}

{% update date="2026-04-13" %}
## Added Prompt Optimizations

**Prompt Optimizations (GEPA-powered)** — Automatically improve system prompts using FastRouter’s Genetic-Pareto optimization engine with iterative reflection, mutation, and scoring

**Run prompt experiments from your own data** — Import datasets from files or Activity Logs, evaluate against custom metrics, and compare optimized prompts against baseline performance

**LLM-as-a-Judge evaluations** — Score prompts across metrics like Accuracy, Helpfulness, Safety, Completeness, or your own custom criteria using a shared evaluator model

**Optimization Insights** — Review improvement %, final scores, accepted iterations, and the full optimized prompt in a dedicated results view

[https://docs.fastrouter.ai/prompt-optimizations](https://docs.fastrouter.ai/prompt-optimizations)
{% endupdate %}

{% update date="2026-04-03" %}
## Added MCP Gateway

**MCP Gateway** — Register any MCP-compatible server (GitHub, Linear, Gmail, or your own APIs) and expose its tools to any model routed through FastRouter, with centralized credential management, project-level scoping, and selective tool exposure

**OAuth 2.0 & Static Header authentication for MCP servers** — Securely store and inject credentials server-side across all tool calls, with support for No Auth, Static Header, and full OAuth 2.0 authorization code flow

**Auto-execution mode** — Set `auto_execute_tools: true` to let FastRouter handle the complete tool-call loop and return a final text response directly, with a configurable `max_tool_rounds` cap (maximum 5)

[https://docs.fastrouter.ai/mcp-gateway](https://docs.fastrouter.ai/mcp-gateway)
{% endupdate %}
{% endupdates %}

{% updates format="full" %}
{% update date="2026-03-27" %}
## Added Priority & Category Based Routing Strategies

**Priority Routing** — Route requests through models in a fixed priority order, with automatic sequential fallback for deterministic, predictable routing

**Category-Based Routing** — Direct requests to different model groups based on detected prompt category, with per-category sub-strategies and a configurable default fallback

[https://docs.fastrouter.ai/explore-features/virtual-model-aliases](https://docs.fastrouter.ai/explore-features/virtual-model-aliases)
{% endupdate %}

{% update date="2026-03-20" %}
## Added Traceparent Support

**Tracing (W3C `traceparent` support)** — Group multiple LLM API calls into a single trace with ordered spans

**Traces view in Activity** — Visualize execution timelines, latency, tokens, and cost across spans

[https://docs.fastrouter.ai/tracing](https://docs.fastrouter.ai/tracing)
{% endupdate %}
{% endupdates %}
