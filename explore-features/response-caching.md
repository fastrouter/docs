---
description: >-
  Response Caching allows FastRouter.ai users to cache LLM responses for
  repeated or similar prompts.
icon: lasso-sparkles
---

# Response Caching

## Overview

Response Caching delivers **faster response times**, **lower costs**, and **consistent outputs** across applications. It works across the following generation modalities on FastRouter:

| Modality             | Generate                          | Retrieve                                                          | Caching status                      | What is cached                          |
| -------------------- | --------------------------------- | ----------------------------------------------------------------- | ----------------------------------- | --------------------------------------- |
| Text                 | `POST /v1/chat/completions`       | —                                                                 | **Supported**                       | Completion payload                      |
| Image (async)        | `POST /api/v1/images/generations` | `POST /api/v1/getAsyncResponse` or `GET /api/v1/images/{task_id}` | **Supported**                       | Task ID + `fastrouter_assets` + payload |
| Video (always async) | `POST /api/v1/videos`             | `POST /api/v1/getAsyncResponse`                                   | **Supported**                       | Task ID + `fastrouter_assets` + payload |
| Image (sync)         | `POST /api/v1/images/generations` | —                                                                 | **Not yet supported — coming soon** | —                                       |

Caching is especially effective for:

* Dashboards, chatbots, agents, FAQs and support flows (text)
* Product catalogs, thumbnails, templated creatives (images)
* Marketing loops, previews, repeated demo prompts (videos)

FastRouter supports **exact-match** and **semantic-match** caching with flexible controls. The request contract (`cache_key` header + optional `cache` object) is identical for every modality, so a single integration pattern works everywhere.

> **Media caching is async-only today.** For images and videos, the cache stores and returns a **task ID**. A cache hit returns the _same_ `task_id` / `taskId` as the original request, and the client retrieves the result through the normal polling flow. Synchronous image generation (`response_type: "sync"`, or a sync-capable model with `response_type` omitted) currently bypasses the cache entirely — no lookup, no store. Support is planned; see [Roadmap](response-caching.md#roadmap).

***

## Key Benefits

| Benefit                 | Description                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------------- |
| Faster Responses        | Text hits return in <10ms; media hits return an existing task ID, skipping the full generation wait     |
| Cost Reduction          | Text cache hits billed at **0.1× token pricing** (90% savings); image and video cache hits are **free** |
| Consistent Outputs      | Identical or similar inputs return the same text, image, or video                                       |
| Reduced Provider Load   | Fewer upstream calls, improved rate-limit headroom, fewer queued media jobs                             |
| In-flight Deduplication | Duplicate async requests share one provider job and one `taskId`                                        |
| Custom Cache Keys       | User-defined namespaces for precise cache control                                                       |

***

## Design Principles

1. **One contract, all modalities.** Same header, same `cache` object. Parameters that don't apply to a modality are ignored (never rejected).
2. **Modality-appropriate defaults.** Text defaults to semantic matching (`0.75`); images and videos default to exact matching (`1.0`) because paraphrases of creative prompts are rarely interchangeable.
3. **Async stays async.** A cache hit on an async request returns the _same_ `taskId` and the standard response shape, so clients need no special-casing — polling `getAsyncResponse` simply resolves immediately.
4. **Org-scoped.** Cache entries never cross organization boundaries, regardless of `cache_key`.

***

## Feature Specification

**Request Schema**

Caching is enabled by including a `cache_key` header and an optional `cache` configuration object in the request body. This applies uniformly to text, async image, and video endpoints.

***

**Headers**

| Header        | Type   | Required          | Description                                                   |
| ------------- | ------ | ----------------- | ------------------------------------------------------------- |
| Authorization | string | Yes               | Bearer token with API key                                     |
| Content-Type  | string | Yes               | `application/json`                                            |
| cache\_key    | string | Yes (for caching) | User-defined cache namespace. If omitted, caching is disabled |

***

**Cache Object Parameters**

| Parameter             | Type    | Default (Text)      | Default (Image/Video) | Applies to | Description                                                                    |
| --------------------- | ------- | ------------------- | --------------------- | ---------- | ------------------------------------------------------------------------------ |
| expiration\_time      | integer | 3600                | 57600 (16h)           | All        | Cache TTL in seconds. Text: 60–86400. Image/Video: 60–604800 (7 days)          |
| filter\_on\_model     | boolean | true                | true                  | All        | Match cache on model name                                                      |
| filter\_on\_provider  | boolean | false               | false                 | All        | Match cache on provider                                                        |
| similarity\_threshold | number  | 0.75                | 1.0                   | All        | Minimum semantic similarity score (0–1) on the prompt to reuse a cached result |
| conversation\_mode    | string  | `full_conversation` | —                     | Text only  | How conversation context is matched. Ignored for image/video                   |
| last\_n\_turns        | integer | 2                   | —                     | Text only  | Used only when `conversation_mode = last_n_turns`. Ignored for image/video     |

***

**🔍 `similarity_threshold` Explained**

* Enables **semantic caching** in addition to exact matches
* A value of:
  * `1.0` → exact match only (default for images and videos)
  * `0.75` → allows minor rewording or paraphrases (default for text)
  * `<0.7` → more aggressive reuse (use with caution)

If no cached entry meets the threshold, the request is treated as a **cache miss**.

For image and video requests, semantic matching is applied to the **prompt text only**. All other generation parameters (size, length, resolution, reference media, etc.) must match exactly regardless of threshold.

> **Why exact match by default for media?** "A red car on a beach" and "a crimson car by the sea" may embed with 0.9 similarity, but a user rarely wants the _same_ image for both. Lower the threshold deliberately for use cases like FAQ illustrations or placeholder assets where interchangeability is acceptable.

***

## Text Caching

**Sample Request**

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

**Conversation Modes**

| Mode                | Description                     | Use Case                 |
| ------------------- | ------------------------------- | ------------------------ |
| full\_conversation  | Entire message history included | Stateful conversations   |
| last\_message\_only | Only last user message          | FAQs, stateless bots     |
| last\_n\_turns      | Last N user–assistant pairs     | Context-aware assistants |

**Turn Definition:** One turn = one user message + one assistant response.

**Prompt Messages For Lookup**

| Conversation Mode   | Messages Included             |
| ------------------- | ----------------------------- |
| full\_conversation  | All messages                  |
| last\_message\_only | Last user message             |
| last\_n\_turns      | Last N turns + system message |

**Parameter Sensitivity (Text)**

Always included in the cache key:

| Parameter   | Notes                                   |
| ----------- | --------------------------------------- |
| temperature | Different values → different cache keys |
| top\_p      | Different values → different cache keys |
| max\_tokens | Different values → different cache keys |

Ignored for cache hashing: `stream`, `user`, `n`, `frequency_penalty`, `presence_penalty`, `stop`

**Responses**

Cache MISS:

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

Cache HIT:

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

**Streaming Behavior**

| Scenario              | Behavior                                                   |
| --------------------- | ---------------------------------------------------------- |
| Cache MISS + stream   | Streamed from provider and cached                          |
| Cache HIT + stream    | Cached response streamed (0ms artificial delay by default) |
| Cache HIT + no stream | Returned instantly                                         |

***

## Media Caching — Shared Behavior (Async Image + Video)

Async image generation and video generation share one cache implementation. Both return a task ID at submission and are retrieved through `POST /api/v1/getAsyncResponse`.

**Eligibility**

| Request                                                                                     | Cached?                                          |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `POST /api/v1/videos` (always async)                                                        | Yes                                              |
| `POST /api/v1/images/generations` with `response_type: "async"`                             | Yes                                              |
| `POST /api/v1/images/generations` on an async-only model (e.g. Leonardo, Flux via Leonardo) | Yes — routed async regardless of `response_type` |
| `POST /api/v1/images/generations` processed synchronously                                   | **No** — cache is bypassed (no lookup, no store) |

A sync-processed request neither reads from nor writes to the cache, so it will not populate an entry that a later async request could hit.

**Task ID Semantics on Hits**

* A hit returns the **original task ID** (`fr_`-prefixed) — `task_id` for images, `data.taskId` for videos. It is a real, pollable task; `getAsyncResponse` with that ID works regardless of which request "owns" it.
* Each hitting request still gets its **own `chat_id`**, so it appears in the Activity Log with its own row.
* Task IDs are org-scoped; a `taskId` is never returned to a different organization.

**Identifying a Cache Hit**

On the generation response, a hit is identifiable by its zeroed estimate:

| Field                         | MISS                                       | HIT                           |
| ----------------------------- | ------------------------------------------ | ----------------------------- |
| `estimate_usage.credits_used` | Estimated cost of the job                  | `0`                           |
| `estimate_usage.provider`     | Routed provider (e.g. `leonardo`, `pollo`) | `""` — no provider is engaged |
| `task_id` / `data.taskId`     | Newly minted                               | The original task's ID        |

> Explicit `cached`, `similarity`, and `cache_age` fields are returned on text responses today but **not yet on media responses**. See [Roadmap](https://claude.ai/chat/333dd98d-417e-49f5-9c43-7e4b1c22c852#roadmap).

**Model Echo on Hits**

A MISS is routed to a provider, so the response echoes the **provider's** model string (`flux-dev`, `phoenix`, `seedance-1-5-pro`). A HIT engages no provider, so it echoes the **requested slug** as sent (`leonardo-ai/phoenix`, `kling-ai/kling-v1-6`). Both refer to the same model; don't key client logic on the exact string.

For cache lookup, FastRouter normalizes to its **canonical model slug** — `bytedance/seedance-1.5-pro` and `bytedance/seedance-1-5-pro` hash identically. With `filter_on_provider: false` (the default), the same model routed through different providers shares one entry.

**Serving `fastrouter_assets`**

`getAsyncResponse` returns both the provider `data[].url` and `fastrouter_assets.urls`. Clients should treat `fastrouter_assets.urls` as authoritative — signed provider URLs (Leonardo CDN, Pollo CDN, TOS-signed links) expire well before the hosted copy does.

`fastrouter_assets.cached_at` / `expires_at` describe **asset hosting retention** (up to 30 days depending on plan), which is independent of the response-cache TTL set by `expiration_time`. The cache TTL governs whether a _new request_ is matched to an existing task; asset retention governs how long the hosted URL stays downloadable.

**Usage on `getAsyncResponse`**

`getAsyncResponse` returns the **original task's** record, including the `usage` of the generation that actually ran. It is not re-billed per poll, and it is not zeroed for callers that arrived via a cache hit. Cost savings should be measured from the generation response (`estimate_usage`) or the Activity Log, not from the polling response.

***

## Image Caching (Async)

**Sample Request**

```bash
curl --location 'https://api.fastrouter.ai/api/v1/images/generations' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'cache_key: imagegen1' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "leonardo-ai/phoenix",
    "n": 1,
    "prompt": "a clean blue sky",
    "cache": {
      "expiration_time": 57600,
      "filter_on_model": true,
      "filter_on_provider": false
    }
  }'
```

To force async routing on a model that also has a sync provider, add `"response_type": "async"`.

**Parameter Sensitivity (Image)**

Always included in the cache key:

| Parameter                | Notes                                                                     |
| ------------------------ | ------------------------------------------------------------------------- |
| prompt                   | Subject to `similarity_threshold`                                         |
| n                        | Requesting 2 images ≠ requesting 4. See "n handling" below                |
| size                     | Exact match. `auto` is resolved to the model default before hashing       |
| quality                  | Exact match. `auto` resolved before hashing                               |
| style                    | Exact match                                                               |
| background               | Exact match                                                               |
| output\_format           | Exact match (changes the bytes produced)                                  |
| seed                     | Exact match if provided. Omitted seed and explicit seed are distinct keys |
| Reference / input images | Hashed by content (SHA-256 of bytes, or of the fetched URL body)          |
| mask (edits)             | Hashed by content                                                         |

Ignored for cache hashing: `response_format`, `user`, tracing headers (`traceparent`, `x-span-*`)

**`n` handling:** A cached entry for `n=4` is _not_ reused for `n=1`. Keeping `n` in the key avoids ambiguity over which subset to return and keeps billing simple.

**`response_format` handling:** Not hashed. The cached asset is served as `url` or `b64_json` per the hitting request.

**`response_type` handling:** Not hashed, but it determines eligibility — only async-processed requests participate in the cache.

**Responses**

`POST /api/v1/images/generations` — **Cache MISS**:

```json
{
  "created": 1788356206,
  "task_id": "fr_01M1H5C7F5EJ67C4528XJCRF40",
  "chat_id": "fr_357e96d9-2943-45c7-ad98-459f2a125441",
  "model": "flux-dev",
  "data": [
    { "revised_prompt": "A group of children playing cricket" }
  ],
  "estimate_usage": {
    "chat_id": "fr_357e96d9-2943-45c7-ad98-459f2a125441",
    "queue_time": 0,
    "input_tokens": 0,
    "output_tokens": 0,
    "total_tokens": 0,
    "api_key_credits_used": 0,
    "user_key_credits_used": 0.0346,
    "credits_used": 0.0346,
    "provider": "leonardo"
  }
}
```

`POST /api/v1/images/generations` — **Cache HIT**:

```json
{
  "created": 1788503458,
  "task_id": "fr_01M1NGQZ2ZMXRH64KTN1NKQ9XV",
  "chat_id": "fr_4824b8d2-e72d-4a79-a72a-6f078ae66b80",
  "model": "leonardo-ai/phoenix",
  "data": [
    { "revised_prompt": "a clean blue sky" }
  ],
  "estimate_usage": {
    "chat_id": "fr_4824b8d2-e72d-4a79-a72a-6f078ae66b80",
    "queue_time": 0,
    "input_tokens": 0,
    "output_tokens": 0,
    "total_tokens": 0,
    "api_key_credits_used": 0,
    "user_key_credits_used": 0,
    "credits_used": 0,
    "provider": ""
  }
}
```

`task_id` is the ID from the original generation; `chat_id` and `created` are new for this request.

`POST /api/v1/getAsyncResponse` (or `GET /api/v1/images/{task_id}`) — resolves immediately for a cached task:

```bash
curl --location 'https://api.fastrouter.ai/api/v1/getAsyncResponse' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'Content-Type: application/json' \
  --data '{ "taskId": "fr_01M1NGQZ2ZMXRH64KTN1NKQ9XV" }'
```

```json
{
  "chat_id": "fr_49415bfb-b0b8-4caa-8b68-2632cc1ff569",
  "created": 1788502550,
  "data": [
    {
      "id": "9078facb-cd97-49b5-8c92-110bde4fc50f",
      "url": "https://cdn.leonardo.ai/users/.../Phoenix_10_a_clean_blue_sky_0.jpg"
    }
  ],
  "fastrouter_assets": {
    "status": "ready",
    "urls": ["https://api.fastrouter.ai/images-content/fr_01M1NGQZ2ZMXRH64KTN1NKQ9XV"],
    "expires_at": 1791094363,
    "cached_at": 1788502363
  },
  "model": "phoenix",
  "size": "512x512",
  "usage": {
    "chat_id": "fr_49415bfb-b0b8-4caa-8b68-2632cc1ff569",
    "api_key_credits_used": 0,
    "user_key_credits_used": 0.0135,
    "credits_used": 0.0135,
    "provider": "leonardo"
  }
}
```

Note that `created` here (`1788502550`) predates the hitting request's `created` (`1788503458`) — it is the original generation's timestamp. The `usage` block likewise reports the original job's cost; the hitting request itself was not charged.

***

## Video Caching

Video generation is always asynchronous: `POST /api/v1/videos` returns `data.taskId`, and the client polls `POST /api/v1/getAsyncResponse` until `data.status` is `succeed` / `completed`. A cache hit returns the **same `taskId`**, so the standard polling flow works unchanged.

**Sample Request**

```bash
curl --location 'https://api.fastrouter.ai/api/v1/videos' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'cache_key: my-video-namespace1' \
  --header 'Content-Type: application/json' \
  --header 'traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01' \
  --header 'x-span-id: MY_SPAN_ID' \
  --header 'x-span-name: MY_SPAN_NAME' \
  --data '{
    "model": "kling-ai/kling-v1-6",
    "length": 5,
    "prompt": "A tennis match between 2 people in a sunny setup",
    "cache": {
      "expiration_time": 57600,
      "filter_on_model": true,
      "filter_on_provider": false
    }
  }'
```

**Parameter Sensitivity (Video)**

Always included in the cache key:

<table data-search="false"><thead><tr><th>Parameter</th><th>Notes</th></tr></thead><tbody><tr><td>prompt</td><td>Subject to <code>similarity_threshold</code></td></tr><tr><td>negative_prompt</td><td>Exact match</td></tr><tr><td>length</td><td>Exact match</td></tr><tr><td>resolution</td><td>Exact match</td></tr><tr><td>aspect_ratio / fps</td><td>Exact match</td></tr><tr><td>generateAudio</td><td>Exact match</td></tr><tr><td>mode / quality (e.g. <code>std</code>, <code>pro</code>)</td><td>Exact match</td></tr><tr><td>seed</td><td>Exact match if provided</td></tr><tr><td>cfg_scale / motion_strength</td><td>Exact match</td></tr><tr><td>Reference image / first-frame / last-frame</td><td>Hashed by content</td></tr><tr><td>camera_control</td><td>Exact match (serialized)</td></tr></tbody></table>

Ignored for cache hashing: `callback_url` / webhook, `user`, tracing headers (`traceparent`, `x-span-id`, `x-span-name`)

**Responses**

`POST /api/v1/videos` — **Cache MISS**:

```json
{
  "chat_id": "fr_c513522e-983c-4e99-8af0-adabf9b71ed0",
  "model": "bytedance/seedance-1-5-pro",
  "code": "success",
  "message": "Video generation started",
  "data": {
    "taskId": "fr_01M1H4T9E9JC0VAQV11EBDQHFW",
    "status": "processing"
  },
  "estimate_usage": {
    "chat_id": "fr_c513522e-983c-4e99-8af0-adabf9b71ed0",
    "prompt_tokens": 0,
    "completion_tokens": 0,
    "total_tokens": 0,
    "user_key_credits_used": 0.192,
    "credits_used": 0.192,
    "provider": "pollo"
  }
}
```

`POST /api/v1/videos` — **Cache HIT**:

```json
{
  "chat_id": "fr_2e52ff0f-8bb2-4044-94a7-b722f3a3ee14",
  "model": "kling-ai/kling-v1-6",
  "code": "success",
  "message": "Video generation started",
  "data": {
    "taskId": "fr_01M1NH55R9NM02M56D57P2F5FY",
    "status": "processing"
  },
  "estimate_usage": {
    "chat_id": "fr_2e52ff0f-8bb2-4044-94a7-b722f3a3ee14",
    "prompt_tokens": 0,
    "completion_tokens": 0,
    "total_tokens": 0,
    "provider": ""
  }
}
```

The `taskId` is the one from the original generation. `status` is reported as `processing`; the client polls as usual and the task resolves on the first poll if the underlying job has already completed.

`POST /api/v1/getAsyncResponse`:

```bash
curl --location 'https://api.fastrouter.ai/api/v1/getAsyncResponse' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'Content-Type: application/json' \
  --data '{ "taskId": "fr_01M1NH55R9NM02M56D57P2F5FY" }'
```

```json
{
  "chat_id": "fr_2cefa70a-cf24-4ced-bd62-8c2a35940557",
  "code": "SUCCESS",
  "message": "success",
  "data": {
    "costUsd": 0.21,
    "credit": 3.5,
    "generations": [
      {
        "createdDate": "2026-09-04T06:19:36.000Z",
        "failMsg": "",
        "id": "cmtmkdhv20eg7r3ouc2q2nz00",
        "mediaType": "video",
        "status": "succeed",
        "updatedDate": "2026-09-04T06:23:08.000Z",
        "url": "https://videocdn.pollo.ai/web-cdn/pollo/production/.../1788502985556-....mp4"
      }
    ],
    "status": "succeed",
    "taskId": "fr_01M1NH55R9NM02M56D57P2F5FY"
  },
  "fastrouter_assets": {
    "status": "ready",
    "urls": ["https://api.fastrouter.ai/videos-content/fr_01M1NH55R9NM02M56D57P2F5FY"],
    "expires_at": 1791095001,
    "cached_at": 1788503001
  },
  "usage": {
    "api_key_credits_used": 0,
    "user_key_credits_used": 0.21,
    "credits_used": 0.21,
    "provider": "pollo"
  }
}
```

The `taskId` / `model` rules for `getAsyncResponse` are unchanged: `fr_`-prefixed IDs require no `model`.

**Billing Timing**

* **MISS:** `estimate_usage` at submission; final `usage` billed at standard price when the job **completes**. Failed jobs are not billed.
* **HIT:** not billed. `estimate_usage.credits_used` is `0` on the generation response, and no charge is applied when the shared job completes. The `usage` returned by `getAsyncResponse` reflects the original generation, not a charge to the hitting request.

***

## Cache Lookup

**Cache Lookup Components**

```
  org_id,
  cache_key,          // header
  endpoint,           // chat / images / videos — entries never cross modalities
  model?,             // canonical slug, if filter_on_model = true
  provider?,          // if filter_on_provider = true
  prompt_input,       // messages (text) or prompt (media) — semantic-eligible
  exact_params        // modality-specific parameter set listed above
```

> `similarity_threshold` is applied **after lookup** to determine semantic eligibility. Only `prompt_input` is semantically matched; `exact_params` must hash identically.

**Cross-Modality Isolation**

The endpoint is part of the key. A `cache_key` of `myapp` used on `/v1/chat/completions` and `/api/v1/images/generations` produces two independent namespaces. This lets teams reuse one namespace string across their stack without risk of collision.

***

## Cache Response Fields

| Field                  | Type    | Modality    | Description                                                                 |
| ---------------------- | ------- | ----------- | --------------------------------------------------------------------------- |
| cached                 | boolean | Text        | True when served from cache                                                 |
| similarity             | number  | Text        | Semantic similarity score (1 = exact match). Present on hits                |
| task\_id / data.taskId | string  | Image/Video | Same ID as the original request on hits                                     |
| chat\_id               | string  | Image/Video | New per request, including hits (for Activity Log and billing)              |
| estimate\_usage        | object  | Image/Video | `credits_used: 0` and `provider: ""` indicate a hit                         |
| fastrouter\_assets     | object  | Image/Video | Hosted asset URLs, `cached_at`, `expires_at`. Authoritative download source |
| usage.credits\_used    | number  | All         | Text hits billed at 0.1×; image/video hits are not charged                  |

***

## Pricing

**Cache Pricing**

| Scenario      | Text             | Image          | Video          |
| ------------- | ---------------- | -------------- | -------------- |
| Cache HIT     | 0.1× token price | Free           | Free           |
| Cache MISS    | Standard price   | Standard price | Standard price |
| Cache Storage | Free             | Free           | Free           |

**Pricing Formulas**

```
text_hit_cost  = (prompt_tokens × input_price × 0.1) + (completion_tokens × output_price × 0.1)
image_hit_cost = 0
video_hit_cost = 0
```

**Savings:** \~90% on text; 100% on image and video hits.

***

## Limits

| Limit                      | Value                               |
| -------------------------- | ----------------------------------- |
| Max TTL — text             | 86,400 s (24h)                      |
| Max TTL — image/video      | 604,800 s (7 days)                  |
| Default TTL — image/video  | 57,600 s (16h)                      |
| Hosted asset retention     | Up to 30 days depending on plan     |
| Pending-job safety timeout | 30 min (provider-specific override) |
| Max `cache_key` length     | 128 chars, `[A-Za-z0-9_\-:.]`       |

***

## Roadmap

| Item                                                     | Status  |
| -------------------------------------------------------- | ------- |
| Sync image caching                                       | Planned |
| `cached` / `similarity` / `cache_age` on media responses | Planned |

***
