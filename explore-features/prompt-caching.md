---
description: >-
  FastRouter supports prompt caching on all major providers that offer it, with
  automatic sticky routing to maximize cache hits.
icon: database
---

# Prompt Caching

### Overview

Prompt caching reduces the cost of repeated context — long system prompts, RAG\
chunks, documents — by charging a fraction of the normal input price on cache hits. Reduce inference costs by caching repeated prompt content across requests.

### Sticky Routing

When a request benefits from caching, FastRouter pins subsequent requests for that model and conversation to the same provider endpoint so the cache stays warm. A "conversation" is identified by hashing the first system message and first user message — so different conversations naturally spread across providers while each individual conversation stays consistent.

Sticky routing only kicks in when the provider's cache read price is lower than its regular input price. If that provider goes down, FastRouter falls back automatically. If you've set a manual `provider.order`, your ordering takes precedence and sticky routing is skipped.

### Zero-Config Providers

The following providers cache automatically. No changes to your requests needed.

<table><thead><tr><th width="249.11328125">Provider</th><th>Cache write</th><th>Cache read</th></tr></thead><tbody><tr><td>OpenAI</td><td>Free</td><td>0.25x – 0.50x input</td></tr><tr><td>DeepSeek</td><td>Same as input</td><td>~0.10x input</td></tr><tr><td>Google AI Studio</td><td>Free</td><td>0.10x input</td></tr><tr><td>Google Vertex AI</td><td>Free</td><td>0.10x input</td></tr><tr><td>Grok</td><td>Free</td><td>See provider pricing</td></tr><tr><td>Moonshot AI</td><td>Free</td><td>See provider pricing</td></tr><tr><td>Baseten</td><td>Free</td><td>See provider pricing</td></tr></tbody></table>

**OpenAI** requires a minimum of 1024 tokens.

**Google AI Studio and Vertex AI** both support implicit caching on Gemini 2.5 and newer models — no configuration needed. FastRouter keeps your prompt prefixes stable to maximize cache hits. The 0.10x cache-read rate (90% discount) applies to all Gemini 2.5+ models; legacy Gemini 2.0 Flash is discounted at 0.25x. Implicit caches are managed entirely by Google's serving infrastructure with no storage cost to you. TTL is typically 3–5 minutes. To maximize cache hits, keep large static content (system instructions, RAG context, few-shot examples) at the beginning of your prompt and push dynamic content to the end.

Minimum token thresholds before caching applies:

| Model                 | Min tokens |
| --------------------- | ---------- |
| Gemini 2.5 Pro        | 4,096      |
| Gemini 2.5 Flash      | 1,024      |
| Gemini 2.5 Flash-Lite | 1,024      |

***

### Anthropic Claude

Anthropic requires you to explicitly mark what should be cached using `cache_control`. FastRouter supports two approaches.

#### Option A — Top-level (recommended for chat)

Add `cache_control` once at the request root. FastRouter automatically places the cache breakpoint at the last cacheable block and advances it as the conversation grows.

```json
{
  "model": "anthropic/claude-sonnet-4.6",
  "cache_control": { "type": "ephemeral" },
  "messages": [...]
}
```

> Only works when routed to Anthropic directly.

#### Option B — Per-block (for precise control)

Place `cache_control` on individual content blocks. Useful when you have a large stable payload (a document, RAG chunks, a character card) and want to cache exactly that. Maximum 4 breakpoints per request.

```json
{
  "messages": [
    {
      "role": "system",
      "content": [
        { "type": "text", "text": "You are a research assistant." },
        {
          "type": "text",
          "text": "<large document>",
          "cache_control": { "type": "ephemeral" }
        }
      ]
    },
    { "role": "user", "content": "Summarize the findings." }
  ]
}
```

Per-block caching works across Anthropic and Vertex.

#### TTL

| TTL             | Syntax                                 | Write cost  | Read cost   |
| --------------- | -------------------------------------- | ----------- | ----------- |
| 5 min (default) | `{ "type": "ephemeral" }`              | 1.25x input | 0.10x input |
| 1 hour          | `{ "type": "ephemeral", "ttl": "1h" }` | 2x input    | 0.10x input |

Use the 1-hour TTL for long sessions where repeated 5-minute cache re-writes would cost more than the higher write price.

#### Model minimums

| Min tokens | Models                                   |
| ---------- | ---------------------------------------- |
| 4096       | Opus 4.5, 4.6, 4.7 · Haiku 4.5           |
| 2048       | Sonnet 4.6 · Haiku 3.5                   |
| 1024       | Sonnet 4, 4.5 · Opus 4, 4.1 · Sonnet 3.7 |

***

### Checking Cache Savings

Every API response includes a `prompt_tokens_details` object:

```json
"prompt_tokens_details": {
  "cached_tokens": 10318,
  "cache_write_tokens": 0
}
```

`cached_tokens` > 0 means you're hitting the cache.

You can also check per-request cache usage on the **Activity Logs** page flyout on the FastRouter dashboard.
