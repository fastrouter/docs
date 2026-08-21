---
description: >-
  FastRouter supports prompt caching on all major providers that offer it, with
  automatic sticky routing to maximize cache hits.
icon: database
---

# Prompt Caching

## Overview

Prompt caching reduces the cost of repeated context — long system prompts, RAG chunks, documents — by charging a fraction of the normal input price on cache hits. Reduce inference costs by caching repeated prompt content across requests.

## Sticky Routing

Caches live on the provider that wrote them, so a cache is only useful if your follow-up requests land back on the same endpoint. FastRouter handles this for you.

When a provider response shows prompt-cache activity — a non-zero `cached_tokens`, `cache_write_tokens`, or the equivalent Anthropic cache fields — FastRouter records which provider served that conversation. Subsequent requests with the same API key, model, and opening messages are routed to that provider first, keeping its prompt cache warm.

A conversation is identified by hashing the first system (or developer) message together with the first non-system message. Every follow-up turn in the same thread produces the same fingerprint, while distinct conversations naturally spread across providers.

Sticky sessions expire after 24 hours without a cache-activity update. If the preferred provider is unavailable, FastRouter falls back to the next provider in your routing list.

Sticky routing is skipped entirely when you set:

* `provider.order`
* `provider.only`
* `previous_response_id`

In those cases your own routing instructions take precedence.

## Zero-Config Providers

The following providers cache automatically. No changes to your requests needed.

<table data-search="false"><thead><tr><th>Provider</th><th>Cache write</th><th>Cache read</th></tr></thead><tbody><tr><td>OpenAI</td><td>Free (1.25x input on GPT-5.6 and newer)</td><td>0.25x – 0.50x input</td></tr><tr><td>DeepSeek</td><td>Same as input</td><td>~0.10x input</td></tr><tr><td>Google AI Studio</td><td>Free</td><td>0.10x input</td></tr><tr><td>Google Vertex AI</td><td>Free</td><td>0.10x input</td></tr><tr><td>Grok</td><td>Free</td><td>See provider pricing</td></tr><tr><td>Moonshot AI</td><td>Free</td><td>See provider pricing</td></tr><tr><td>Baseten</td><td>Free</td><td>See provider pricing</td></tr></tbody></table>

**OpenAI** requires a minimum of 1024 tokens. If you want direct control over where the cache boundary falls, see [OpenAI Explicit Prompt Caching](https://claude.ai/chat/546169f8-c8a5-475e-8fd4-d5640ad25e63#openai-explicit-prompt-caching) below.

**Google AI Studio and Vertex AI** both support implicit caching on Gemini 2.5 and newer models — no configuration needed. FastRouter keeps your prompt prefixes stable to maximize cache hits. The 0.10x cache-read rate (90% discount) applies to all Gemini 2.5+ models; legacy Gemini 2.0 Flash is discounted at 0.25x. Implicit caches are managed entirely by Google's serving infrastructure with no storage cost to you. TTL is typically 3–5 minutes. To maximize cache hits, keep large static content (system instructions, RAG context, few-shot examples) at the beginning of your prompt and push dynamic content to the end.

Minimum token thresholds before caching applies:

| Model                 | Min tokens |
| --------------------- | ---------- |
| Gemini 2.5 Pro        | 4,096      |
| Gemini 2.5 Flash      | 1,024      |
| Gemini 2.5 Flash-Lite | 1,024      |

***

## OpenAI Explicit Prompt Caching

Automatic caching lets OpenAI decide where the reusable prefix ends. Explicit caching lets you decide instead — useful when the stable part of your prompt is a document, a tool schema, or a long set of few-shot examples that you know will repeat.

Supported on both the [Chat Completions](https://claude.ai/api-reference/chat/create-a-chat-completion) and [Responses](https://claude.ai/api-reference/responses/create-a-response) APIs.

> Explicit prompt caching requires OpenAI GPT-5.6 or newer. Requests to older models are served by automatic caching.

**Pricing**

| Operation   | Cost                                                              |
| ----------- | ----------------------------------------------------------------- |
| Cache write | 1.25x input — the same rate as automatic cache writes on GPT-5.6+ |
| Cache read  | The model's standard discounted cache-read rate                   |

**Controls**

There are two fields:

* **`prompt_cache_breakpoint`** — placed on an individual text content block (`input_text` on Responses, `text` on Chat Completions). It marks the end of a reusable prefix: everything up to and including that block becomes the candidate cached prefix. Automatic caching stays on alongside it.
* **`prompt_cache_options`** — placed at the request root. Set `mode` to `"explicit"` to turn off OpenAI-managed breakpoints so that only the blocks you marked participate in caching. Use `ttl` to request a cache duration.

`prompt_cache_key` is an optional session identifier that is forwarded upstream and helps OpenAI route repeat traffic to the same cache. It is independent of FastRouter's own sticky routing, which works whether or not you send it.

**TTL**

Cached prefixes have a minimum TTL of 30 minutes. FastRouter does not set or rewrite a default — whatever you pass in `prompt_cache_options.ttl` is forwarded unchanged. OpenAI currently accepts `"30m"`, so that is the value we suggest sending.

**Responses API**

```bash
{
  "model": "openai/...",
  "prompt_cache_key": "my-session-key",
  "prompt_cache_options": {
    "mode": "explicit",
    "ttl": "30m"
  },
  "input": [
    {
      "role": "user",
      "content": [
        {
          "type": "input_text",
          "text": "<REUSABLE_PREFIX>",
          "prompt_cache_breakpoint": {
            "mode": "explicit"
          }
        },
        {
          "type": "input_text",
          "text": "<TASK_SPECIFIC_SUFFIX>"
        }
      ]
    }
  ]
}
```

**Chat Completions API**

```bash
{
  "model": "openai/...",
  "prompt_cache_key": "my-session-key",
  "prompt_cache_options": {
    "mode": "explicit",
    "ttl": "30m"
  },
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "<REUSABLE_PREFIX>",
          "prompt_cache_breakpoint": {
            "mode": "explicit"
          }
        },
        {
          "type": "text",
          "text": "<TASK_SPECIFIC_SUFFIX>"
        }
      ]
    }
  ]
}
```

For upstream behavior and edge cases, see OpenAI's [prompt caching guide](https://developers.openai.com/api/docs/guides/prompt-caching?prompt-cache-api=chat-completions#prompt-cache-breakpoints).

***

## Anthropic Claude

Anthropic requires you to explicitly mark what should be cached using `cache_control`. FastRouter supports two approaches.

**Option A — Top-level (recommended for chat)**

Add `cache_control` once at the request root. FastRouter automatically places the cache breakpoint at the last cacheable block and advances it as the conversation grows.

```json
{
  "model": "anthropic/claude-sonnet-4.6",
  "cache_control": { "type": "ephemeral" },
  "messages": [...]
}
```

> Only works when routed to Anthropic directly.

**Option B — Per-block (for precise control)**

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

**TTL**

| TTL             | Syntax                                 | Write cost  | Read cost   |
| --------------- | -------------------------------------- | ----------- | ----------- |
| 5 min (default) | `{ "type": "ephemeral" }`              | 1.25x input | 0.10x input |
| 1 hour          | `{ "type": "ephemeral", "ttl": "1h" }` | 2x input    | 0.10x input |

Use the 1-hour TTL for long sessions where repeated 5-minute cache re-writes would cost more than the higher write price.

**Model minimums**

| Min tokens | Models                                   |
| ---------- | ---------------------------------------- |
| 4096       | Opus 4.5, 4.6, 4.7 · Haiku 4.5           |
| 2048       | Sonnet 4.6 · Haiku 3.5                   |
| 1024       | Sonnet 4, 4.5 · Opus 4, 4.1 · Sonnet 3.7 |

***

## Checking Cache Savings

Every API response includes a `prompt_tokens_details` object:

```json
"prompt_tokens_details": {
  "cached_tokens": 10318,
  "cache_write_tokens": 0
}
```

`cached_tokens` > 0 means you're hitting the cache. These are also the fields FastRouter watches to decide when to pin a conversation to a provider — see [Sticky Routing](https://claude.ai/chat/546169f8-c8a5-475e-8fd4-d5640ad25e63#sticky-routing).

You can also check per-request cache usage on the **Activity Logs** page flyout on the FastRouter dashboard.
