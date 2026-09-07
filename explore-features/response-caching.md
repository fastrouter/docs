---
description: >-
  This page covers caching for text completions (POST /v1/chat/completions). For
  image and video generation, see Response Caching: Images & Video (Async).
icon: lasso-sparkles
---

# Response Caching: Text

## Overview

Response Caching delivers **faster response times**, **lower costs**, and **consistent outputs**. For text, a cache hit returns the stored completion payload directly — in under 10ms — billed at **0.1× standard token pricing**.

Caching is especially effective for dashboards, chatbots, agents, FAQs, support flows, and any API with predictable or repetitive queries.

FastRouter supports **exact-match** and **semantic-match** caching with flexible controls.

***

## Key Benefits

| Benefit                  | Description                                               |
| ------------------------ | --------------------------------------------------------- |
| Faster Responses         | Cache hits return in <10ms                                |
| Cost Reduction           | Cache hits billed at **0.1× token pricing** (90% savings) |
| Consistent Outputs       | Identical or similar inputs return consistent responses   |
| Reduced Provider Load    | Fewer upstream API calls, improved rate-limit headroom    |
| Conversation Flexibility | Multiple caching strategies for multi-turn chats          |
| Custom Cache Keys        | User-defined namespaces for precise cache control         |

***

## Feature Specification

Caching is enabled by including a `cache_key` header and an optional `cache` configuration object in the request body.

#### Headers

| Header        | Type   | Required          | Description                                                   |
| ------------- | ------ | ----------------- | ------------------------------------------------------------- |
| Authorization | string | Yes               | Bearer token with API key                                     |
| Content-Type  | string | Yes               | `application/json`                                            |
| cache\_key    | string | Yes (for caching) | User-defined cache namespace. If omitted, caching is disabled |

#### Cache Key Header

The `cache_key` header defines the **primary cache namespace**. It groups related requests under a shared cache scope.

Examples: `myapp-faq`, `user_123_session`, `product-descriptions`, `chatbot-v2`

FastRouter combines `cache_key` with hashed request attributes to form the final lookup key. The endpoint is part of the key, so the same `cache_key` string used on `/v1/chat/completions` and on the media endpoints produces two independent namespaces — teams can reuse one namespace string across their stack without collision.

#### Cache Object Parameters

| Parameter             | Type    | Default             | Required    | Description                                                                 |
| --------------------- | ------- | ------------------- | ----------- | --------------------------------------------------------------------------- |
| expiration\_time      | integer | 3600                | No          | Cache TTL in seconds (60–86400)                                             |
| filter\_on\_model     | boolean | true                | No          | Match cache on model name                                                   |
| filter\_on\_provider  | boolean | false               | No          | Match cache on provider                                                     |
| conversation\_mode    | string  | `full_conversation` | No          | How conversation context is matched                                         |
| last\_n\_turns        | integer | 2                   | Conditional | Used only when `conversation_mode = last_n_turns`                           |
| similarity\_threshold | number  | 0.75                | No          | Minimum semantic similarity score (0–1) required to reuse a cached response |

#### 🔍 `similarity_threshold` Explained

* Enables **semantic caching** in addition to exact matches
* A value of:
  * `1.0` → exact match only
  * `0.75` (default) → allows minor rewording or paraphrases
  * `<0.7` → more aggressive reuse (use with caution)

If no cached entry meets the threshold, the request is treated as a **cache miss**.

***

## Sample Request

```bash
curl --location 'https://api.fastrouter.ai/v1/chat/completions' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'cache_key: CACHE-KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "openai/gpt-4.1-mini",
    "messages": [
      { "role": "user", "content": "Tell me about physics" }
    ],
    "max_tokens": 182,
    "stream": false,
    "cache": {
      "filter_on_provider": false,
      "filter_on_model": true,
      "expiration_time": 3600,
      "conversation_mode": "full_conversation",
      "last_n_turns": 2,
      "similarity_threshold": 0.75
    }
  }'
```

***

## Conversation Modes

| Mode                | Description                     | Use Case                 |
| ------------------- | ------------------------------- | ------------------------ |
| full\_conversation  | Entire message history included | Stateful conversations   |
| last\_message\_only | Only last user message          | FAQs, stateless bots     |
| last\_n\_turns      | Last N user–assistant pairs     | Context-aware assistants |

**Turn Definition:** One turn = one user message + one assistant response.

## Prompt Messages For Lookup

| Conversation Mode   | Messages Included             |
| ------------------- | ----------------------------- |
| full\_conversation  | All messages                  |
| last\_message\_only | Last user message             |
| last\_n\_turns      | Last N turns + system message |

***

## Cache Lookup

The final cache lookup is computed from:

```
  org_id,
  cache_key,         // header
  model?,            // if filter_on_model = true
  provider?,         // if filter_on_provider = true
  prompt_messages,   // per conversation_mode
  temperature,
  top_p,
  max_tokens
```

> `similarity_threshold` is applied **after lookup** to determine semantic eligibility.

## Parameter Sensitivity

Always included in the cache key:

| Parameter   | Notes                                   |
| ----------- | --------------------------------------- |
| temperature | Different values → different cache keys |
| top\_p      | Different values → different cache keys |
| max\_tokens | Different values → different cache keys |

Ignored for cache hashing: `stream`, `user`, `n`, `frequency_penalty`, `presence_penalty`, `stop`

***

## API Responses

#### Cache MISS

Returned normally and stored in cache.

```json
{
  "cached": false,
  "usage": {
    "prompt_tokens": 11,
    "completion_tokens": 182,
    "total_tokens": 193,
    "cost": 0.0002956
  }
}
```

#### Cache HIT

Returned instantly with cache metadata.

```json
{
  "cached": true,
  "similarity": 0.92,
  "usage": {
    "prompt_tokens": 11,
    "completion_tokens": 182,
    "total_tokens": 193,
    "cost": 0.00002956
  }
}
```

#### Cache Response Fields

| Field      | Type    | Description                                 |
| ---------- | ------- | ------------------------------------------- |
| cached     | boolean | True when served from cache                 |
| similarity | number  | Semantic similarity score (1 = exact match) |
| usage.cost | number  | Cache hits billed at 0.1×                   |

***

## Streaming Support

On a cache hit with `stream: true`, the cached response is chunked and streamed back, with minimal artificial delay (default: 0ms).

| Scenario              | Behavior                          |
| --------------------- | --------------------------------- |
| Cache MISS + stream   | Streamed from provider and cached |
| Cache HIT + stream    | Cached response streamed          |
| Cache HIT + no stream | Returned instantly                |

***

## Pricing

| Scenario      | Pricing                   |
| ------------- | ------------------------- |
| Cache HIT     | 0.1× standard token price |
| Cache MISS    | Standard token price      |
| Cache Storage | Free                      |

```
cache_hit_cost =
(prompt_tokens × input_price × 0.1) +
(completion_tokens × output_price × 0.1)
```

**Savings:** \~90%

***

## Limits

| Limit                  | Value                         |
| ---------------------- | ----------------------------- |
| Min TTL                | 60 s                          |
| Default TTL            | 3,600 s (1h)                  |
| Max TTL                | 86,400 s (24h)                |
| Max `cache_key` length | 128 chars, `[A-Za-z0-9_\-:.]` |
