---
description: >-
  FastRouter lets you restrict any request to Zero Data Retention (ZDR)
  providers by adding the :zdr suffix to the model slug.
icon: empty-set
---

# Zero Data Retention

## Introduction

Some workloads can't have prompts or completions stored by a model provider, not even temporarily for logging, abuse monitoring, or training. FastRouter lets you restrict any request to **Zero Data Retention (ZDR)** providers by adding the `:zdr` suffix to the model slug.

When you use `:zdr`, FastRouter routes your request **only** to providers that have committed to not retaining your request or response data. If no ZDR provider can serve the model, the request fails instead of silently falling back to a provider that retains data.

***

## How it works

Append `:zdr` to any model ID:

```
<provider>/<model>:zdr
```

For example:

| Standard slug                  | ZDR slug                           |
| ------------------------------ | ---------------------------------- |
| `deepseek/deepseek-v4.1-flash` | `deepseek/deepseek-v4.1-flash:zdr` |

With the suffix in place, FastRouter:

1. Filters the model's available providers down to ZDR-compliant providers only.
2. Routes the request among those providers using your normal routing preferences (for example, `provider.sort`).
3. Returns an error if no ZDR-compliant provider is available for that model after all filters are applied.

No other changes to your request are needed. The request and response formats are identical to a standard chat completion.

***

## Supported providers

The following providers are currently treated as Zero Data Retention providers on FastRouter:

| Provider   |
| ---------- |
| Fireworks  |
| Baseten    |
| Groq       |
| Perplexity |
| DeepInfra  |
| Together   |

A model can be used with `:zdr` only if at least one of these providers serves it on FastRouter.

***

## Example request

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Authorization: Bearer KEY' \
--header 'Content-Type: application/json' \
--data '{
    "model": "deepseek/deepseek-v4.1-flash:zdr",
    "stream": false,
    "max_completion_tokens": 200,
    "messages": [
        {
            "role": "user",
            "content": "What is 2+2"
        }
    ]
}'
```

## Example response

```json
{
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "2 + 2 = 4",
                "annotations": null
            },
            "finish_reason": "stop",
            "index": 0
        }
    ],
    "guardrails": {},
    "citations": null,
    "usage": {
        "chat_id": "fr_768d0989-2830-4f47-af04-00286e7e119c",
        "completion_tokens": 8,
        "object": "",
        "completion_tokens_details": {},
        "prompt_tokens": 10,
        "cost": 0.000006799999999999999,
        "prompt_tokens_details": {
            "cached_tokens": 0
        },
        "total_tokens": 18,
        "provider": "deepinfra"
    },
    "id": "fr_768d0989-2830-4f47-af04-00286e7e119c",
    "model": "deepseek-ai/DeepSeek-V4.1-Flash",
    "object": "chat.completion",
    "service_tier": "default"
}
```

The `usage.provider` field tells you which provider served the request. For a `:zdr` request, this will always be one of the supported ZDR providers listed above. In this example, the request was served by **DeepInfra**.

***

## Combining `:zdr` with provider routing

`:zdr` works alongside the `provider` object. ZDR filtering is applied first, and your provider preferences are then applied to the remaining ZDR providers.

For example, to route to the highest-throughput ZDR provider:

```json
{
    "model": "deepseek/deepseek-v4.1-flash:zdr",
    "messages": [
        { "role": "user", "content": "What is 2+2" }
    ],
    "provider": {
        "sort": "throughput"
    }
}
```

If you use `provider.only` to pin specific providers, include only ZDR providers. If none of the providers left after filtering are ZDR-compliant, the request is rejected.

***

## Error handling

If no ZDR-compliant provider can serve the request, FastRouter returns an error rather than routing to a non-ZDR provider.

#### Example: no ZDR provider available

Request:

```json
{
    "model": "deepseek/deepseek-v4.1-flash:zdr",
    "stream": false,
    "max_completion_tokens": 200,
    "messages": [
        {
            "role": "user",
            "content": "What is 2+2"
        }
    ],
    "provider": {
        "sort": "throughput",
        "only": [
            "non-zdr-provider"
        ]
    }
}
```

Response:

```json
{
    "error": "deepseek/deepseek-v4.1-flash is not a valid model ID or no providers found . No provider for this model supports Zero Data Retention (:zdr)"
}
```

Here, `provider.only` limits routing to a provider that isn't ZDR-compliant, so no eligible provider is left.

This error can occur when:

* The model isn't served by any of the supported ZDR providers.
* Your `provider.only` list (or other provider filters) excludes every ZDR provider serving the model.
* The model ID itself is invalid.

**To resolve it**, remove or adjust the conflicting provider filters, choose a model served by at least one ZDR provider, or check the model ID for typos.

***

## FAQ

**Does `:zdr` change the response format?** No. Requests and responses follow the standard chat completions format. The only visible difference is that `usage.provider` will always be a ZDR provider.

**Will FastRouter ever fall back to a non-ZDR provider?** No. When `:zdr` is set, requests are routed only to ZDR-compliant providers. If none are available, the request fails with an error.

**How do I confirm which provider handled my request?** Check the `usage.provider` field in the response.

**Can I use `:zdr` with every model?** Only with models served by at least one supported ZDR provider. For other models, the request returns the error shown above.
