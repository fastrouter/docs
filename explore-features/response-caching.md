---
description: >-
  Response Caching allows FastRouter.ai users to cache LLM responses for
  repeated or similar prompts.
icon: lasso-sparkles
---

# Response Caching

### Overview

Response Caching delivers delivers **faster response times**, **lower costs**, and **consistent outputs** across applications.

Caching is especially effective for:

* Dashboards
* Chatbots and agents
* FAQs and support flows
* APIs with predictable or repetitive queries

FastRouter supports **exact-match** and **semantic-match** caching with flexible controls.

***

### Key Benefits

| Benefit                  | Description                                               |
| ------------------------ | --------------------------------------------------------- |
| Faster Responses         | Cache hits return in <10ms                                |
| Cost Reduction           | Cache hits billed at **0.1× token pricing** (90% savings) |
| Consistent Outputs       | Identical or similar inputs return consistent responses   |
| Reduced Provider Load    | Fewer upstream API calls, improved rate-limit headroom    |
| Conversation Flexibility | Multiple caching strategies for multi-turn chats          |
| Custom Cache Keys        | User-defined namespaces for precise cache control         |

***

### Feature Specification

#### Request Schema

Caching is enabled by including a `cache_key` header and an optional `cache` configuration object in the request body.

***

#### Headers

| Header        | Type   | Required          | Description                                                   |
| ------------- | ------ | ----------------- | ------------------------------------------------------------- |
| Authorization | string | Yes               | Bearer token with API key                                     |
| Content-Type  | string | Yes               | `application/json`                                            |
| cache\_key    | string | Yes (for caching) | User-defined cache namespace. If omitted, caching is disabled |

***

#### Request Body

```json
{
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
}
```

***

#### Sample Request

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
    "cache": {
      "filter_on_model": true,
      "expiration_time": 3600
    }
  }'
```

***

### Cache Key Header

The `cache_key` header defines the **primary cache namespace**.

**Purpose**

* Groups related requests under a shared cache scope

**Examples**

* `myapp-faq`
* `user_123_session`
* `product-descriptions`
* `chatbot-v2`

FastRouter combines `cache_key` with hashed request attributes to form the final lookup key.

***

### Cache Object Parameters

| Parameter             | Type    | Default             | Required    | Description                                                                 |
| --------------------- | ------- | ------------------- | ----------- | --------------------------------------------------------------------------- |
| expiration\_time      | integer | 3600                | No          | Cache TTL in seconds (60–86400)                                             |
| filter\_on\_model     | boolean | true                | No          | Match cache on model name                                                   |
| filter\_on\_provider  | boolean | false               | No          | Match cache on provider                                                     |
| conversation\_mode    | string  | `full_conversation` | No          | How conversation context is matched                                         |
| last\_n\_turns        | integer | 2                   | Conditional | Used only when `conversation_mode = last_n_turns`                           |
| similarity\_threshold | number  | 0.75                | No          | Minimum semantic similarity score (0–1) required to reuse a cached response |

***

#### 🔍 `similarity_threshold` Explained

* Enables **semantic caching** in addition to exact matches
* A value of:
  * `1.0` → exact match only
  * `0.75` (default) → allows minor rewording or paraphrases
  * `<0.7` → more aggressive reuse (use with caution)

If no cached entry meets the threshold, the request is treated as a **cache miss**.

***

### Conversation Modes

| Mode                | Description                     | Use Case                 |
| ------------------- | ------------------------------- | ------------------------ |
| full\_conversation  | Entire message history included | Stateful conversations   |
| last\_message\_only | Only last user message          | FAQs, stateless bots     |
| last\_n\_turns      | Last N user–assistant pairs     | Context-aware assistants |

**Turn Definition:**\
One turn = one user message + one assistant response.

***

### Cache Lookup

#### Cache Lookup Components

The final cache lookup is computed based on:

```
  org_id,
  model?,            // if filter_on_model = true
  provider?,         // if filter_on_provider = true
  prompt_messages,
  temperature,
  top_p,
  max_tokens
```

> `similarity_threshold` is applied **after lookup** to determine semantic eligibility.

***

#### Prompt Messages For Lookup

| Conversation Mode   | Messages Included             |
| ------------------- | ----------------------------- |
| full\_conversation  | All messages                  |
| last\_message\_only | Last user message             |
| last\_n\_turns      | Last N turns + system message |

***

#### Parameter Sensitivity

Always included in response caching:

| Parameter   | Notes                                   |
| ----------- | --------------------------------------- |
| temperature | Different values → different cache keys |
| top\_p      | Different values → different cache keys |
| max\_tokens | Different values → different cache keys |

Ignored for cache hashing:

* `stream`
* `user`
* `n`
* `frequency_penalty`
* `presence_penalty`
* `stop`

***

### API Responses

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

***

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

***

#### Cache Response Fields

| Field      | Type    | Description                                 |
| ---------- | ------- | ------------------------------------------- |
| cached     | boolean | True when served from cache                 |
| similarity | number  | Semantic similarity score (1 = exact match) |
| usage.cost | number  | Cache hits billed at 0.1×                   |

***

### Pricing

#### Cache Pricing

| Scenario      | Pricing                   |
| ------------- | ------------------------- |
| Cache HIT     | 0.1× standard token price |
| Cache MISS    | Standard token price      |
| Cache Storage | Free                      |

***

#### Pricing Formula

```
cache_hit_cost =
(prompt_tokens × input_price × 0.1) +
(completion_tokens × output_price × 0.1)
```

**Savings:** \~90%

***

### Streaming Support

#### Cached Streaming Responses

On cache hit + `stream: true`:

* Cached response is chunked and streamed
* Minimal artificial delay (default: 0ms)

***

#### Streaming Behavior

| Scenario              | Behavior                          |
| --------------------- | --------------------------------- |
| Cache MISS + stream   | Streamed from provider and cached |
| Cache HIT + stream    | Cached response streamed          |
| Cache HIT + no stream | Returned instantly                |
