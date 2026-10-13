---
description: >-
  Get typed yes/no probabilities, choices, and rubric scores from OpenAI's
  gpt-6-luna through FastRouter's Decisions endpoint.
icon: gavel
---

# OpenAI Decisions: GPT-6 Luna

***

## Introduction

FastRouter supports OpenAI's **Decisions API** through the `/decisions` endpoint. You send shared evidence (the **input**: text, images, or both) along with a list of **typed questions**, and get back structured answers: the probability that a condition is true, one option from a fixed set, or a score against a rubric.

Use it to classify content, route requests, moderate images, triage issues, and prioritize work wherever your application needs a decision rather than generated text.

**Why use Decisions through FastRouter**

* **One API key:** use your existing FastRouter key, with no separate OpenAI account needed.
* **Cost in every response:** `usage.cost` reports what each request cost.
* **Observability:** every request appears in the Activity Log with tokens, latency, and cost.
* **Switchable response format:** receive answers in the Decisions format (default) or the TypeSafe systemone format with `response_format`.

> OpenAI's Decisions API is currently in **public beta**.

***

### Endpoint

```http
POST https://api.fastrouter.ai/api/v1/decisions
Authorization: Bearer <FASTROUTER_API_KEY>
Content-Type: application/json
```

### Available models

| Model        | Description                                                          |
| ------------ | -------------------------------------------------------------------- |
| `gpt-6-luna` | OpenAI's decision model; currently the only model for this endpoint. |

***

### How a request works

| Field             | Type            | Required | Description                                                                                                                                     |
| ----------------- | --------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `model`           | string          | Yes      | `gpt-6-luna`.                                                                                                                                   |
| `input`           | string \| array | Yes      | Shared evidence for all questions: a text string, or user messages containing text and images.                                                  |
| `questions`       | array           | Yes      | The questions to evaluate. Each has a `type`, a `name`, `instructions`, and type-specific options.                                              |
| `response_format` | string          | No       | `"decisions"` (default) or `"systemone"`. See [Response formats](https://claude.ai/chat/1394a175-27d7-41bf-9c69-db45cb8377c5#response-formats). |

Give every question a **unique `name`**. The API echoes it on each answer so you can match answers to questions. It is required if you use `response_format: "systemone"`, where answers are keyed by name.

#### Question types

| Type        | Use it to                                                            | Main result                                                     |
| ----------- | -------------------------------------------------------------------- | --------------------------------------------------------------- |
| `predicate` | Check a condition, such as visible damage or whether text is urgent. | `probability`: estimate from 0 to 1 that the condition is true. |
| `choice`    | Select one unordered option, such as a department or category.       | `choice`: one of the values you supplied.                       |
| `score`     | Rate against ordered levels, such as severity.                       | `score`: probability-weighted average of the level indices.     |

Use `choice` for categories with no order and `score` for ordered levels. Both return a probability distribution and a `confidence` value.

***

### Quickstart

Route a customer complaint to a department.

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-6-luna",
    "input": "I was charged twice for my order.",
    "questions": [{
      "type": "choice",
      "name": "department",
      "instructions": "Which department should handle this complaint?",
      "choices": [
        {"value": "billing", "description": "Payments, invoices, and refunds."},
        {"value": "technical", "description": "Problems using the product."},
        {"value": "shipping", "description": "Delivery and tracking."},
        {"value": "other", "description": "Requests outside these categories."}
      ]
    }]
  }'
```
{% endtab %}

{% tab title="Python (requests)" %}
```python
import os
import requests

resp = requests.post(
    "https://api.fastrouter.ai/api/v1/decisions",
    headers={"Authorization": f"Bearer {os.environ['FASTROUTER_API_KEY']}"},
    json={
        "model": "gpt-6-luna",
        "input": "I was charged twice for my order.",
        "questions": [
            {
                "type": "choice",
                "name": "department",
                "instructions": "Which department should handle this complaint?",
                "choices": [
                    {"value": "billing", "description": "Payments, invoices, and refunds."},
                    {"value": "technical", "description": "Problems using the product."},
                    {"value": "shipping", "description": "Delivery and tracking."},
                    {"value": "other", "description": "Requests outside these categories."},
                ],
            }
        ],
    },
    timeout=30,
)
resp.raise_for_status()
print(resp.json()["answers"][0]["choice"])  # "billing"
```
{% endtab %}
{% endtabs %}

#### Response

```json
{
  "answers": [
    {
      "type": "choice",
      "name": "department",
      "choice": "billing",
      "probabilities": [
        { "value": "billing", "probability": 1 },
        { "value": "technical", "probability": 0 },
        { "value": "shipping", "probability": 0 },
        { "value": "other", "probability": 0 }
      ],
      "confidence": 1
    }
  ],
  "id": "fr_64320228-976f-40d3-91b6-1f81b3c5c10f",
  "model": "gpt-6-luna",
  "usage": {
    "input_tokens": 145,
    "output_tokens": 0,
    "total_tokens": 145,
    "cost": 0.0000145,
    "chat_id": "fr_64320228-976f-40d3-91b6-1f81b3c5c10f",
    "provider": "openai"
  }
}
```

***

### Question types in detail

#### Predicate: check a condition

Returns the probability that a condition is true.

| Field          | Type          | Required    | Description                                           |
| -------------- | ------------- | ----------- | ----------------------------------------------------- |
| `type`         | `"predicate"` | Yes         |                                                       |
| `name`         | string        | Recommended | Identifier echoed on the answer.                      |
| `instructions` | string        | Yes         | The condition to check, phrased as a yes/no question. |

**Example: check an image for visible damage**

Images must be **inline base64 data URLs**. Hosted HTTP/HTTPS image URLs and `file_id` inputs are not supported. Combine `input_text` and `input_image` parts in a user message to evaluate an image together with context.

**cURL**

```bash
IMAGE_BASE64="$(base64 < product.png | tr -d '\r\n')"

curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  --data-binary @- <<JSON
{
  "model": "gpt-6-luna",
  "input": [{
    "role": "user",
    "content": [
      {"type": "input_text", "text": "Inspect the product in this photo."},
      {"type": "input_image", "image_url": "data:image/png;base64,$IMAGE_BASE64"}
    ]
  }],
  "questions": [{
    "type": "predicate",
    "name": "visible_damage",
    "instructions": "Does the product have visible damage, such as a crack, tear, or dent? Ignore shadows and damage to the packaging."
  }]
}
JSON
```

The heredoc (`<<JSON`) lets the shell substitute `$IMAGE_BASE64` into the body. A single-quoted `-d '...'` body would send the literal text `$IMAGE_BASE64`.

**Response**

```json
{
  "answers": [
    {
      "type": "predicate",
      "name": "visible_damage",
      "probability": 1
    }
  ],
  "id": "fr_8ab42702-f204-4c70-84ee-ca4bd0f41c71",
  "model": "gpt-6-luna",
  "usage": {
    "input_tokens": 687,
    "output_tokens": 0,
    "total_tokens": 687,
    "cost": 0.0000687,
    "chat_id": "fr_8ab42702-f204-4c70-84ee-ca4bd0f41c71",
    "provider": "openai"
  }
}
```

#### Choice: pick one option

| Field                   | Type       | Required    | Description                         |
| ----------------------- | ---------- | ----------- | ----------------------------------- |
| `type`                  | `"choice"` | Yes         |                                     |
| `name`                  | string     | Recommended | Identifier echoed on the answer.    |
| `instructions`          | string     | Yes         | What the model should decide.       |
| `choices`               | array      | Yes         | The options.                        |
| `choices[].value`       | string     | Yes         | Returned as `choice` when selected. |
| `choices[].description` | string     | No          | When this option applies.           |

**Answer fields**

| Field           | Description                                                     |
| --------------- | --------------------------------------------------------------- |
| `choice`        | The selected value.                                             |
| `probabilities` | Array of `{ value, probability }` for every option (sums to 1). |
| `confidence`    | 0–1 certainty derived from the distribution.                    |

Include a fallback such as `"other"` when your options might not cover every input, and send those results to a general review queue.

#### Score: rate against a rubric

| Field                  | Type      | Required    | Description                       |
| ---------------------- | --------- | ----------- | --------------------------------- |
| `type`                 | `"score"` | Yes         |                                   |
| `name`                 | string    | Recommended | Identifier echoed on the answer.  |
| `instructions`         | string    | Yes         | What the model should rate.       |
| `levels`               | array     | Yes         | Ordered levels, **lowest first**. |
| `levels[].label`       | string    | Yes         | Short level name.                 |
| `levels[].description` | string    | No          | Criteria for this level.          |

**Request**

```bash
curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-6-luna",
    "input": "Export fails in Safari but works in Chrome.",
    "questions": [{
      "type": "score",
      "name": "severity",
      "instructions": "How severe is this issue?",
      "levels": [
        {"label": "Cosmetic", "description": "Appearance only; no lost functionality."},
        {"label": "Workaround available", "description": "A task fails, but another way works."},
        {"label": "Fully blocked", "description": "A task fails with no workaround."}
      ]
    }]
  }'
```

**Response**

```json
{
  "answers": [
    {
      "type": "score",
      "name": "severity",
      "score": 0.99,
      "probabilities": [
        { "value": 0, "label": "Cosmetic", "probability": 0.02 },
        { "value": 1, "label": "Workaround available", "probability": 0.97 },
        { "value": 2, "label": "Fully blocked", "probability": 0.01 }
      ],
      "confidence": 0.96
    }
  ],
  "id": "fr_05f25c5e-2f0d-4ad6-94d2-e48f96500656",
  "model": "gpt-6-luna",
  "usage": {
    "input_tokens": 146,
    "output_tokens": 0,
    "total_tokens": 146,
    "cost": 0.0000146,
    "chat_id": "fr_05f25c5e-2f0d-4ad6-94d2-e48f96500656",
    "provider": "openai"
  }
}
```

Level indices start at **0**. `score` is the probability-weighted average of the indices, so it can fall between levels. Here, 0.02 × 0 + 0.97 × 1 + 0.01 × 2 = 0.99.

***

### Response formats

By default, `/decisions` returns the **Decisions** format shown above. Set `response_format` to `"systemone"` to receive the same answers in the **TypeSafe systemone** format. This is useful if your code already parses [TypeSafe Jev](https://claude.ai/typesafe-jev) responses.

| `response_format`       | `answers` shape                   | Choice/score `probabilities`                                         |
| ----------------------- | --------------------------------- | -------------------------------------------------------------------- |
| `"decisions"` (default) | Array; each answer carries `name` | Array of `{ value, label?, probability }`                            |
| `"systemone"`           | Map keyed by question `name`      | Map of option or level index → probability, plus `legend` for scores |

**Request** (same score question as above, with `response_format`)

```bash
curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-6-luna",
    "input": "Export fails in Safari but works in Chrome.",
    "response_format": "systemone",
    "questions": [{
      "type": "score",
      "name": "severity",
      "instructions": "How severe is this issue?",
      "levels": [
        {"label": "Cosmetic", "description": "Appearance only; no lost functionality."},
        {"label": "Workaround available", "description": "A task fails, but another way works."},
        {"label": "Fully blocked", "description": "A task fails with no workaround."}
      ]
    }]
  }'
```

**Response**

```json
{
  "answers": {
    "severity": {
      "type": "score",
      "score": 0.99,
      "legend": {
        "0": "Cosmetic",
        "1": "Workaround available",
        "2": "Fully blocked"
      },
      "probabilities": { "0": 0.02, "1": 0.97, "2": 0.01 },
      "confidence": 0.96
    }
  },
  "id": "fr_d6bb65ce-b6e8-4f6d-a8cc-0f0dbb715c1d",
  "model": "gpt-6-luna",
  "usage": {
    "input_tokens": 146,
    "output_tokens": 0,
    "total_tokens": 146,
    "cost": 0.0000146,
    "chat_id": "fr_d6bb65ce-b6e8-4f6d-a8cc-0f0dbb715c1d",
    "provider": "openai"
  }
}
```

`response_format` changes only the **response** shape. The request body must still use the Decisions schema (`input`, a `questions` array, `choices`, `levels`).

#### Decisions vs. systemone field mapping

<table data-header-hidden data-search="false"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td>Concept</td><td><code>/decisions</code> request</td><td><code>/systemone</code> request</td></tr><tr><td>Content to evaluate</td><td><code>input</code></td><td><code>state</code></td></tr><tr><td>Question list</td><td><code>questions</code> array</td><td><code>questions</code> map</td></tr><tr><td>Question identifier</td><td><code>name</code></td><td>Map key</td></tr><tr><td>Yes/no question</td><td><code>predicate</code></td><td><code>noul</code></td></tr><tr><td>Choice options</td><td><code>choices[]</code> (<code>value</code>, <code>description</code>)</td><td><code>criteria</code> map (option → description)</td></tr><tr><td>Score levels</td><td><code>levels[]</code> (<code>label</code>, <code>description</code>)</td><td><code>criteria</code> array</td></tr></tbody></table>

***

### Response body

<table data-search="false"><thead><tr><th>Field</th><th>Type</th><th>Description</th></tr></thead><tbody><tr><td><code>answers</code></td><td>array (or map with <code>systemone</code>)</td><td>One answer per question.</td></tr><tr><td><code>id</code></td><td>string</td><td>FastRouter request ID, also shown in the Activity Log.</td></tr><tr><td><code>model</code></td><td>string</td><td>Model that served the request.</td></tr><tr><td><code>usage.input_tokens</code></td><td>integer</td><td>Input tokens billed.</td></tr><tr><td><code>usage.output_tokens</code></td><td>integer</td><td>Always <code>0</code>; Decisions bills input tokens only.</td></tr><tr><td><code>usage.total_tokens</code></td><td>integer</td><td>Total tokens.</td></tr><tr><td><code>usage.cost</code></td><td>number</td><td>Cost of the request in USD.</td></tr><tr><td><code>usage.chat_id</code></td><td>string</td><td>Same as <code>id</code>.</td></tr><tr><td><code>usage.provider</code></td><td>string</td><td>Upstream provider (<code>openai</code>).</td></tr></tbody></table>

#### Refusals

If the model declines to answer a question, that question's answer has `"type": "refusal"` and carries the question's `name`. Always check an answer's `type` before reading `probability`, `choice`, or `score`.

```json
{ "type": "refusal", "name": "visible_damage" }
```

***

### Pricing

`gpt-6-luna` on `/decisions` is billed on **input tokens only** at **$0.10 per 1M tokens**. There are no output, cache-read, or cache-write charges. Each response reports the exact charge in `usage.cost`.

***

### Interpreting answers

* **Predicates** return the probability that the condition is true.
* **Choice and score** return a full probability distribution plus `confidence`.
* **Set thresholds on labeled examples** from your own application, weighing the cost of false positives against false negatives. Route low-confidence results to a human or a stronger model.

### Best practices

* **Write questions around observable criteria.** "Is there a visible crack?" is better than "Is this product bad?"
* **One concern per question.** Split compound checks into separate questions in the same request.
* **Make options distinct.** Choices should not overlap, and adjacent score levels should have clearly different criteria.
* **Batch independent questions** about the same input in one request; it's cheaper and faster than separate calls.
* **Use Structured Outputs instead** when you need a generated object (extracted fields, written explanations) rather than a probability, choice, or score.

### Common use cases

* **Support triage:** urgency (predicate), department (choice), severity (score).
* **Image moderation and QA:** damage, policy violations, or product category from photos.
* **Content moderation:** policy checks with calibrated probabilities.
* **LLM output grading:** score other models' responses for accuracy or tone.
* **Agent guardrails:** check a proposed action before allowing it.
