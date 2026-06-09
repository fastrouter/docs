---
icon: wand-sparkles
---

# Automatic Model Selection

### Intelligent Model Routing **Overview**

FastRouter’s intelligent model routing mode lets you delegate model selection to us. Instead of picking a specific model, just set:

```bash
"model": "fastrouter/auto"
```

### **How It Works**

FastRouter will automatically select the most appropriate model based on:

* Complexity of the query
* Domain or topic of the request
* Cost-efficiency

This is the fastest way to get started — no manual tuning, no need to maintain model preference lists.

### Example

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer API-KEY' \
--data '{
  "model": "fastrouter/auto",
  "messages": [
    {
      "role": "user",
      "content": "What is 2+2?"
    }
  ]
}'
```

FastRouter will analyze the input and select the best model from the available pool.

