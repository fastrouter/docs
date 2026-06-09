---
icon: magnifying-glass
---

# Web Search

When using a web-search-enabled model, you can pass the `web_search_options` parameter to control how much search context is retrieved and processed. Models with this capability can dynamically integrate search results into their reasoning process.&#x20;

There's a per-request fee applied by these models. Additionally, these models charge based on search context size, which controls how much data is retrieved and processed per query.

### **Web-Search Enabled Models**

These models support built-in web search:

* `openai/gpt-4o-mini-search-preview`
* `openai/gpt-4o-search-preview`
* `perplexity/sonar-pro`
* `perplexity/sonar-reasoning-pro`
* `perplexity/sonar`
* `perplexity/sonar-reasoning`&#x20;

### **Search Context Size**

The `search_context_size` setting controls how much information is pulled from search results. Pricing may vary based on the selected level.

| Level    | Description                                        | Use Case                           |
| -------- | -------------------------------------------------- | ---------------------------------- |
| `low`    | Minimal context for basic queries                  | Quick facts, dates, headlines      |
| `medium` | Moderate context with broader information coverage | General knowledge, short summaries |
| `high`   | Extensive search context for deep research         | In-depth topics, analysis, reports |

### **Sample Request**

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

### **Enabling Web Search for Any Model**

You can incorporate relevant web search results for _any_ model on FastRouter by appending **:online** to the model slug:

```
{
  "model": "openai/gpt-oss-20b:online"
}
```

### **Pricing**

The web plugin uses your FastRouter credits and charges _$5 per 1000 answers_.
