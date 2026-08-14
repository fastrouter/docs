---
description: >-
  FastRouter exposes select models at no charge via the :free slug — up to 10
  requests per org per day, with no payment required for organizations having
  paid credit balance >$1.
icon: square-0
---

# Free Model Router (:free)

### Introduction

Free model access is provided at FastRouter's discretion. The available models, quotas, and eligibility requirements may change periodically, and access to any model may be paused or removed at any time.

> **Important:** Your organization must have a **positive credit balance** to use free models. Free models cannot be used when your organization's paid credit balance is < **$1**. Free requests themselves do not consume your credits.

You can use free models in two ways:

1. **Use a specific free model** by appending `:free` to its model ID.
2. **Use `fastrouter/free`** to let FastRouter automatically select an eligible free model for each request.

***

#### How It Works

**Option 1: Use a Specific Free Model**

Append `:free` to a supported model ID:

```bash
openai/gpt-oss-120b:free
```

FastRouter recognizes the `:free` suffix, verifies that free access is enabled for the model and that your organization meets the free-model eligibility requirements, then routes the request normally.

The `:free` suffix is handled entirely by FastRouter and is not passed to the downstream provider.

If `:free` is not enabled for a model, the request is rejected. The standard model ID without the `:free` suffix continues to work normally.

**Option 2: Use `fastrouter/free`**

Use `fastrouter/free` as the model ID to have FastRouter automatically select a free model:

```bash
fastrouter/free
```

The **FastRouter Free Model Router** automatically selects a free model from the currently available pool for each request.

FastRouter filters the available models based on the capabilities required by your request. For example, requests requiring **image understanding**, **tool calling**, or **structured outputs** are routed only to compatible free models.

This lets you use a single model ID without maintaining a list of individual free models as availability changes.

> **Note:** Requests made through `fastrouter/free` consume the free quota of the underlying model selected for that request.

***

#### Supported Models

The following models currently support `:free`:

| Model                         | Free Model ID                     |
| ----------------------------- | --------------------------------- |
| OpenAI: GPT-OSS 120B          | `openai/gpt-oss-120b:free`        |
| OpenAI: GPT-OSS 20B           | `openai/gpt-oss-20b:free`         |
| Google: Gemma 4 26B           | `google/gemma4-26b:free`          |
| NVIDIA: Nemotron 3 Nano 30B   | `nvidia/nemotron-3-nano-30b:free` |
| NVIDIA: Nemotron 3 Super 120B | `nvidia/nemotron-3-super:free`    |
| Sarvam: Sarvam 105B           | `sarvam/sarvam-105b:free`         |

`:free` is enabled on a per-model basis. Check the [model catalog](https://fastrouter.ai/models?order=newest) for the current list of eligible models. Models available for free access display a **Free** badge on their detail page.

***

#### Usage

**Using `fastrouter/free`**

**cURL**

```bash
curl 'https://api.fastrouter.ai/api/v1/chat/completions' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "fastrouter/free",
    "messages": [
      {
        "role": "user",
        "content": "Explain backpressure in streaming systems."
      }
    ]
  }'
```

**Python (OpenAI SDK)**

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.fastrouter.ai/api/v1",
    api_key="YOUR_API_KEY",
)

response = client.chat.completions.create(
    model="fastrouter/free",
    messages=[
        {
            "role": "user",
            "content": "Explain backpressure in streaming systems."
        }
    ],
)

print(response.choices[0].message.content)
```

**Using a Specific Free Model**

To use a specific free model instead of the router:

```
openai/gpt-oss-120b:free
```

This sends the request specifically to the selected free model and does not use the `fastrouter/free` router.

***

#### Quota & Limits

<table data-search="false"><thead><tr><th>Property</th><th>Value</th></tr></thead><tbody><tr><td>Credit balance requirement</td><td>Organization must have a positive credit balance</td></tr><tr><td>Free requests</td><td>10 per organization per free model per day</td></tr><tr><td>Quota scope</td><td>Tracked independently for each underlying model</td></tr><tr><td>Quota reset</td><td>Daily at UTC midnight</td></tr><tr><td>Carry-over</td><td>None — unused requests do not roll over</td></tr><tr><td>Request cost</td><td>$0.00</td></tr><tr><td>Credit consumption</td><td>Free requests do not consume billing credits</td></tr><tr><td><code>fastrouter/free</code></td><td>Automatically selects from currently available free models</td></tr></tbody></table>

> **Note:** The free request limit may vary or change periodically.

> **Important:** A positive credit balance is required even though free requests do not consume credits. An organization with < **$1 paid credit balance cannot use free models**.

For example, if your organization has $5 in credits, you can make free requests without reducing that $5 balance. If the balance reaches < $1, free-model requests are blocked until additional credits are added.

***

#### Error Responses

**`:free` Not Enabled — `400`**

Returned when `:free` is used with a model that does not have free access enabled.

```json
{
  "error": {
    "code": "free_slug_not_enabled",
    "message": "The :free slug is not available for this model. Use the standard model ID or check the model catalog for supported free models.",
    "type": "invalid_request_error"
  }
}
```

**Daily Free Quota Exhausted — `429`**

Returned when your organization has exhausted the free quota for the selected underlying model.

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

When using a specific `:free` model, you can remove the `:free` suffix to continue using the model as a standard paid request.

When using `fastrouter/free`, FastRouter excludes models whose free quota has been exhausted and selects from the remaining eligible models.

**No Credit Balance — `402`**

Returned when your organization's paid credit balance is < $1.

```json
{
  "error": {
    "code": "insufficient_credits",
    "message": "Your organisation must have a positive credit balance to use free models.",
    "type": "billing_error"
  }
}
```

Add credits to your organization to continue using free models.

***

#### Activity Log

Free-model requests appear in your Activity Log with a **Free** tier indicator.

The cost of free requests is recorded as **$0.00**. Free-tier traffic is tracked separately in usage analytics and does not contribute to paid consumption metrics.

For requests made through `fastrouter/free`, the Activity Log also shows the underlying model selected for the request.

***

#### FAQ

**What is `fastrouter/free`?**

`fastrouter/free` is FastRouter's free model router. It automatically selects an eligible free model for each request, allowing you to use a single model ID instead of specifying an individual free model.

**How does `fastrouter/free` choose a model?**

FastRouter filters the available free models based on the capabilities required by your request and randomly selects from the compatible models. This can include requirements such as image understanding, tool calling, and structured outputs.

**Can I choose a specific free model?**

Yes. Append `:free` to an eligible model ID, such as:

```shellscript
openai/gpt-oss-120b:free
```

**Do I need credits to use free models?**

Yes. Your organization must have a **positive credit balance**. Free models cannot be used when your organization's paid credit balance is < $1.

**Do free requests consume my billing credits?**

No. Free requests are charged **$0.00** and do not consume your billing credits. However, a positive credit balance > $1 is required to access the free-model service.

**Is the free request limit shared across all models?**

No. The quota is tracked **per underlying free model**. For example, if the limit is 10 requests per model per day, your organization can use up to 10 requests on each eligible free model, subject to the current limits.

> **Note:** Free-model quotas may vary or change periodically.

**What happens when a free model's quota is exhausted?**

That model is excluded from `fastrouter/free` routing until its quota resets. Other eligible free models can continue to be selected.

**What happens if my organization's paid credit balance reaches below $1?**

Free-model access is blocked. Add credits to your organization to continue using `:free` models or `fastrouter/free`.

**What happens if no free models are available?**

If no eligible free model can satisfy your request, or all available free models have exhausted their quotas, `fastrouter/free` cannot route the request and returns an appropriate error.

**Does `fastrouter/free` support all API parameters?**

FastRouter filters models based on the capabilities required by your request. Parameters such as structured outputs, tool use, streaming, and multimodal inputs are routed only to compatible free models.

**Can I combine `:free` with `:flex`?**

No. `:free` and `:flex` are mutually exclusive suffixes.

**Can paid organizations use free models?**

Yes. Both paid and unpaid organizations can use eligible free models, provided the organization has a positive credit balance.

**Where can I find the currently available free models?**

Check the [model catalog](https://fastrouter.ai/models?order=newest). Models currently eligible for free access display a **Free** badge.
