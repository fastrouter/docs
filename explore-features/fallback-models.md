---
icon: list-tree
---

# Fallback Models

### Fallback **Model Lists Overview**

FastRouter supports **fallback** **model lists**, allowing you to provide multiple fallback models in a single request. If the primary model is unavailable or returns an error (e.g., due to rate limits, downtime, or moderation), FastRouter will automatically attempt to route the request to the next available model in your list.

### **How It Works**

* Use the `model` field to define your **primary model** and the `models` array to define one or more **fallback models**.
* FastRouter will try the primary model first. If it fails, it will iterate through the list in order until a successful response is received or all models fail.
* The final model used is returned in the `model` field of the response body.
* Billing is based on the model that actually processes the request.

### Example

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer API-KEY' \
--data '{
    "model": "openai/gpt-4o",
    "models":["openai/o1", "google/gemini-1.5-pro"],
    "messages": [
        {
            "role": "user",
            "content": "Why would I need an LLM judge?"
        }
    ],
    "stream": true
} 
'
```

#### What Happens:

1. FastRouter first tries `openai/gpt-4o`.
2. If it fails (due to downtime, rate limit, moderation, etc.), it tries `openai/o1`.
3. If that fails, it then tries `google/gemini-1.5-pro`.
4. If all fail, the final error is returned to the user.

