---
description: >-
  Send OpenAI Decisions-shaped requests to any decision model (gpt-6-luna,
  gemini-decision-1, or TypeSafe Jev) through FastRouter's /decisions endpoint.
icon: gavel
---

# Decisions Endpoint

### Introduction

The `/decisions` endpoint accepts the **OpenAI Decisions** request schema. You send shared evidence (the **input**: text, images, or both) along with a list of **typed questions**, and get back structured answers: the probability that a condition is true, one option from a fixed set, or a score against a rubric.

The schema is the same for every model. To compare or switch models, change only `model`.

> OpenAI's Decisions API is currently in **public beta**.

***

#### Endpoint

```http
POST https://api.fastrouter.ai/api/v1/decisions
Authorization: Bearer <FASTROUTER_API_KEY>
Content-Type: application/json
```

#### Supported models

| Model                 | Provider | Input        |
| --------------------- | -------- | ------------ |
| `gpt-6-luna`          | OpenAI   | Text, images |
| `gemini-decision-1`   | Google   | Text         |
| `typesafe/jev-latest` | TypeSafe | Text, images |
| `typesafe/jev-1.14.0` | TypeSafe | Text, images |
| `typesafe/jev-1.13.0` | TypeSafe | Text, images |

See [Decision Models](decision-models.md) for model notes and pricing.

***

#### Request body

| Field             | Type            | Required | Description                                                                                               |
| ----------------- | --------------- | -------- | --------------------------------------------------------------------------------------------------------- |
| `model`           | string          | Yes      | Any model from [Supported models](decisions-endpoint.md#supported-models).                                |
| `input`           | string \| array | Yes      | Shared evidence for all questions: a text string, or user messages containing text and images.            |
| `questions`       | array           | Yes      | The questions to evaluate. Each has a `type`, a `name`, `instructions`, and type-specific options.        |
| `response_format` | string          | No       | `"decisions"` (default) or `"systemone"`. See [Response formats](decisions-endpoint.md#response-formats). |

This endpoint also accepts the Gemini request format, with any model.

Give every question a **unique `name`**. The API echoes it on each answer so you can match answers to questions. It is required if you use `response_format: "systemone"`, where answers are keyed by name.

**Question types**

| Type        | Use it to                                                            | Main result                                                     |
| ----------- | -------------------------------------------------------------------- | --------------------------------------------------------------- |
| `predicate` | Check a condition, such as visible damage or whether text is urgent. | `probability`: estimate from 0 to 1 that the condition is true. |
| `choice`    | Select one unordered option, such as a department or category.       | `choice`: one of the values you supplied.                       |
| `score`     | Rate against ordered levels, such as severity.                       | `score`: probability-weighted average of the level indices.     |

Use `choice` for categories with no order and `score` for ordered levels. Both return a probability distribution and a `confidence` value.

***

#### Quickstart

Route a customer complaint to a department. The same request works with any supported model.

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
```py
import os
import requests

resp = requests.post(
    "https://api.fastrouter.ai/api/v1/decisions",
    headers={"Authorization": f"Bearer {os.environ['FASTROUTER_API_KEY']}"},
    json={
        "model": "gpt-6-luna",  # or "gemini-decision-1", "typesafe/jev-latest"
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

**Response**

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

**Using other models**

Change `model` and keep the rest of the request. For example, with Gemini:

{% tabs %}
{% tab title="gemini-decision-1" %}
```bash
curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-decision-1",
    "input": "The blue mug I ordered on Monday arrived cracked on Wednesday.",
    "questions": [{
      "type": "predicate",
      "name": "is_damaged",
      "instructions": "Did the item arrive damaged?"
    }]
  }'
```
{% endtab %}

{% tab title="typesafe/jev-latest" %}
```bash
curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "typesafe/jev-latest",
    "input": "The blue mug I ordered on Monday arrived cracked on Wednesday.",
    "questions": [{
      "type": "predicate",
      "name": "is_damaged",
      "instructions": "Did the item arrive damaged?"
    }]
  }'
```
{% endtab %}
{% endtabs %}

The response has the same Decisions shape whichever model serves it. The `model` field reports the model that served the request; for `typesafe/jev-latest` it reports the exact Jev version.

**Gemini request format**

This endpoint also accepts Google's Gemini Decisions request format, with any decision model: `input` for the content and a `questions` **map** keyed by question name, using `noul`, `choice`, and `score` questions with `criteria`. Send existing Gemini requests unchanged, and change `model` to run them on `gpt-6-luna` or Jev.

```bash
curl https://api.fastrouter.ai/api/v1/decisions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-decision-1",
    "input": "The blue mug I ordered on Monday arrived cracked on Wednesday.",
    "questions": {
      "is_damaged": {
        "type": "noul",
        "instructions": "Did the item arrive damaged?"
      },
      "queue": {
        "type": "choice",
        "instructions": "Which support queue should handle this message?",
        "criteria": {
          "billing": "Payment, invoice, or refund questions.",
          "replacements": "Damaged, missing, or wrong items.",
          "general": "Product questions and account updates."
        }
      }
    }
  }'
```

A Gemini-format request returns a **Gemini-format response** by default, even on this endpoint:

```json
{
  "answers": {
    "is_damaged": { "type": "noul", "noul": 0.97, "confidence": 0.97 },
    "queue": {
      "type": "choice",
      "choice": "replacements",
      "probabilities": { "billing": 0.03, "replacements": 0.95, "general": 0.02 },
      "confidence": 0.95
    }
  },
  "usage": {
    "total_input_tokens": 128,
    "input_tokens_by_modality": [{ "modality": "TEXT", "tokens": 128 }],
    "total_output_tokens": 0,
    "total_tokens": 128
  }
}
```

To get a Decisions-format response instead, add `"response_format": "decisions"`.&#x20;

***

#### Question types in detail

**Predicate: check a condition**

Returns the probability that a condition is true.

| Field          | Type          | Required    | Description                                           |
| -------------- | ------------- | ----------- | ----------------------------------------------------- |
| `type`         | `"predicate"` | Yes         |                                                       |
| `name`         | string        | Recommended | Identifier echoed on the answer.                      |
| `instructions` | string        | Yes         | The condition to check, phrased as a yes/no question. |

**Example: check an image for visible damage**

Images go in `input` as `input_image` parts. Combine them with `input_text` parts in a user message to evaluate an image together with context. Image input works with `gpt-6-luna` and the Jev models; see [Images](images.md) for formats, limits, and model support.

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

**Choice: pick one option**

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

**Score: rate against a rubric**

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

**Model-specific limits**

| Limit                         | `gpt-6-luna` | `gemini-decision-1` | Jev models |
| ----------------------------- | ------------ | ------------------- | ---------- |
| Questions per request         | —            | Up to 1,000         | —          |
| Options per `choice` question | —            | —                   | Up to 255  |
| Levels per `score` question   | —            | —                   | 2 to 10    |

A dash means the provider does not publish a specific limit.

***

#### Response formats

By default, a Decisions-format request returns the **Decisions** format shown above, and a Gemini-format request returns the Gemini format. Set `response_format` to `"systemone"` to receive the same answers in the **TypeSafe SystemOne** format. This is useful if your code already parses [SystemOne](https://claude.ai/chat/systemone-endpoint.md) responses. Accepted aliases are `decision` and `openai` for `decisions`, and `system_one` and `typesafe` for `systemone`.

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

`response_format` changes only the **response** shape. The request body must still use the Decisions schema (`input`, a `questions` array, `choices`, `levels`). See the field mapping to convert between the two schemas.

***

#### Response body

<table data-search="false"><thead><tr><th>Field</th><th>Type</th><th>Description</th></tr></thead><tbody><tr><td><code>answers</code></td><td>array (or map with <code>systemone</code>)</td><td>One answer per question.</td></tr><tr><td><code>id</code></td><td>string</td><td>FastRouter request ID, also shown in the Activity Log.</td></tr><tr><td><code>model</code></td><td>string</td><td>Model that served the request. For <code>typesafe/jev-latest</code>, the exact Jev version.</td></tr><tr><td><code>usage.input_tokens</code></td><td>integer</td><td>Input tokens billed, including image tokens.</td></tr><tr><td><code>usage.output_tokens</code></td><td>integer</td><td>Output tokens. Always <code>0</code> for <code>gpt-6-luna</code> and <code>gemini-decision-1</code>.</td></tr><tr><td><code>usage.total_tokens</code></td><td>integer</td><td>Total tokens.</td></tr><tr><td><code>usage.cost</code></td><td>number</td><td>Cost of the request in USD.</td></tr><tr><td><code>usage.chat_id</code></td><td>string</td><td>Same as <code>id</code>.</td></tr><tr><td><code>usage.provider</code></td><td>string</td><td>Upstream provider, such as <code>openai</code>, <code>google</code>, or <code>typesafe</code>.</td></tr></tbody></table>

**Refusals**

If a model declines to answer a question, that question's answer has `"type": "refusal"` and carries the question's `name`. Always check an answer's `type` before reading `probability`, `choice`, or `score`.

```json
{ "type": "refusal", "name": "visible_damage" }
```

***

#### Pricing

See [Decision Models → Pricing](decision-models.md#pricing). `gpt-6-luna` and `gemini-decision-1` are billed on **input tokens only** at **$0.10 per 1M tokens**. Each response reports the exact charge in `usage.cost`.

***

#### Interpreting answers

* **Predicates** return the probability that the condition is true.
* **Choice and score** return a full probability distribution plus `confidence`.
* **Set thresholds on labeled examples** from your own application, weighing the cost of false positives against false negatives. Route low-confidence results to a human or a stronger model.
* **Re-check thresholds when you change models.** Probabilities from different models are not interchangeable; a threshold tuned on `gpt-6-luna` may not suit `gemini-decision-1` or Jev.

#### Best practices

* **Write questions around observable criteria.** "Is there a visible crack?" is better than "Is this product bad?"
* **One concern per question.** Split compound checks into separate questions in the same request.
* **Make options distinct.** Choices should not overlap, and adjacent score levels should have clearly different criteria.
* **Batch independent questions** about the same input in one request; it's cheaper and faster than separate calls.
* **Use Structured Outputs instead** when you need a generated object (extracted fields, written explanations) rather than a probability, choice, or score.

#### Common use cases

* **Support triage:** urgency (predicate), department (choice), severity (score).
* **Image moderation and QA:** damage, policy violations, or product category from photos.
* **Content moderation:** policy checks with calibrated probabilities.
* **LLM output grading:** score other models' responses for accuracy or tone.
* **Agent guardrails:** check a proposed action before allowing it.
