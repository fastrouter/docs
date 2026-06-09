---
description: >-
  Access OpenAI and Google Gemini models at up to 50% lower cost by opting into
  flexible inference — ideal for background tasks, batch workloads, and
  latency-tolerant applications.
icon: shuffle
---

# Flex Pricing

> 💡 **Instant savings, zero code changes** Append `:flex` to any supported model ID. Your API key, endpoint, and payload stay exactly the same.

### What is Flex Pricing?

Flex inference is a tiered pricing mode offered by OpenAI and Google on select models. When you use the `:flex` suffix, FastRouter routes your request to the provider's Flex tier — significantly reducing token costs in exchange for variable throughput and potentially higher latency under peak load.

Flex is well-suited for workloads that don't require guaranteed low latency: data extraction pipelines, classification jobs, offline summarisation, evals runs, and other async or background tasks.

> ⚠️ **Not recommended for real-time user-facing responses** Flex requests may experience higher tail latencies during peak provider load. Use the standard tier for interactive or streaming use-cases.

### Pricing Comparison

Example using **GPT-5.4 Nano** via the OpenAI provider:

| Tier         | Input             | Output            | Blended           | Context |
| ------------ | ----------------- | ----------------- | ----------------- | ------- |
| **Standard** | $0.20 / 1M tokens | $1.25 / 1M tokens | $0.46 / 1M tokens | 400,000 |
| **✦ Flex**   | $0.10 / 1M tokens | $0.63 / 1M tokens | —                 | 400,000 |

**↓ \~50% savings on tokens**

> Actual savings vary by model. Check the [model catalog](https://chat.fastrouter.ai/compare-playground/chat/dee2b9cb-660f-4ceb-a879-8d968b8a88c1) for per-model Flex pricing across all supported providers.

### Supported Providers

Flex pricing is currently available on the following providers:

| Supported Provider                    |
| ------------------------------------- |
| OpenAI                                |
| Gemini API on Vertex AI and AI Studio |

### How to Use Flex

1. **Identify a Flex-supported model** Check the model catalog for the **Flex** tab in Provider Details. If the tab is present, Flex pricing is available for that model.
2. **Append `:flex` to the model ID** Change your model field from `openai/gpt-5.4-nano` to `openai/gpt-5.4-nano:flex` or from `google/gemini-3.1-pro-preview` to `google/gemini-3.1-pro-preview:flex` That's the only change required.
3. **Optionally pin the provider** Use `"provider": {"only": ["openai"]}`  or `"provider": {"only": ["googleaistudio"]}` or `"provider": {"only": ["googlevertexai"]}` to ensure the request goes to the correct provider for the Flex tier and isn't rerouted.

### Code Examples

{% tabs %}
{% tab title="cURL" %}
```bash
# ✦ With Flex — ~50% cheaper
curl 'https://api.fastrouter.ai/api/v1/chat/completions' \
  --header 'Authorization: Bearer YOUR_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "model": "openai/gpt-5.4-nano:flex",
    "provider": { "only": ["openai"] },
    "messages": [
      { "role": "user", "content": "Summarise this document..." }
    ]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.fastrouter.ai/api/v1",
    api_key="YOUR_API_KEY",
)

response = client.chat.completions.create(
    model="openai/gpt-5.4-nano:flex",
    extra_body={"provider": {"only": ["openai"]}},
    messages=[
        {"role": "user", "content": "Summarise this document..."}
    ],
)

print(response.choices[0].message.content)

```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.fastrouter.ai/api/v1",
  apiKey: process.env.FASTROUTER_API_KEY,
});

const response = await client.chat.completions.create({
  model: "openai/gpt-5.4-nano:flex",
  // @ts-expect-error - FastRouter routing extension
  provider: { only: ["openai"] },
  messages: [
    { role: "user", content: "Summarise this document..." },
  ],
});

console.log(response.choices[0].message.content);

```
{% endtab %}
{% endtabs %}

### When to Use Flex

| Use Flex ✦                                 | Use Standard                     |
| ------------------------------------------ | -------------------------------- |
| Data extraction & classification pipelines | Real-time chat & interactive UIs |
| Batch document summarisation               | Streaming responses to end users |
| Eval or fine-tuning dataset generation     | Latency-sensitive agent loops    |
| Scheduled background jobs                  | Voice or real-time applications  |
| Cost-optimised preprocessing at scale      | SLA-bound enterprise workflows   |
