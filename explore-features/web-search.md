---
icon: magnifying-glass
---

# Web Search

## Introduction

FastRouter supports two ways to give a model access to live web results:

1. **Native web search** — for models that ship with built-in search, controlled via the `web_search_options` parameter.
2. **`:online` suffix** — appends Exa-powered web search results to _any_ model on FastRouter.

***

### Native Web Search

When using a web-search-enabled model, you can pass the `web_search_options` parameter to control how much search context is retrieved and processed. Models with this capability dynamically integrate search results into their reasoning process.

These models apply a per-request fee, and additionally charge based on search context size, which controls how much data is retrieved and processed per query.

#### Web-Search Enabled Models

These models support built-in web search:

* `openai/gpt-4o-mini-search-preview`
* `openai/gpt-4o-search-preview`
* `perplexity/sonar-pro`
* `perplexity/sonar-reasoning-pro`
* `perplexity/sonar`
* `perplexity/sonar-reasoning`

#### Search Context Size

The `search_context_size` setting controls how much information is pulled from search results. Pricing varies based on the selected level.

| Level    | Description                                        | Use Case                           |
| -------- | -------------------------------------------------- | ---------------------------------- |
| `low`    | Minimal context for basic queries                  | Quick facts, dates, headlines      |
| `medium` | Moderate context with broader information coverage | General knowledge, short summaries |
| `high`   | Extensive search context for deep research         | In-depth topics, analysis, reports |

#### Sample Request

This example uses `openai/gpt-4o-mini-search-preview` with medium search context to get real-time sports event info:

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer API-KEY' \
--data '{
  "model": "openai/gpt-4o-mini-search-preview",
  "messages": [
    {
      "role": "user",
      "content": "Which teams are playing the UEFA Champions League final?"
    }
  ],
  "stream": false,
  "top_p": 1,
  "temperature": 0,
  "max_completion_tokens": 120,
  "web_search_options": {
    "search_context_size": "medium"
  }
}'
```

***

### Enabling Web Search for Any Model (`:online`)

You can incorporate relevant web search results for _any_ model on FastRouter by appending **`:online`** to the model slug. Search is performed by **Exa**, and the retrieved results are injected into the model's context before the completion is generated — so this works with models that have no native search capability of their own.

```json
{
  "model": "openai/gpt-oss-20b:online"
}
```

#### Sample Request

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer API-KEY' \
--data '{
  "model": "openai/gpt-oss-20b:online",
  "messages": [
    {
      "role": "user",
      "content": "Which teams are playing the UEFA Champions League final?"
    }
  ],
  "stream": false
}'
```

***

### Pricing

Billing depends on which method you use.

| Method            | Search cost                                                 | Model cost                                      |
| ----------------- | ----------------------------------------------------------- | ----------------------------------------------- |
| Native web search | Charged by the provider (per request + search context size) | Standard token pricing for that model           |
| `:online` (Exa)   | **$7 per 1,000 requests**                                   | Standard token pricing for the underlying model |

Both are billed against your FastRouter credits.

With `:online`, the Exa search fee is charged per request that triggers a search, and is separate from the token cost of the model you pair it with. Search results injected into the prompt count toward your input tokens.
