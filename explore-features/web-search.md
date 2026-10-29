---
description: >-
  The fastrouter:web_search server tool gives any model on FastRouter access to
  real-time web information.
icon: magnifying-glass
---

# Web Search

## Introduction

FastRouter supports two ways to give a model access to live web results:

1. `fastrouter:web_search` server tool gives any model on FastRouter access to real-time web information. When the model determines it needs current information, it calls the tool with a search query. FastRouter executes the search and returns results that the model uses to formulate a grounded, cited response.
2. <mark style="color:$danger;">**Deprecated**</mark>**: `:online` suffix.** Appends Exa-powered web search results to any model on FastRouter.

> <mark style="color:red;">The \`:online\` web search slug is deprecated. Use the \`fastrouter:web\_search\` server tool instead.</mark>&#x20;

### How It Works

1. You include `{ "type": "fastrouter:web_search" }` in your `tools` array.
2. Based on the user's prompt, the model decides whether a web search is needed and generates a search query.
3. FastRouter executes the search using the configured engine (defaults to `auto`, which uses the provider's native search when the model supports it, or falls back to Exa).
4. The search results (URLs, titles, and content snippets) are returned to the model.
5. The model synthesizes the results into its response. It may search multiple times in a single request if needed.

### Quick Start

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.fastrouter.ai/v1/chat/completions \
  -H "Authorization: Bearer <FASTROUTER_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "openai/gpt-5.2",
    "messages": [
      { "role": "user", "content": "What were the major AI announcements this week?" }
    ],
    "tools": [
      { "type": "fastrouter:web_search" }
    ]
  }'
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
const response = await fetch('https://api.fastrouter.ai/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <FASTROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    messages: [
      { role: 'user', content: 'What were the major AI announcements this week?' }
    ],
    tools: [
      { type: 'fastrouter:web_search' }
    ]
  }),
});

const data = await response.json();
console.log(data.choices[0].message.content);
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

response = requests.post(
    "https://api.fastrouter.ai/v1/chat/completions",
    headers={
        "Authorization": "Bearer <FASTROUTER_API_KEY>",
        "Content-Type": "application/json",
    },
    json={
        "model": "openai/gpt-5.2",
        "messages": [
            {"role": "user", "content": "What were the major AI announcements this week?"}
        ],
        "tools": [
            {"type": "fastrouter:web_search"}
        ],
    },
)

data = response.json()
print(data["choices"][0]["message"]["content"])
```
{% endtab %}
{% endtabs %}

### Configuration

The web search tool accepts optional `parameters` to customize search behavior:

```json
{
  "type": "fastrouter:web_search",
  "parameters": {
    "engine": "exa",
    "max_results": 5,
    "max_total_results": 20,
    "search_context_size": "medium",
    "allowed_domains": ["arxiv.org", "nature.com"],
    "excluded_domains": ["reddit.com"]
  }
}
```

<table data-search="false"><thead><tr><th>Parameter</th><th>Type</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td><code>engine</code></td><td>string</td><td><code>auto</code></td><td>Search engine to use: <code>auto</code>, <code>native</code>, <code>exa</code>, <code>jina</code>, <code>parallel</code>, or <code>you</code>.</td></tr><tr><td><code>max_results</code></td><td>integer</td><td>5</td><td>Maximum results per search call (1–25). Applied by engines that support result-count limits; ignored with native provider search.</td></tr><tr><td><code>max_total_results</code></td><td>integer</td><td>—</td><td>Maximum total results across all search calls in a single request. Useful for controlling cost and context size in agentic loops.</td></tr><tr><td><code>search_context_size</code></td><td>string</td><td>—</td><td>How much context to retrieve: <code>low</code>, <code>medium</code>, or <code>high</code>. For Exa, pins a fixed per-result character cap (5K / 15K / 30K); when omitted, Exa picks adaptively (~2–4K per result). For Parallel, controls total characters across all results. Ignored by Jina, You.com, and native provider search. Overridden by <code>max_characters</code> when both are set.</td></tr><tr><td><code>max_characters</code></td><td>integer</td><td>—</td><td>Exact maximum characters of content per result (1–100,000). For Exa, caps highlight content per result. For Parallel, caps excerpt content per result. Ignored by Jina, You.com, and native provider search. Takes precedence over <code>search_context_size</code>.</td></tr><tr><td><code>user_location</code></td><td>object</td><td>—</td><td>Approximate user location for location-biased results. Currently only honored by native provider search; ignored by Exa, Jina, Parallel, and You.com.</td></tr><tr><td><code>allowed_domains</code></td><td>string[]</td><td>—</td><td>Limit results to these domains. See Domain Filtering.</td></tr><tr><td><code>excluded_domains</code></td><td>string[]</td><td>—</td><td>Exclude results from these domains. See Domain Filtering.</td></tr></tbody></table>

#### User Location

Pass an approximate user location to bias search results geographically. All fields are optional.

```json
{
  "type": "fastrouter:web_search",
  "parameters": {
    "user_location": {
      "type": "approximate",
      "city": "San Francisco",
      "region": "California",
      "country": "US",
      "timezone": "America/Los_Angeles"
    }
  }
}
```

### Engine Selection

The web search server tool supports multiple search engines:

* **`auto`** (default): Uses the provider's native search if the model supports it, otherwise falls back to Exa.
* **`native`**: Prefers the provider's built-in web search; falls back to Exa if the model doesn't support native search. See Native Search Providers.
* **`exa`**: Uses Exa's search API, which combines keyword and embeddings-based search. Returns Exa highlights — excerpts drawn from each page that are most relevant to the query — rather than truncated page text. See the Exa section below.
* **`jina`**: Uses Jina's search API. Billed by tokens of returned content.
* **`parallel`**: Uses Parallel's search API, with domain filtering and total context-size control.
* **`you`**: Uses the You.com Search API for ranked web results.

#### Engine Capabilities

| Feature                  | Exa         | Jina        | Parallel    | You.com     |
| ------------------------ | ----------- | ----------- | ----------- | ----------- |
| **Domain filtering**     | Yes         | Yes         | Yes         | Yes         |
| **Context size control** | Yes\*       | No          | Yes\*\*     | No          |
| **API key**              | Server-side | Server-side | Server-side | Server-side |



> _\* Exa: limit applies **per result.**_
>
> _\*\* Parallel: limit applies as a **total across all results**._&#x20;

#### Exa

FastRouter requests Exa highlights for each result rather than the `text` content option. Highlights are extractive excerpts drawn directly from the page that Exa selects as most relevant to the query, typically yielding higher-quality context per token than truncated page text for agentic web tooling.

By default, Exa selects an adaptive highlight size per query and document — typically \~2,000–4,000 characters per result. You can control the per-result character budget in two ways:

**Coarse presets via `search_context_size`** — maps to Exa's `contents.highlights.maxCharacters`:

* `low` — 5,000 characters per result
* `medium` — 15,000 characters per result
* `high` — 30,000 characters per result

**Exact value via `max_characters`** — pass any integer (1–100,000) to set a precise per-result content budget. When both `max_characters` and `search_context_size` are set, `max_characters` takes precedence.

```json
{
  "type": "fastrouter:web_search",
  "parameters": {
    "engine": "exa",
    "max_characters": 2000
  }
}
```

The selected excerpts are returned to the model on each result and surfaced to API callers via `url_citation` annotations. Within a single result, excerpts from different parts of the page are separated by Exa's `[...]` markers, so the `content` field of a `url_citation` annotation may look like:

```
First excerpt drawn from the page.
[...]
Second excerpt drawn from elsewhere in the same page.
[...]
Third excerpt.
```

#### Jina

Uses Jina's search API to retrieve page content for each result. Jina is metered on the tokens of returned content rather than on result count (see Pricing), so `search_context_size` and `max_characters` do not apply.

#### Parallel

Parallel supports domain filtering and context-size control via `search_context_size` (applied as a total across all returned results). Each call includes up to 10 results; additional results are billed incrementally (see Pricing).

#### You.com

You.com returns ranked web results (titles, URLs, snippets). Billing is a fixed charge per call, independent of result count.

### Native Search Providers

When `engine` is `auto` (the default) or `native`, FastRouter uses the provider's built-in search for supported models. Parameters such as `max_results`, `search_context_size`, and `max_characters` are ignored with native search; `user_location` is honored. The following providers have native web search:

* [**OpenAI**](https://developers.openai.com/api/docs/guides/tools-web-search?api-mode=responses) — GPT-4.1 (including Mini and Nano), GPT-5 and later.
* [**Anthropic**](https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool) — Claude 3.5 Haiku, Claude 3.7 Sonnet, and Claude 4 and later (all Opus/Sonnet variants).
* [**Google**](https://ai.google.dev/gemini-api/docs/google-search) — Gemini models that support Google Search grounding; see the provider's [supported models](https://ai.google.dev/gemini-api/docs/google-search#supported_models).
* [**xAI**](https://docs.x.ai/developers/tools/web-search) — Grok models (includes both web search and X search).

> For models without native web search, \`engine: "native"\` (and \`engine: "auto"\`) fall back to Exa. Set \`engine\` explicitly to \`exa\`, \`jina\`, \`parallel\`, or \`you\` to choose a specific non-native engine.

### Domain Filtering

Restrict which domains appear in search results using `allowed_domains` and `excluded_domains`:

json

```json
{
  "type": "fastrouter:web_search",
  "parameters": {
    "allowed_domains": ["arxiv.org", "nature.com"],
    "excluded_domains": ["reddit.com"]
  }
}
```

| Engine       | `allowed_domains` | `excluded_domains` | Notes                                                                               |
| ------------ | ----------------- | ------------------ | ----------------------------------------------------------------------------------- |
| **Exa**      | Yes               | Yes                | Both can be used simultaneously.                                                    |
| **Parallel** | Yes               | Yes                | Mutually exclusive.                                                                 |
| **Jina**     | Yes               | Yes                | —                                                                                   |
| **You.com**  | Yes               | Yes                | —                                                                                   |
| **Native**   | Varies            | Varies             | Support depends on the provider; unsupported filters fall back to Exa under `auto`. |

### Controlling Total Results

When the model searches multiple times in a single request, use `max_total_results` to cap the cumulative number of results:

json

```json
{
  "type": "fastrouter:web_search",
  "parameters": {
    "max_results": 5,
    "max_total_results": 15
  }
}
```

Once the limit is reached, subsequent search calls return a message telling the model the limit was hit instead of performing another search. This is useful for controlling cost and context-window usage in agentic loops.

### Works with the Responses API

The web search server tool also works with the Responses API:

{% tabs %}
{% tab title="Python" %}
```python
import requests

response = requests.post(
    "https://api.fastrouter.ai/v1/responses",
    headers={
        "Authorization": "Bearer <FASTROUTER_API_KEY>",
        "Content-Type": "application/json",
    },
    json={
        "model": "openai/gpt-5.2",
        "input": "What is the current price of Bitcoin?",
        "tools": [
            {"type": "fastrouter:web_search", "parameters": {"max_results": 3}}
        ],
    },
)

data = response.json()
print(data)
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
const response = await fetch('https://api.fastrouter.ai/v1/responses', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <FASTROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: 'openai/gpt-5.2',
    input: 'What is the current price of Bitcoin?',
    tools: [
      { type: 'fastrouter:web_search', parameters: { max_results: 3 } }
    ]
  }),
});

const data = await response.json();
console.log(data);
```
{% endtab %}
{% endtabs %}

### Usage Tracking

Web search usage is reported in the response `usage` object:

```json
{
  "usage": {
    "input_tokens": 105,
    "output_tokens": 250,
    "server_tool_use": {
      "web_search_requests": 2
    }
  }
}
```

The `web_search_requests` field counts the total number of search queries the model made during the request. Search engine, request count, and cost are also recorded in your [Activity Log](https://fastrouter.ai/activity) for per-request reconciliation.

### Pricing

Search cost is charged from your wallet in addition to standard LLM token costs for processing the returned content.

| Engine       | FastRouter charge                     | Equivalent per 1,000 calls            | Calculation                                                                         |
| ------------ | ------------------------------------- | ------------------------------------- | ----------------------------------------------------------------------------------- |
| **Jina**     | $0.05 per million tokens              | $0.50 if each call uses 10,000 tokens | Uses upstream `meta.usage.tokens`; falls back to 10,000 tokens when unavailable     |
| **Parallel** | $0.005 per call, including 10 results | $5 with ≤10 results per call          | Adds $0.001 per returned result above 10, before local filtering                    |
| **Exa**      | Actual upstream cost                  | Depends on Exa's reported cost        | Uses `costDollars.total`; fallback is $0.005 + $0.001 per requested result above 10 |
| **You.com**  | $0.005 per call                       | $5                                    | Fixed charge, independent of result count                                           |
| **Native**   | Passed through from the provider      | Varies by provider                    | Billed at the provider's own web search rate                                        |

> Native web search pricing is passed through from the provider with no FastRouter markup. Refer to each provider's pricing documentation for current rates:&#x20;
>
> * OpenAI (https://developers.openai.com/api/docs/guides/tools-web-search?api-mode=responses)
> * Anthropic (https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool)
> * Google (https://ai.google.dev/gemini-api/docs/google-search)
> * xAI (https://docs.x.ai/developers/tools/web-search)

### Migrating from `:online`

The `:online` web search slug is deprecated. Use the `fastrouter:web_search` server tool instead.

```json
// Before (deprecated)
{
  "model": "openai/gpt-5.2:online"
}

// After
{
  "model": "openai/gpt-5.2",
  "tools": [
    { "type": "fastrouter:web_search" }
  ]
}
```

The key differences:

| **What**                  | `:online` (deprecated) | `fastrouter:web_search`                        |
| ------------------------- | ---------------------- | ---------------------------------------------- |
| **How to enable**         | `:online` model slug   | `tools: [{ "type": "fastrouter:web_search" }]` |
| **Who decides to search** | Always searches once   | Model decides when/whether to search           |
| **Call frequency**        | Once per request       | 0 to N times per request                       |
| **Total results cap**     | No                     | Yes (`max_total_results`)                      |
