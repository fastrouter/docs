---
description: >-
  This page covers caching for asynchronous image generation (POST
  /api/v1/images/generations) and video generation (POST /api/v1/videos). For
  text completions, see Response Caching: Text.
icon: image-circle-arrow-down
---

# Response Caching: Images & Video (Async)

## Overview

For images and videos, FastRouter caches the **task**, not just the payload. When a request matches a cached entry, FastRouter returns the **same task ID** as the original request along with `cached: true`, and the client retrieves the result through the normal `getAsyncResponse` polling flow. No new provider job is created, and the request is not billed.

| Modality             | Generate                          | Retrieve                                                          | Caching status                      |
| -------------------- | --------------------------------- | ----------------------------------------------------------------- | ----------------------------------- |
| Image (async)        | `POST /api/v1/images/generations` | `POST /api/v1/getAsyncResponse` or `GET /api/v1/images/{task_id}` | **Supported**                       |
| Video (always async) | `POST /api/v1/videos`             | `POST /api/v1/getAsyncResponse`                                   | **Supported**                       |
| Image (sync)         | `POST /api/v1/images/generations` | —                                                                 | **Not yet supported — coming soon** |

Caching is especially effective for product catalogs, thumbnails and templated creatives (images), and for marketing loops, previews and repeated demo prompts (videos).

***

## Key Benefits

| Benefit               | Description                                                                       |
| --------------------- | --------------------------------------------------------------------------------- |
| Faster Results        | A hit returns an existing task ID, skipping the full generation wait entirely     |
| Cost Reduction        | Image and video cache hits are **free** — the hitting request is not billed       |
| Consistent Outputs    | Identical prompts return the identical asset                                      |
| Reduced Provider Load | Fewer upstream jobs queued at Leonardo, Pollo, and other providers                |
| Explicit Hit Signal   | `cached: true` on the creation response tells clients the task ID came from cache |
| Custom Cache Keys     | User-defined namespaces for precise cache control                                 |

***

## Eligibility

| Request                                                                                     | Cached?                                          |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| `POST /api/v1/videos` (always async)                                                        | Yes                                              |
| `POST /api/v1/images/generations` with `response_type: "async"`                             | Yes                                              |
| `POST /api/v1/images/generations` on an async-only model (e.g. Leonardo, Flux via Leonardo) | Yes — routed async regardless of `response_type` |
| `POST /api/v1/images/generations` processed synchronously                                   | **No** — cache is bypassed (no lookup, no store) |

A sync-processed request neither reads from nor writes to the cache, so it will not populate an entry that a later async request could hit. Sync image caching is planned.

***

## Feature Specification

Caching is enabled by including a `cache_key` header and an optional `cache` configuration object in the request body. The contract is identical for images and videos.

#### Headers

| Header        | Type   | Required          | Description                                                   |
| ------------- | ------ | ----------------- | ------------------------------------------------------------- |
| Authorization | string | Yes               | Bearer token with API key                                     |
| Content-Type  | string | Yes               | `application/json`                                            |
| cache\_key    | string | Yes (for caching) | User-defined cache namespace. If omitted, caching is disabled |

Tracing headers (`traceparent`, `x-span-id`, `x-span-name`) are passed through and do not affect the cache key.

#### Cache Key Header

The `cache_key` header defines the **primary cache namespace** — for example `product-thumbnails`, `marketing-loops`, or `imagegen1`. FastRouter combines it with hashed request attributes to form the final lookup key.

The endpoint is part of the key, so the same `cache_key` used on `/api/v1/images/generations` and `/api/v1/videos` produces two independent namespaces.

#### Cache Object Parameters

| Parameter             | Type    | Default     | Required | Description                                                            |
| --------------------- | ------- | ----------- | -------- | ---------------------------------------------------------------------- |
| expiration\_time      | integer | 57600 (16h) | No       | Cache TTL in seconds (60–604800)                                       |
| filter\_on\_model     | boolean | true        | No       | Match cache on model name                                              |
| filter\_on\_provider  | boolean | false       | No       | Match cache on provider                                                |
| similarity\_threshold | number  | 1.0         | No       | Semantic similarity score (0–1) on the prompt required to reuse a task |

`conversation_mode` and `last_n_turns` are text-only and are ignored here.

#### 🔍 `similarity_threshold` Explained

Semantic matching is applied to the **prompt text only**. All other generation parameters (size, length, resolution, reference media, etc.) must match exactly regardless of threshold.

| Value           | Behavior                |
| --------------- | ----------------------- |
| `1.0` (default) | Exact prompt match only |

> **Why exact match by default for media?** "A red car on a beach" and "a crimson car by the sea" may embed at 0.9 similarity, but a user rarely wants the _same_ image for both. Lower the threshold deliberately for cases like FAQ illustrations or placeholder assets, where interchangeability is acceptable.

***

## How a Cache Hit Behaves

**Detecting a hit.** The creation response carries a top-level `cached: true` when the returned task ID came from cache. The field is absent on a miss. This is the field clients should branch on.

**Task ID semantics.**

* A hit returns the **original task ID** (`fr_`-prefixed) — `task_id` for images, `data.taskId` for videos. It is a real, pollable task; `getAsyncResponse` with that ID works regardless of which request created it.
* Each hitting request still gets its **own `chat_id`**, so it appears in the Activity Log as its own row.
* Task IDs are org-scoped; a task ID is never returned to a different organization.

**Billing.** A hit is not charged. `estimate_usage.credits_used` is `0` on an image hit; on a video hit the credit fields are omitted from `estimate_usage` entirely. `estimate_usage.provider` still names the provider that produced the original asset.

**Polling.** `getAsyncResponse` returns the **original task's** record — the same `data`, the same `fastrouter_assets`, and the `usage` of the generation that actually ran. It does **not** carry `cached`, and its `usage` is not zeroed. Poll responses are not re-billed; measure cache savings from the creation response or the Activity Log, not from the polling response.

**Assets.** `getAsyncResponse` returns both the provider `data[].url` and `fastrouter_assets.urls`. Treat `fastrouter_assets.urls` as authoritative — signed provider URLs (Leonardo CDN, Pollo CDN, TOS-signed links) expire well before the hosted copy does.

`fastrouter_assets.cached_at` / `expires_at` describe **asset hosting retention** (up to 30 days depending on plan), which is independent of the response-cache TTL set by `expiration_time`. The cache TTL governs whether a _new request_ is matched to an existing task; asset retention governs how long the hosted URL stays downloadable.

***

## Cache Lookup

```
  org_id,
  cache_key,          // header
  endpoint,           // images / videos — entries never cross modalities
  model?,             // canonical slug, if filter_on_model = true
  provider?,          // if filter_on_provider = true
  prompt,             // semantic-eligible, per similarity_threshold
  exact_params        // per-modality parameter set below
```

FastRouter normalizes to its **canonical model slug** for lookup — `bytedance/seedance-1.5-pro` and `bytedance/seedance-1-5-pro` hash identically. With `filter_on_provider: false` (the default), the same model routed through different providers shares one entry.

***

## Image Caching (Async)

#### Sample Request

```bash
curl --location 'https://api.fastrouter.ai/api/v1/images/generations' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'cache_key: imagegen1' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "leonardo-ai/phoenix",
    "n": 1,
    "prompt": "a clean pink sky",
    "cache": {
      "expiration_time": 57600,
      "filter_on_model": true,
      "filter_on_provider": false
    }
  }'
```

To force async routing on a model that also has a sync provider, add `"response_type": "async"`.

#### Parameter Sensitivity

Always included in the cache key:

<table data-search="false"><thead><tr><th>Parameter</th><th>Notes</th></tr></thead><tbody><tr><td>prompt</td><td>Subject to <code>similarity_threshold</code></td></tr><tr><td>n</td><td>Requesting 2 images ≠ requesting 4. See "n handling" below</td></tr><tr><td>size</td><td>Exact match. <code>auto</code> is resolved to the model default before hashing</td></tr><tr><td>quality</td><td>Exact match. <code>auto</code> resolved before hashing</td></tr><tr><td>style</td><td>Exact match</td></tr><tr><td>background</td><td>Exact match</td></tr><tr><td>output_format</td><td>Exact match (changes the bytes produced)</td></tr><tr><td>seed</td><td>Exact match if provided. Omitted seed and explicit seed are distinct keys</td></tr><tr><td>Reference / input images</td><td>Hashed by content (SHA-256 of bytes, or of the fetched URL body)</td></tr><tr><td>mask (edits)</td><td>Hashed by content</td></tr></tbody></table>

Ignored for cache hashing: `response_format`, `user`, tracing headers.

**`n` handling:** A cached entry for `n=4` is _not_ reused for `n=1`. Keeping `n` in the key avoids ambiguity over which subset to return.

**`response_format` handling:** Not hashed. The cached asset is served as `url` or `b64_json` per the hitting request.

**`response_type` handling:** Not hashed, but it determines eligibility — only async-processed requests participate in the cache.

#### Creation Response — Cache MISS

```json
{
  "created": 1788789917,
  "task_id": "fr_01M1Y301NZXA1JBPA4CGQJW02H",
  "chat_id": "fr_a4764b7a-ed54-4970-ae0f-9df6f772fe0f",
  "model": "phoenix",
  "data": [
    { "revised_prompt": "a clean pink sky" }
  ],
  "estimate_usage": {
    "chat_id": "fr_a4764b7a-ed54-4970-ae0f-9df6f772fe0f",
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

#### Creation Response — Cache HIT

An identical request sent 62 seconds later:

```json
{
  "created": 1788789979,
  "task_id": "fr_01M1Y301NZXA1JBPA4CGQJW02H",
  "chat_id": "fr_c518a0dc-0164-4946-890a-1317df2787bb",
  "model": "phoenix",
  "data": [
    { "revised_prompt": "a clean pink sky" }
  ],
  "estimate_usage": {
    "chat_id": "fr_c518a0dc-0164-4946-890a-1317df2787bb",
    "queue_time": 0,
    "input_tokens": 0,
    "output_tokens": 0,
    "total_tokens": 0,
    "api_key_credits_used": 0,
    "user_key_credits_used": 0,
    "credits_used": 0,
    "provider": "leonardo"
  },
  "cached": true
}
```

`task_id` is identical to the first request's. `chat_id`, `created`, and `estimate_usage.chat_id` are new for this request, and `credits_used` is `0`.

#### Retrieval

```bash
curl --location 'https://api.fastrouter.ai/api/v1/getAsyncResponse' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'Content-Type: application/json' \
  --data '{ "taskId": "fr_01M1Y301NZXA1JBPA4CGQJW02H" }'
```

```json
{
  "chat_id": "fr_0901d142-1dc0-4cab-b261-8d7796195846",
  "created": 1788790006,
  "data": [
    {
      "id": "3d1c9346-81cd-40c1-93a2-cf41bf4393cd",
      "url": "https://cdn.leonardo.ai/users/{user}/generations/{generation}/segments/1:1:1/Phoenix_10_a_clean_pink_sky_0.jpg"
    }
  ],
  "fastrouter_assets": {
    "status": "ready",
    "urls": ["https://api.fastrouter.ai/images-content/fr_01M1Y301NZXA1JBPA4CGQJW02H"],
    "expires_at": 1791381937,
    "cached_at": 1788789937
  },
  "model": "phoenix",
  "size": "512x512",
  "usage": {
    "chat_id": "fr_0901d142-1dc0-4cab-b261-8d7796195846",
    "api_key_credits_used": 0,
    "user_key_credits_used": 0.0135,
    "credits_used": 0.0135,
    "provider": "leonardo"
  }
}
```

Polling the shared task ID returns byte-identical `data` and `fastrouter_assets` for every caller. `usage` reports the original generation's cost (`0.0135`) — this is a record of what the asset cost to produce, not a charge against the hitting request.

***

## Video Caching

Video generation is always asynchronous: `POST /api/v1/videos` returns `data.taskId`, and the client polls `POST /api/v1/getAsyncResponse` until `data.status` is `succeed` / `completed`.

#### Sample Request

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
    "prompt": "A tennis match between 2 people in a sunny outdoor space",
    "mode": "std",
    "cache": {
      "expiration_time": 57600,
      "filter_on_model": true,
      "filter_on_provider": false
    }
  }'
```

#### Parameter Sensitivity

Always included in the cache key:

<table data-search="false"><thead><tr><th>Parameter</th><th>Notes</th></tr></thead><tbody><tr><td>prompt</td><td>Subject to <code>similarity_threshold</code></td></tr><tr><td>negative_prompt</td><td>Exact match</td></tr><tr><td>length</td><td>Exact match</td></tr><tr><td>resolution</td><td>Exact match</td></tr><tr><td>aspect_ratio / fps</td><td>Exact match</td></tr><tr><td>generateAudio</td><td>Exact match</td></tr><tr><td>mode / quality (e.g. <code>std</code>, <code>pro</code>)</td><td>Exact match</td></tr><tr><td>seed</td><td>Exact match if provided</td></tr><tr><td>cfg_scale / motion_strength</td><td>Exact match</td></tr><tr><td>Reference image / first-frame / last-frame</td><td>Hashed by content</td></tr><tr><td>camera_control</td><td>Exact match (serialized)</td></tr></tbody></table>

Ignored for cache hashing: `callback_url` / webhook, `user`, tracing headers (`traceparent`, `x-span-id`, `x-span-name`).

#### Creation Response — Cache MISS

```json
{
  "chat_id": "fr_abdbddc2-36f4-4b6f-b014-ecf60ed2cb9f",
  "model": "kling-ai/kling-v1-6",
  "code": "success",
  "message": "Video generation started",
  "data": {
    "taskId": "fr_01M1Y389KN0KN28AT3733Q505H",
    "status": "processing"
  },
  "estimate_usage": {
    "chat_id": "fr_abdbddc2-36f4-4b6f-b014-ecf60ed2cb9f",
    "prompt_tokens": 0,
    "completion_tokens": 0,
    "total_tokens": 0,
    "user_key_credits_used": 0.21,
    "credits_used": 0.21,
    "provider": "pollo"
  }
}
```

#### Creation Response — Cache HIT

```json
{
  "chat_id": "fr_de1d6850-50b7-4011-bec2-4a2b0b435182",
  "model": "kling-ai/kling-v1-6",
  "code": "success",
  "message": "Video generation started",
  "data": {
    "taskId": "fr_01M1Y389KN0KN28AT3733Q505H",
    "status": "processing"
  },
  "estimate_usage": {
    "chat_id": "fr_de1d6850-50b7-4011-bec2-4a2b0b435182",
    "prompt_tokens": 0,
    "completion_tokens": 0,
    "total_tokens": 0,
    "provider": "pollo"
  },
  "cached": true
}
```

`data.taskId` is identical to the first request's, and the credit fields are omitted from `estimate_usage` because nothing is charged. `cached` sits at the top level, alongside `data` — not inside it.

> `message` reads `"Video generation started"` and `data.status` reads `"processing"` on a hit as well. Branch on `cached`, not on the message or status, to tell a reused task from a new one.

#### Retrieval

```bash
curl --location 'https://api.fastrouter.ai/api/v1/getAsyncResponse' \
  --header 'Authorization: Bearer API-KEY' \
  --header 'Content-Type: application/json' \
  --data '{ "taskId": "fr_01M1Y389KN0KN28AT3733Q505H" }'
```

```json
{
  "chat_id": "fr_6ca8b68f-5282-4409-9a7b-7f999a42ece6",
  "code": "SUCCESS",
  "message": "success",
  "data": {
    "costUsd": 0.21,
    "credit": 3.5,
    "generations": [
      {
        "createdDate": "2026-09-07T14:07:31.000Z",
        "failMsg": "",
        "id": "cmtrbet3h0jizn3inp6tqsh74",
        "mediaType": "video",
        "status": "succeed",
        "updatedDate": "2026-09-07T14:11:01.000Z",
        "url": "https://videocdn.pollo.ai/web-cdn/pollo/production/{account}/ori/{asset}.mp4"
      }
    ],
    "status": "succeed",
    "taskId": "fr_01M1Y389KN0KN28AT3733Q505H"
  },
  "fastrouter_assets": {
    "status": "ready",
    "urls": ["https://api.fastrouter.ai/videos-content/fr_01M1Y389KN0KN28AT3733Q505H"],
    "expires_at": 1791382312,
    "cached_at": 1788790312
  },
  "message": "success",
  "usage": {
    "api_key_credits_used": 0,
    "user_key_credits_used": 0.21,
    "credits_used": 0.21,
    "provider": "pollo"
  }
}
```

Every caller polling this task ID receives the same `data`, `generations`, and `fastrouter_assets`. The `taskId` / `model` rules for `getAsyncResponse` are unchanged: `fr_`-prefixed IDs require no `model`.

***

## Response Fields

#### Creation Response

| Field                                  | Type    | Description                                                                         |
| -------------------------------------- | ------- | ----------------------------------------------------------------------------------- |
| cached                                 | boolean | Top-level. `true` when the task ID came from cache; absent on a miss                |
| task\_id (image) / data.taskId (video) | string  | The task to poll. Identical to the original request's on a hit                      |
| chat\_id                               | string  | New per request, including hits (for Activity Log and billing)                      |
| created                                | integer | Timestamp of this request                                                           |
| estimate\_usage                        | object  | Projected cost. `credits_used: 0` (image) or credit fields omitted (video) on a hit |

#### Retrieval Response (`getAsyncResponse`)

| Field              | Type         | Description                                                                 |
| ------------------ | ------------ | --------------------------------------------------------------------------- |
| data               | object/array | The generated asset record from the original task                           |
| fastrouter\_assets | object       | Hosted asset URLs, `cached_at`, `expires_at`. Authoritative download source |
| usage              | object       | The original generation's cost. Not a charge against this request           |

> `cached` is not returned on retrieval responses — only on the creation response.

***

## Billing

| Scenario   | Behavior                                                                                                                  |
| ---------- | ------------------------------------------------------------------------------------------------------------------------- |
| Cache MISS | `estimate_usage` at submission; final `usage` billed at standard price when the job completes. Failed jobs are not billed |
| Cache HIT  | **Not billed.** No credits are deducted at submission or at completion                                                    |
| Storage    | Free                                                                                                                      |

```
image_hit_cost = 0
video_hit_cost = 0
```

***

## Limits

| Limit                  | Value                           |
| ---------------------- | ------------------------------- |
| Min TTL                | 60 s                            |
| Default TTL            | 57,600 s (16h)                  |
| Max TTL                | 604,800 s (7 days)              |
| Hosted asset retention | Up to 30 days depending on plan |
| Max `cache_key` length | 128 chars, `[A-Za-z0-9_\-:.]`   |

***

## Roadmap

| Item                                      | Status  |
| ----------------------------------------- | ------- |
| Sync image caching                        | Planned |
| `similarity` score on media hit responses | Planned |
