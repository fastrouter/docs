---
description: >-
  FastRouter exposes select models at no charge via the :free slug — up to 10
  requests per org per day, with no payment required.
icon: square-0
---

# Free Models (:free)

### Introduction

This feature is available to all orgs regardless of billing status. Provided at FastRouter's discretion; free access on any given model can be paused or removed at any time.

***

### How It Works

Append `:free` to any supported model ID:

```
sarvam/sarvam-105b:free
```

FastRouter strips the suffix, checks whether `:free` is enabled for that model, verifies your org's daily quota, then routes the request normally. The `:free` suffix is invisible to the downstream provider.

If a model does not have `:free` enabled, the request is rejected. The standard model ID (without the suffix) is unaffected.

> **Note:** The 10-request daily quota is org-wide and shared across all API keys and members. Paid orgs using `:free` consume from this free quota — not from billing credits.

***

### Supported Models

Currently, supported on Sarvam models in our catalog for a limited time.

| Model               | Free Model ID             |
| ------------------- | ------------------------- |
| Sarvam: Sarvam 105B | `sarvam/sarvam-105b:free` |
| Sarvam: Sarvam 30B  | `sarvam/sarvam-30b:free`  |
| Sarvam: Saaras V3   | `sarvam/saaras:v3:free`   |
| Sarvam: Bulbul V2   | `sarvam/bulbul:v2:free`   |

`:free` is enabled on a per-model basis. Check the [model catalog](https://fastrouter.ai/models?order=newest) — eligible models display a **Free** badge on their detail page.

***

### Usage

#### cURL

```bash
curl 'https://api.fastrouter.ai/api/v1/chat/completions' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "sarvam/sarvam-105b:free",
    "messages": [
      { "role": "user", "content": "Explain backpressure in streaming systems." }
    ]
  }'
```

#### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.fastrouter.ai/api/v1",
    api_key="YOUR_API_KEY",
)

response = client.chat.completions.create(
    model="sarvam/sarvam-105b:free",
    messages=[
        {"role": "user", "content": "Explain backpressure in streaming systems."}
    ],
)

print(response.choices[0].message.content)
```

***

### Quota & Limits

| Property                 | Value                                                  |
| ------------------------ | ------------------------------------------------------ |
| Requests per org per day | 10                                                     |
| Scope                    | Per model — quota tracked independently per `model_id` |
| Reset                    | Daily at UTC midnight                                  |
| Carry-over               | None — unused requests do not roll over                |
| Paid org behaviour       | Consumes free quota, not billing credits               |

***

### Error Responses

#### `:free` not enabled on this model — `400`

Returned when `:free` is used on a model that does not have the slug enabled.

```json
{
  "error": {
    "code": "free_slug_not_enabled",
    "message": "The :free slug is not available for this model. Use the standard model ID or check the model catalog for supported free models.",
    "type": "invalid_request_error"
  }
}
```

#### Daily quota exhausted — `429`

Returned when your org has used all 10 free requests for the day on this model. The response includes a `Retry-After` header pointing to the next UTC midnight reset.

```json
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

To continue without waiting for the reset, remove the `:free` suffix to route as a standard paid request.

***

### Activity Log

All `:free` requests appear in your [Activity Log](https://claude.ai/dashboard/activity) tagged with a **Free** tier indicator. Cost is recorded as `$0.00`. Usage analytics include free-tier traffic separately so it does not skew your paid consumption metrics.

***

### FAQ

**Does `:free` support all API parameters?** Yes — structured outputs, tool use, streaming, and all parameters supported by the underlying model work with `:free`.

**Can I combine `:free` with `:flex`?** No. `:free` and `:flex` are mutually exclusive suffixes.

**Is the 10-request limit shared across all free models, or per model?** Per model. Each model has its own independent quota counter, so using `sarvam/sarvam-105b:free` does not consume quota for any other `:free` model.

**What happens to a request if `:free` access is removed from a model?** It returns a `400` error, the same as using `:free` on a model that never had it enabled. The standard model ID continues to work normally.
