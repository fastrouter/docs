---
description: >-
  Group related LLM API calls into a single trace using a simple traceparent
  header.
icon: list-timeline
---

# Tracing

### Overview

FastRouter supports the **W3C Trace Context** standard via the `traceparent` header, enabling you to group multiple LLM API calls into a single trace with ordered spans.

This helps you understand the full lifecycle of complex workflows—across models, providers, and steps—in one place.

**Common use cases:**

* **Agentic workflows** — multi-step chains with tool/function calls
* **Chat sessions** — linking all turns in a conversation
* **Parallel requests** — grouping concurrent calls

Tracing works with any HTTP client. No SDK or proprietary tooling is required.

***

### How Tracing Works

When FastRouter receives a request with a `traceparent` header:

* Extracts `trace_id` to group related requests
* Records `parent_id` as the caller (application) span
* Applies overrides from optional headers (if present)
* Generates a new `span_id` for the gateway span
* Captures latency, tokens, cost, and full request/response
* Stores the span and groups it under the corresponding trace

All API calls sharing the same `trace_id` appear as a **single trace with multiple spans**.

> **Key rule:** Reuse the same `traceparent` across all requests in a workflow.

***

### traceparent Header Format

```
traceparent: {version}-{trace_id}-{parent_id}-{flags}
```

| Field      | Format       | Description                                                           |
| ---------- | ------------ | --------------------------------------------------------------------- |
| version    | `00`         | Fixed (W3C standard)                                                  |
| trace\_id  | 32 hex chars | Unique ID for the entire trace                                        |
| parent\_id | 16 hex chars | Caller span ID (your application)                                     |
| flags      | `01`         | Sampling flag. Note: Currently, all requests are shown on FastRouter. |

**Example:**

```
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

***

### Optional Headers

FastRouter supports additional headers to improve trace readability and control:

| Header        | Description                                          | Example            |
| ------------- | ---------------------------------------------------- | ------------------ |
| `x-span-name` | Human-readable label for this span                   | `hotel-search`     |
| `x-span-id`   | Custom span ID (optional; auto-generated if omitted) | `a1b2c3d4e5f6a7b8` |
| `x-trace-id`  | Overrides `trace_id` from `traceparent`              | `abc123...`        |

***

### Usage

{% tabs %}
{% tab title="cURL" %}
```bash
TRACE_ID=$(xxd -p -l 16 /dev/urandom)
PARENT_ID=$(xxd -p -l 8 /dev/urandom)
TRACEPARENT="00-${TRACE_ID}-${PARENT_ID}-01"

curl https://api.fastrouter.ai/v1/chat/completions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -H "traceparent: $TRACEPARENT" \
  -d '{
    "model": "gpt-4.1",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import secrets
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_FASTROUTER_API_KEY",
    base_url="https://api.fastrouter.ai/v1"
)

def generate_traceparent():
    return f"00-{secrets.token_hex(16)}-{secrets.token_hex(8)}-01"

traceparent = generate_traceparent()

# Step 1
response1 = client.chat.completions.create(
    model="gpt-4.1",
    messages=[...],
    extra_headers={"traceparent": traceparent}
)

# Step 2 (same trace)
response2 = client.chat.completions.create(
    model="gpt-4.1",
    messages=[...],
    extra_headers={"traceparent": traceparent}
)
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
import OpenAI from "openai";
import { randomBytes } from "crypto";

const client = new OpenAI({
  apiKey: process.env.FASTROUTER_API_KEY,
  baseURL: "https://api.fastrouter.ai/v1",
});

function generateTraceparent(): string {
  const traceId = randomBytes(16).toString("hex");
  const parentId = randomBytes(8).toString("hex");
  return `00-${traceId}-${parentId}-01`;
}

const traceparent = generateTraceparent();

await client.chat.completions.create(
  {
    model: "gpt-4.1",
    messages: [{ role: "user", content: "Hello" }],
  },
  {
    headers: { traceparent },
  }
);
```
{% endtab %}
{% endtabs %}

***

### What FastRouter does

* Groups by `trace_id`
* Uses your `parent_id`
* Generates `span_id`
* Names spans: `POST /api/v1/chat/completions`

***

### Output

One trace → multiple spans

```
Trace: <trace_id>
├── POST /api/v1/chat/completions
└── POST /api/v1/chat/completions
```

Each span includes: latency, tokens, cost, request, response.
