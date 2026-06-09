---
name: FastRouter
description: Use when routing AI requests through a unified LLM gateway, managing multiple providers, setting up virtual model aliases, configuring fallback policies, enabling BYOK (Bring Your Own Key), processing batch requests, tracking costs, enforcing guardrails, evaluating model outputs, or integrating the MCP Gateway. Also use when setting up FastRouter as a model provider in OpenClaw — triggered by phrases like "set up fastrouter", "add fastrouter provider", "configure fastrouter with API key sk-v1-xxxxx", or "update fastrouter models". Reach for this skill when building AI applications that need multi-provider support, reliability, cost optimization, multimodal capabilities, or analytics.
metadata:
    docs-proj: fastrouter
    version: "1.0"
---

# FastRouter Skill

## Product Summary

FastRouter.ai is an enterprise-grade LLM Gateway that acts as a control plane for routing requests across 100+ AI models from multiple providers through a single OpenAI-compatible API. It provides intelligent routing, automatic failover, cost governance, multimodal support (text, image, video, audio), and built-in observability — eliminating vendor lock-in while reducing AI spend. No setup fees, no monthly minimums, and free credits to start.

FastRouter.ai sits between your application and LLM providers (OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more), handling request routing, credential management, cost tracking, error fallback, and performance optimization. Key differentiators include the Auto Router (dynamic model selection by cost/latency/quality), Virtual Model Aliases (custom model pools with policy-driven selection), per-key budget controls, batch processing, custom evaluations, guardrails, MCP Gateway, and native support for multiple API formats (OpenAI Chat Completions, OpenAI Responses, Anthropic Messages, Gemini Native).

See [docs.fastrouter.ai](https://docs.fastrouter.ai) for full documentation.

---

## When to Use

Reach for FastRouter when:

- **Multi-model routing**: You need to route requests across providers for cost, latency, or quality optimization
- **High availability**: You want automatic failover and retries across providers when one goes down
- **Cost governance**: You need per-key budgets, rate limits, model restrictions, and real-time spend tracking
- **Model comparison**: You want side-by-side evaluation of model outputs in interactive playgrounds
- **Multimodal pipelines**: You're working with text, image, video, or audio through one API with consistent auth and billing
- **Batch processing**: You're running bulk operations (up to 50K requests) across OpenAI and Anthropic models
- **Enterprise controls**: You need RBAC, BYOK, provisioning keys, project isolation, and audit logging
- **IDE integration**: You want a drop-in replacement for OpenAI in Cursor, Cline, Claude Code, and other tools
- **OpenClaw setup**: A user wants to add FastRouter as a provider in OpenClaw (see OpenClaw section below)

---

## Quick Reference

### Base URLs

| URL | Purpose |
|-----|---------|
| `https://go.fastrouter.ai/api/v1` | Primary API gateway (LLM endpoints) |
| `https://api.fastrouter.ai/api/v1` | Alternative API gateway |
| `https://api.fastrouter.ai/prod/` | Provisioning / admin endpoints only |
| `https://api.fastrouter.ai/v1/models` | Model catalog (for OpenClaw setup) |

### Authentication

All endpoints use Bearer token authentication:

```
Authorization: Bearer YOUR_FASTROUTER_API_KEY
Content-Type: application/json
```

### Core API Endpoints

| Endpoint | Method | Path | Description |
|----------|--------|------|-------------|
| Chat Completions | POST | `/api/v1/chat/completions` | Text generation, function calling, structured outputs |
| Responses | POST | `/api/v1/responses` | OpenAI Responses API format |
| Embeddings | POST | `/api/v1/embeddings` | Vector embeddings |
| Image Generation | POST | `/api/v1/images/generations` | Image creation (GPT Image 1, DALL-E) |
| Image Edit | POST | `/api/v1/images/edits` | Image editing |
| Video Generation | POST | `/api/v1/videos` | Video creation (Veo 3, Sora 2, Kling, etc.) |
| Video Status | POST | `/api/v1/getVideoResponse` | Poll video generation status |
| Audio Transcription | POST | `/v1/audio/transcriptions` | Speech-to-text (Whisper) |
| Audio Translation | POST | `/v1/audio/translations` | Audio to English text |
| Text-to-Audio | POST | `/api/v1/chat/completions` | Audio generation (ace-step model) |
| Audio Status | POST | `/api/v1/getPromptToAudioResponse` | Poll audio generation status |
| List Models | GET | `/api/v1/models` | Full model catalog with metadata |
| Generations | GET | `/api/v1/generation` | Request generation details/stats |
| Moderations | POST | (see docs) | Content moderation |

### Model Naming Convention

Models use the `provider/model-name` format:

```
openai/gpt-5.2
anthropic/claude-4.5-sonnet
google/gemini-3.1-pro-preview
x-ai/grok-4.1-fast
perplexity/sonar-pro
```

Append `:online` to enable web search on any model: `x-ai/grok-4.1-fast:online`

### Common Model IDs

| Provider | Example Models |
|----------|----------------|
| OpenAI | `openai/gpt-5.4-nano`, `openai/gpt-5.4`, `openai/gpt-5.4-mini`, `openai/o4-mini`, `openai/gpt-image-1`, `openai/sora-2-pro` |
| Anthropic | `anthropic/claude-opus-4.6`, `anthropic/claude-sonnet-4.6`, `anthropic/claude-haiku-4.5` |
| Google | `google/gemini-3.1-pro-preview`, `google/gemini-3.1-flash-image-preview`, `google/veo3.1-fast`, `google/gemini-3.1-flash-lite-preview` |
| xAI | `x-ai/grok-4`, `x-ai/grok-4.20-beta` |
| Minimax | `minimax/minimax-m2.7`, `minimax/minimax-m2.5-highspeed` |
| Perplexity | `perplexity/sonar-pro`, `perplexity/sonar-reasoning-pro` |
| Audio | `ace-step/prompt-to-audio` |
| Video | `kling-ai/kling-v3`, `wanx/wan-v2-6`, `bytedance/seedance-pro` |

### Request Parameters

```json
{
  "model": "openai/gpt-5.4",
  "messages": [{"role": "user", "content": "..."}],
  "stream": true,
  "temperature": 0.7,
  "max_completion_tokens": 2000,
  "response_format": {"type": "json_schema", "json_schema": {...}},
  "tools": [...],
  "tool_choice": "auto",
  "web_search_options": {"search_context_size": "medium"}
}
```

**FastRouter-specific parameters** (pass via `extra_body` in OpenAI SDK):

| Parameter | Type | Description |
|-----------|------|-------------|
| `reasoning` | object | `{"max_tokens": N}` — enable reasoning/thinking tokens |
| `tags` | array | `["tag1", "tag2"]` — custom metadata tags for analytics filtering |

### Key Types

| Key Type | Purpose |
|----------|---------|
| **API Key** | Standard LLM requests via `Authorization: Bearer` |
| **Provisioning Key** | Admin-only; creates/manages Service Account Keys |
| **Service Account Key** | Scoped access keys with per-key budgets, rate limits, model restrictions |

### Routing Strategy Types

| Strategy | Use Case |
|----------|----------|
| **Auto Router** | Dynamic model selection by cost/latency/quality — no configuration needed |
| **Virtual Model Aliases** | Custom pool of models with policy-driven selection (weighted, priority, round-robin, cost-optimized) |
| **Fallback Models** | Prioritized fallback chain across providers for high availability |
| **Provider Routing** | Multi-provider selection for same model (lowest latency, lowest cost, round robin, weighted, priority) |

---

## Decision Guidance

### When to Use Each Routing Strategy

| Scenario | Strategy |
|----------|----------|
| No strong model preference, want best value | **Auto Router** |
| A/B testing or gradual rollout across models | **Virtual Model Alias** (weighted) |
| Critical workload needing guaranteed uptime | **Fallback Models** (priority chain) |
| Same model available on multiple providers | **Provider Routing** (lowest latency/cost) |
| Category-based tasks needing different models | **Virtual Model Alias** (category routing) |
| Cost-first, quality-second selection | **Auto Router** (cost-optimized mode) |

### When to Use BYOK vs. FastRouter Credits

| Scenario | Recommendation |
|----------|----------------|
| Getting started / low volume | FastRouter credits (simpler) |
| You have existing provider contracts | **BYOK** — use your own rate limits and billing |
| Need specific model versions | **BYOK** — you control the provider account |
| Azure or AWS Bedrock integration | **BYOK** — required for these providers |
| Simplicity is priority | FastRouter credits |
| Cost optimization across providers | BYOK + FastRouter credits as fallback |

### When to Use Batch vs. Real-time

| Scenario | Use Batch | Use Real-time |
|----------|-----------|---------------|
| Dataset processing (10K+ rows) | ✅ Yes | ❌ No |
| User-facing chatbot | ❌ No | ✅ Yes |
| Scheduled overnight jobs | ✅ Yes | ❌ No |
| Latency-sensitive requests | ❌ No | ✅ Yes |
| OpenAI or Anthropic only | ✅ Yes | ✅ Yes |
| Cost-sensitive bulk inference | ✅ Yes | ❌ No |

---

## Workflows

### 1. Basic Setup (Python / OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://go.fastrouter.ai/api/v1",
    api_key="YOUR_FASTROUTER_API_KEY"
)

response = client.chat.completions.create(
    model="openai/gpt-5.4",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

### 2. Basic Setup (cURL)

```bash
curl -X POST https://go.fastrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "openai/gpt-5.4",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is FastRouter?"}
    ]
  }'
```

### 3. Structured JSON Output

```python
response = client.chat.completions.create(
    model="openai/gpt-4o",
    messages=[
        {"role": "system", "content": "Respond in JSON format only."},
        {"role": "user", "content": "Give me weather for London."}
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "weather",
            "strict": True,
            "schema": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                    "temperature": {"type": "number"},
                    "conditions": {"type": "string"}
                },
                "required": ["location", "temperature", "conditions"],
                "additionalProperties": False
            }
        }
    }
)
```

### 4. Streaming with Reasoning Tokens

```python
response = client.chat.completions.create(
    model="google/gemini-3.1-pro-preview",
    messages=[{"role": "user", "content": "Explain quantum entanglement"}],
    stream=True,
    extra_body={"reasoning": {"max_tokens": 2000}}
)
for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### 5. Web Search

```python
# Option A: Use a search-native model
response = client.chat.completions.create(
    model="perplexity/sonar-pro",
    messages=[{"role": "user", "content": "Latest AI news today"}]
)

# Option B: Append :online to any model
response = client.chat.completions.create(
    model="openai/gpt-5.2:online",
    messages=[{"role": "user", "content": "Latest AI news today"}],
    extra_body={"web_search_options": {"search_context_size": "medium"}}
)
```

### 6. Batch Processing

Upload a JSONL file to the dashboard or API. Each line follows this format:

```json
{"custom_id": "req-1", "provider": "openai", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "openai/gpt-4.1-nano", "messages": [{"role": "user", "content": "Summarize this text..."}], "max_tokens": 500}}
{"custom_id": "req-2", "provider": "anthropic", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "anthropic/claude-haiku-4.5", "messages": [{"role": "user", "content": "Translate to French: Hello"}], "max_tokens": 100}}
```

Limits: up to 50,000 requests per file. Results delivered as downloadable JSONL within 24h.

### 7. Provisioning Keys (Programmatic Key Management)

```bash
# Create a scoped service key
curl -X POST https://api.fastrouter.ai/prod/createServiceKey \
  -H "Authorization: Bearer YOUR_PROVISIONING_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "credit_limit": 10.00,
    "reset_budget_interval": "monthly",
    "models": ["openai/gpt-4o", "anthropic/claude-haiku-4.5"],
    "rpm_limit": 60,
    "tpm_limit": 100000,
    "tags": ["team:engineering", "env:production"]
  }'
```

Provisioning endpoints:

| Action | Endpoint |
|--------|----------|
| Create key | `POST /prod/createServiceKey` |
| Update key | `POST /prod/updateServiceKey` |
| List keys | `POST /prod/getServiceKeys` |
| Delete key | `POST /prod/deleteServiceKey` |

> **Important**: Save the `api_key_id_hash` returned on creation — required for all future updates and deletions.

---

## OpenClaw Integration

Use this section when a user wants to set up FastRouter as a model provider in OpenClaw. Triggered by phrases like "set up fastrouter", "add fastrouter provider", "configure fastrouter", "update fastrouter models", or when a user provides an API key starting with `sk-v1-`.

### Inputs

- **API Key** (required): Starts with `sk-v1-` followed by a hex string. Ask for it if not provided.
- **Base URL** (optional): Defaults to `https://api.fastrouter.ai`

### Steps

**Step 1 — Extract the API key**

Parse the API key from the user's message. Must start with `sk-v1-`. Do NOT proceed without one.

**Step 2 — Fetch the live model list**

```
web_fetch url="https://api.fastrouter.ai/v1/models" extractMode="text"
```

**Step 3 — Filter models**

Keep only models where:
- `is_active` is `true`
- `architecture.output_modalities` includes `"text"`
- `architecture.input_modalities` includes `"text"` or `"image"`

For each qualifying model, extract:
- `id` — the model identifier
- `context_length` — context window size
- `top_provider.max_completion_tokens` — max output tokens (if 0 or missing, use `min(context_length, 8192)`)
- Input types: list of `"text"` and/or `"image"` from input_modalities

**Step 4 — Build the provider config**

Do NOT include a `"name"` key — OpenClaw rejects it:

```json
{
  "baseUrl": "https://api.fastrouter.ai",
  "api": "openai-completions",
  "apiKey": "THE_API_KEY",
  "models": [
    {
      "id": "provider/model-id",
      "name": "Display Name",
      "contextWindow": 128000,
      "maxTokens": 8192,
      "input": ["text", "image"],
      "cost": { "input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0 },
      "reasoning": false
    }
  ]
}
```

**Step 5 — Update openclaw.json**

1. Use `read` to load `~/.openclaw/openclaw.json`
2. Merge the provider into `models.providers.fastrouter` (preserving all other config)
3. Add model references to `agents.defaults.models` — for each model: `"fastrouter/MODEL_ID": {}`
4. Use `write` to save the updated config

**Step 6 — Restart the gateway** *(requires user approval)*

```bash
openclaw gateway restart
```

**Step 7 — Report to user**

Tell the user:
- How many models were added
- They can switch models with `/model fastrouter/MODEL_ID`
- Suggest popular models: claude, gpt, gemini, deepseek variants

### OpenClaw Error Handling

| Error | Action |
|-------|--------|
| API unreachable | Tell user the FastRouter API may be down; try again later |
| No qualifying models | Warn that no text/image models were found |
| Config file missing | Create the full structure from scratch |
| Invalid API key format | Ask user to double-check their key starts with `sk-v1-` |

### OpenClaw Notes

- Provider key in config is `fastrouter`
- Existing fastrouter config is replaced with fresh model list on update
- All other providers and settings are preserved
- Cost is set to zero — FastRouter handles billing separately
- Video-only and audio-only models are excluded from the model list

---

## Common Gotchas

- **Two base URLs**: `go.fastrouter.ai` and `api.fastrouter.ai` both serve LLM endpoints; provisioning uses `api.fastrouter.ai/prod/` exclusively
- **Model naming is required**: Always use `provider/model-name` format (e.g., `openai/gpt-4o`, not `gpt-4o`); requests with bare model names may fail or route incorrectly
- **Reasoning token continuity**: When using reasoning-capable models, you **must** pass `reasoning_details` from the assistant message back in follow-up requests or you will get errors in multi-turn conversations
- **Audio and video are async**: Text-to-audio and video generation return a job ID; you must poll `getPromptToAudioResponse` or `getVideoResponse` for the result — there is no synchronous response
- **Audio endpoint path differs**: Transcription/translation use `/v1/audio/...` (no `/api` prefix), unlike other endpoints at `/api/v1/...`
- **Batch file consistency**: All requests in a JSONL batch must target the same endpoint (chat completions OR embeddings), but can mix models and providers within that constraint
- **Batch provider support**: Batch processing currently supports OpenAI and Anthropic only — other providers are not available in batch mode
- **BYOK billing**: When using Bring Your Own Key, rate limits and costs are governed by your provider account, not FastRouter credits
- **Provisioning Keys ≠ API Keys**: Provisioning Keys cannot make LLM requests; they are admin-only for key lifecycle management
- **Service key hash**: Save the `api_key_id_hash` returned from `createServiceKey` — it is the only identifier for future updates and deletions
- **Web search pricing**: `:online` suffix and search-preview models cost $5 per 1,000 answers in addition to model token costs
- **Nano Banana (gemini-2.5-flash-image)**: Use through Chat Completions, not the Image Generation endpoint; supports custom `aspectRatio` parameter
- **Streaming + tool calls**: When using `stream: true` with function calling, tool calls only appear in the final streaming chunk
- **Free credits expire**: Promotional credits expire in 30 days

---

## Verification Checklist

- [ ] **API key** set in `Authorization: Bearer <key>` header
- [ ] **Base URL** changed to `https://go.fastrouter.ai/api/v1` (not OpenAI's URL)
- [ ] **Model name** follows `provider/model-name` format
- [ ] **Structured outputs**: `response_format` paired with a JSON-instructing system prompt
- [ ] **Reasoning continuity**: `reasoning_details` preserved in follow-up messages when using reasoning models
- [ ] **Batch JSONL**: Consistent endpoint across all lines; `provider` field included per request
- [ ] **Async operations**: Polling logic in place for video/audio result retrieval
- [ ] **Per-key budget + rate limits**: Configured in dashboard before production deployment
- [ ] **Tags applied**: `extra_body={"tags": [...]}` added for analytics filtering
- [ ] **Credits sufficient**: Monitored via `usage.credits_used` in responses
- [ ] **Service key hash saved**: `api_key_id_hash` stored securely if using Provisioning Keys
- [ ] **OpenClaw (if applicable)**: No `"name"` key in provider config; gateway restarted after config update

---

## Resources

| Resource | URL |
|----------|-----|
| Main website | https://fastrouter.ai |
| Documentation home | https://docs.fastrouter.ai |
| Model catalog | https://fastrouter.ai/models |
| Chat Completions API | https://docs.fastrouter.ai/api-reference/chat-completions |
| Responses API | https://docs.fastrouter.ai/api-reference/responses |
| Embeddings API | https://docs.fastrouter.ai/api-reference/embeddings |
| List Models API | https://docs.fastrouter.ai/api-reference/models |
| Auto Router | https://docs.fastrouter.ai/api-reference/auto-router |
| Image Generation API | https://docs.fastrouter.ai/api-reference/image-generation |
| Video Generation API | https://docs.fastrouter.ai/api-reference/video-generation |
| Transcriptions & Translations | https://docs.fastrouter.ai/api-reference/transcriptions-and-translations-api |
| Text-to-Audio API | https://docs.fastrouter.ai/api-reference/text-to-audio-generation-api |
| Batch Processing API | https://docs.fastrouter.ai/api-reference/batch-processing |
| Generations (Request Details) | https://docs.fastrouter.ai/api-reference/generations |
| Error Codes | https://docs.fastrouter.ai/api-reference/error-codes |
| Virtual Model Aliases | https://docs.fastrouter.ai/virtual-model-aliases |
| Fallback Models | https://docs.fastrouter.ai/fallback-models |
| Provider Routing Strategies | https://docs.fastrouter.ai/provider-routing-strategies |
| Automatic Model Selection | https://docs.fastrouter.ai/automatic-model-selection |
| BYOK (External Keys) | https://docs.fastrouter.ai/add-external-keys-byok |
| Guardrails | https://docs.fastrouter.ai/guardrails |
| Custom Evaluations | https://docs.fastrouter.ai/custom-evaluations |
| Batch Processing (Feature) | https://docs.fastrouter.ai/batch-processing |
| Structured Outputs | https://docs.fastrouter.ai/structured-outputs |
| Function Calling | https://docs.fastrouter.ai/function-calling |
| Reasoning Tokens | https://docs.fastrouter.ai/reasoning-tokens |
| Response Caching | https://docs.fastrouter.ai/response-caching |
| MCP Gateway | https://docs.fastrouter.ai/mcp-gateway |
| Web Search | https://docs.fastrouter.ai/web-search |
| Dynamic Tags | https://docs.fastrouter.ai/dynamic-tags-per-request |
| File & Image Inputs | https://docs.fastrouter.ai/file-and-image-inputs |
| Tracing | https://docs.fastrouter.ai/tracing |
| Alerts | https://docs.fastrouter.ai/alerts |
| Credits | https://docs.fastrouter.ai/credits |
| Provisioning Keys | https://docs.fastrouter.ai/provisioning-keys |
| Keys & Settings | https://docs.fastrouter.ai/keys-and-settings |
| Projects | https://docs.fastrouter.ai/projects |
| Organization & Members | https://docs.fastrouter.ai/organization-and-members |
| IDE Integrations | https://docs.fastrouter.ai/integrations/ide-integrations |
| Claude Code Integration | https://docs.fastrouter.ai/integrations/claude-code |
| Changelog | https://docs.fastrouter.ai/changelog |

---

> For full documentation and navigation, see: https://docs.fastrouter.ai
