---
description: >-
  FastRouter exposes select models at no charge via the :free slug — up to 10
  requests per org per day, with no payment required.
icon: square-0
---

# Free Model Router (:free)

### Introduction

FastRouter provides free access to a rotating set of models for all orgs, regardless of billing status. Free access is provided at FastRouter's discretion; availability of any model can be paused or removed at any time.

You can access free models in two ways:

1. **Use a specific free model** by appending `:free` to its model ID.
2. **Use `fastrouter/free`** to let FastRouter automatically select and rotate between available free models.

***

### How It Works

**Option 1: Use a specific free model**

Append `:free` to any supported model ID:

```
openai/gpt-oss-120b:free
```

FastRouter strips the suffix, checks whether `:free` is enabled for that model, verifies your org's daily quota, then routes the request normally. The `:free` suffix is invisible to the downstream provider.

If a model does not have `:free` enabled, the request is rejected. The standard model ID (without the suffix) is unaffected.

**Option 2: Use `fastrouter/free`**

You can use `fastrouter/free` as the model ID to have FastRouter automatically select a free model for each request:

```
fastrouter/free
```

The **FastRouter Free Model Router** automatically rotates between available free models. For each request, FastRouter selects a free model at random from the currently available pool, while intelligently filtering the pool based on the capabilities required by your request.

This allows you to use a single model ID without having to maintain a list of individual free models as availability changes.

> **Note:** The 10-request daily quota applies to each underlying free model. Requests made through `fastrouter/free` consume the quota of the free model selected for that request.

***

### Supported Models

`:free` is currently available on the following models:

| Model                         | Free Model ID                     |
| ----------------------------- | --------------------------------- |
| OpenAI: GPT-OSS 120B          | `openai/gpt-oss-120b:free`        |
| OpenAI: GPT-OSS 20B           | `openai/gpt-oss-20b:free`         |
| Google: Gemma 4 26B           | `google/gemma4-26b:free`          |
| NVIDIA: Nemotron 3 Nano 30B   | `nvidia/nemotron-3-nano-30b:free` |
| NVIDIA: Nemotron 3 Super 120B | `nvidia/nemotron-3-super:free`    |
| Sarvam: Sarvam 105B           | `sarvam/sarvam-105b:free`         |

`:free` is enabled on a per-model basis. Check the [model catalog](https://fastrouter.ai/models?order=newest) — eligible models display a **Free** badge on their detail page.

***

### Usage

**Using `fastrouter/free`**

**cURL**

```
curl 'https://api.fastrouter.ai/api/v1/chat/completions' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "fastrouter/free",
    "messages": [
      { "role": "user", "content": "Explain backpressure in streaming systems." }
    ]
  }'
```

**Python (OpenAI SDK)**

```
from openai import OpenAI

client = OpenAI(
    base_url="https://api.fastrouter.ai/api/v1",
    api_key="YOUR_API_KEY",
)

response = client.chat.completions.create(
    model="fastrouter/free",
    messages=[
        {"role": "user", "content": "Explain backpressure in streaming systems."}
    ],
)

print(response.choices[0].message.content)
```

**Using a specific free model**

You can also target a specific free model directly:

```
openai/gpt-oss-120b:free
```

This bypasses the free model router and sends the request to the specified free model.

***

### Quota & Limits

<table data-header-hidden data-search="false"><thead><tr><th>Property</th><th>Value</th></tr></thead><tbody><tr><td>Requests per org per day</td><td>10 per free model. <strong>This limit may vary or change periodically.</strong></td></tr><tr><td>Scope</td><td>Per underlying model — quota tracked independently for each model</td></tr><tr><td>Reset</td><td>Daily at UTC midnight</td></tr><tr><td>Carry-over</td><td>None — unused requests do not roll over</td></tr><tr><td>Paid org behaviour</td><td>Consumes free quota, not billing credits</td></tr><tr><td><code>fastrouter/free</code></td><td>Automatically selects from currently available free models</td></tr></tbody></table>

> **Example:** If your org uses `fastrouter/free`, each request is routed to one of the currently available free models. If 10 requests have already been made to a particular underlying model, that model is excluded from further free routing for the remainder of the day. This limit may vary or change periodically.

***

#### Error Responses

**`:free` not enabled on this model — `400`**

Returned when `:free` is used on a model that does not have the slug enabled.

```
{
  "error": {
    "code": "free_slug_not_enabled",
    "message": "The :free slug is not available for this model. Use the standard model ID or check the model catalog for supported free models.",
    "type": "invalid_request_error"
  }
}
```

**Daily quota exhausted — `429`**

Returned when your org has exhausted the free quota for a model. The response includes a `Retry-After` header pointing to the next UTC midnight reset.

```
{
  "error": {
    "code": "free_quota_exceeded",
    "message": "Your organisation has reached the daily free request limit for this model.",
    "type": "rate_limit_error",
    "quota_limit": 10,
    "quota_used": 10,
    "reset_at": "2026-05-30T00:00:00Z"
  }
}
```

When using a specific `:free` model, remove the `:free` suffix to continue as a standard paid request.

When using `fastrouter/free`, FastRouter automatically excludes models whose free quota has been exhausted and selects from the remaining eligible free models.

***

#### Activity Log

All `:free` requests appear in your Activity Log tagged with a **Free** tier indicator. Cost is recorded as `$0.00`. Usage analytics include free-tier traffic separately so it does not skew your paid consumption metrics.

Requests made through `fastrouter/free` also show the underlying model selected for the request.

***

#### FAQ

**What is `fastrouter/free`?**\
`fastrouter/free` is FastRouter's free model router. It automatically selects and rotates between currently available free models, so you can use a single model ID instead of specifying an individual free model.

**How does `fastrouter/free` choose a model?**\
FastRouter randomly selects from the currently available free models after filtering for the capabilities required by your request, such as image understanding, tool calling, and structured outputs.

**Can I choose a specific free model?**\
Yes. Append `:free` to an eligible model ID, such as `openai/gpt-oss-120b:free`.

**Is the 10-request limit shared across all free models, or per model?**\
The limit is **per model**. Your org gets up to 10 free requests per eligible model per day. `fastrouter/free` distributes requests across the available free models. **Note: This limit may vary or change periodically.**

**What happens when a free model's quota is exhausted?**\
That model is excluded from `fastrouter/free` routing until the daily quota resets. Other eligible free models remain available.

**What happens if no free models are available?**\
If all eligible free models are unavailable or your org has exhausted the free quota across the available pool, `fastrouter/free` cannot route the request and returns an appropriate error.

**Does `fastrouter/free` support all API parameters?**\
Yes — requests are filtered based on the capabilities required by the request. Parameters such as structured outputs, tool use, streaming, and multimodal inputs are routed only to compatible free models.

**Can I combine `:free` with `:flex`?**\
No. `:free` and `:flex` are mutually exclusive suffixes.

**Do free requests consume my billing credits?**\
No. Requests made using `:free` or `fastrouter/free` consume the free quota and are not charged against your billing credits.

**Can paid orgs use free models?**\
Yes. Paid and unpaid orgs can use eligible free models. Free requests remain subject to the applicable daily quota.

**Where can I find currently available free models?**\
Check the [model catalog](https://fastrouter.ai/models?order=newest). Models currently eligible for free access display a **Free** badge.
