---
description: >-
  Send TypeSafe SystemOne-shaped requests to any decision model (TypeSafe Jev,
  gpt-6-luna, or gemini-decision-1) through FastRouter's /systemone endpoint.
icon: gavel
---

# SystemOne Endpoint

### Introduction

The `/systemone` endpoint accepts the **TypeSafe SystemOne** request schema. Instead of prompting a chat model and parsing free-form text, you send a piece of content (the **state**) along with a set of **typed questions**, and get back **structured, probability-calibrated answers**, one per question.

The schema is the same for every model. To compare or switch models, change only `model`.

Use it for classification, routing, triage, moderation, quality scoring, deduplication, and any workflow where you need a reliable, machine-readable decision rather than prose.

**Why use `/systemone` through FastRouter**

* **One API key:** use your existing FastRouter key, with no separate TypeSafe, OpenAI, or Google account.
* **Unified billing and usage:** decision model usage appears alongside all your other model spend.
* **Observability:** every request is visible in the Activity Log with tokens, latency, and cost.
* **Structured by design:** answers are typed (`noul`, `choice`, `score`) and include probabilities and confidence, so no output parsing is needed.
* **Switchable response format:** receive answers in the SystemOne format or the OpenAI Decisions format with `response_format`. **By default, the response matches the request shape**: a SystemOne-format request receives a SystemOne-format response, and a Gemini-format request receives a Gemini-format response. Accepted aliases are `decision` and `openai` for `decisions`, and `system_one` and `typesafe` for `systemone`.

***

#### Endpoint

```http
POST https://api.fastrouter.ai/api/v1/systemone
Authorization: Bearer <FASTROUTER_API_KEY>
Content-Type: application/json
```

#### Supported models

| Model                 | Provider | Input        | Description                                                                               |
| --------------------- | -------- | ------------ | ----------------------------------------------------------------------------------------- |
| `typesafe/jev-latest` | TypeSafe | Text, images | TypeSafe's flagship evaluation model; alias that always points to the latest Jev release. |
| `typesafe/jev-1.14.0` | TypeSafe | Text, images | Pinned Jev 1.14 release.                                                                  |
| `typesafe/jev-1.13.0` | TypeSafe | Text, images | Pinned Jev 1.13 release.                                                                  |
| `gpt-6-luna`          | OpenAI   | Text, images | OpenAI's decision model.                                                                  |
| `gemini-decision-1`   | Google   | Text         | Google's decision model.                                                                  |

For `typesafe/jev-latest`, the response's `model` field reports the exact version that served the request (for example `jev-1.14.0`). See [Decision Models](decision-models.md) for model notes and pricing.

***

#### Quickstart

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.fastrouter.ai/api/v1/systemone \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "typesafe/jev-latest",
    "state": "Help! My payouts have been failing for 3 days.",
    "questions": {
      "is_urgent": {
        "type": "noul",
        "instructions": "Does this convey urgency?"
      }
    }
  }'
```
{% endtab %}

{% tab title="Python" %}
```py
import os
import requests

response = requests.post(
    "https://api.fastrouter.ai/api/v1/systemone",
    headers={
        "Authorization": f"Bearer {os.environ['FASTROUTER_API_KEY']}",
        "Content-Type": "application/json",
    },
    json={
        "model": "typesafe/jev-latest",  # or "gpt-6-luna", "gemini-decision-1"
        "state": "Help! My payouts have been failing for 3 days.",
        "questions": {
            "is_urgent": {
                "type": "noul",
                "instructions": "Does this convey urgency?",
            }
        },
    },
    timeout=30,
)
response.raise_for_status()
result = response.json()

print(result["answers"]["is_urgent"]["noul"])  # e.g. 0.95
```
{% endtab %}

{% tab title="Node.JS" %}
```typescript
const response = await fetch("https://api.fastrouter.ai/api/v1/systemone", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FASTROUTER_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "typesafe/jev-latest", // or "gpt-6-luna", "gemini-decision-1"
    state: "Help! My payouts have been failing for 3 days.",
    questions: {
      is_urgent: {
        type: "noul",
        instructions: "Does this convey urgency?",
      },
    },
  }),
});

if (!response.ok) {
  throw new Error(`Request failed: ${response.status} ${await response.text()}`);
}

const result = await response.json();
console.log(result.answers.is_urgent.noul); // e.g. 0.95
```
{% endtab %}
{% endtabs %}

**Response**

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "is_urgent": {
      "type": "noul",
      "noul": 0.95
    }
  },
  "usage": { "input_tokens": 296, "output_tokens": 20 }
}
```

**Using other models**

Change `model` and keep the rest of the request.

{% tabs %}
{% tab title="gemini-decision-1" %}
```bash
curl https://api.fastrouter.ai/api/v1/systemone \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-decision-1",
    "state": "The blue mug I ordered on Monday arrived cracked on Wednesday.",
    "questions": {
      "is_damaged": {
        "type": "noul",
        "instructions": "Did the item arrive damaged?"
      }
    }
  }'
```
{% endtab %}

{% tab title="gpt-6-luna" %}
```bash
curl https://api.fastrouter.ai/api/v1/systemone \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-6-luna",
    "state": "The blue mug I ordered on Monday arrived cracked on Wednesday.",
    "questions": {
      "is_damaged": {
        "type": "noul",
        "instructions": "Did the item arrive damaged?"
      }
    }
  }'
```
{% endtab %}

{% tab title="typesafe/jev-1.14.0" %}
```bash
curl https://api.fastrouter.ai/api/v1/systemone \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "typesafe/jev-1.14.0",
    "state": "The blue mug I ordered on Monday arrived cracked on Wednesday.",
    "questions": {
      "is_damaged": {
        "type": "noul",
        "instructions": "Did the item arrive damaged?"
      }
    }
  }'
```
{% endtab %}
{% endtabs %}

The answer comes back in the SystemOne shape whichever model serves it:

```json
"is_damaged": { "type": "noul", "noul": 0.98 }
```

**Gemini request format**

This endpoint also accepts Google's Gemini Decisions request format, with any decision model. It is close to the SystemOne format, with `input` in place of `state` and `true_description` / `false_description` as the keys of `noul` criteria. Send existing Gemini requests unchanged, and change `model` to run them on Jev or `gpt-6-luna`.

```bash
curl https://api.fastrouter.ai/api/v1/systemone \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "typesafe/jev-latest",
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

A Gemini-format request returns a **Gemini-format response** by default, even on this endpoint. It is keyed by question name like a SystemOne response, but `noul` answers also carry `confidence`, and `usage` uses Gemini's field names:

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

To get a SystemOne-format response instead, add `"response_format": "systemone"`.&#x20;

***

#### Core concepts

| Concept       | What it is                                                                                                                                            |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **State**     | The content being evaluated: a string, or structured JSON (object/array) such as a chat log or record. With Jev models, the state can include images. |
| **Questions** | A map of named, typed questions. You choose each key.                                                                                                 |
| **Answers**   | One typed answer per question, returned under the same key you used.                                                                                  |
| **Assets**    | Images in the request, each listed once and referenced by a string wherever it belongs. See [Images](images.md).                                      |

Question keys are identifiers only. They are **not** sent to the model and do not affect inference. Put everything the model needs into `instructions` and `criteria`.

***

#### Request body

| Field             | Type                      | Required | Description                                                                                                  |
| ----------------- | ------------------------- | -------- | ------------------------------------------------------------------------------------------------------------ |
| `model`           | string                    | Yes      | Any model from [Supported models](systemone-endpoint.md#supported-models).                                   |
| `state`           | string \| object \| array | Yes      | The content to evaluate.                                                                                     |
| `questions`       | map\<string, Question>    | Yes      | Named questions. Each value is a `noul`, `choice`, or `score` question.                                      |
| `assets`          | map\<string, Asset>       | No       | Images referenced in `state`, `instructions`, or `criteria`. See [Images](https://claude.ai/chat/images.md). |
| `response_format` | string                    | No       | `"systemone"` (default) or `"decisions"`. See [Response formats](systemone-endpoint.md#response-formats).    |

This endpoint also accepts the Gemini request format, which uses `input` instead of `state`, with any model.

Every question has a `type` and `instructions`. `instructions` (and criteria values) can be a string, object, or array.

***

#### Question types

**Noul: yes/no**

Returns the probability (0 to 1) that the answer is **yes**.

| Field            | Type                      | Required | Description                        |
| ---------------- | ------------------------- | -------- | ---------------------------------- |
| `type`           | `"noul"`                  | Yes      |                                    |
| `instructions`   | string \| object \| array | Yes      | The yes/no question.               |
| `criteria.true`  | string \| object \| array | No       | What a "yes" (value near 1) means. |
| `criteria.false` | string \| object \| array | No       | What a "no" (value near 0) means.  |

```json
"is_urgent": {
  "type": "noul",
  "instructions": "Does this convey urgency?",
  "criteria": {
    "true": "Explicitly time-sensitive",
    "false": "No urgency expressed"
  }
}
```

**Answer**

```json
"is_urgent": { "type": "noul", "noul": 0.95 }
```

**Choice: pick one option**

Selects one option from a set you define and returns the full probability distribution.

| Field          | Type                                            | Required | Description                                                                                         |
| -------------- | ----------------------------------------------- | -------- | --------------------------------------------------------------------------------------------------- |
| `type`         | `"choice"`                                      | Yes      |                                                                                                     |
| `instructions` | string \| object \| array                       | Yes      | What the model should decide.                                                                       |
| `criteria`     | map\<string, string \| object \| array \| null> | Yes      | Option → description. Use `null` for self-explanatory options. Max **255** options with Jev models. |

```json
"department": {
  "type": "choice",
  "instructions": "Which team should handle this?",
  "criteria": {
    "billing": "Payments, invoicing, refunds",
    "technical": "Bugs, outages, integrations",
    "sales": "Pricing, upgrades, new accounts"
  }
}
```

**Answer**

```json
"department": {
  "type": "choice",
  "choice": "billing",
  "probabilities": { "billing": 0.88, "technical": 0.12, "sales": 0.0 },
  "confidence": 0.81
}
```

| Field           | Description                                         |
| --------------- | --------------------------------------------------- |
| `choice`        | The highest-probability option.                     |
| `probabilities` | Every option mapped to its probability (sums to 1). |
| `confidence`    | 0–1 certainty derived from the distribution.        |

**Score: rate on a rubric**

Rates the state along an ordered scale and returns a probability-weighted score.

| Field          | Type                              | Required | Description                                                                          |
| -------------- | --------------------------------- | -------- | ------------------------------------------------------------------------------------ |
| `type`         | `"score"`                         | Yes      |                                                                                      |
| `instructions` | string \| object \| array         | Yes      | What the model should rate.                                                          |
| `criteria`     | array\<string \| object \| array> | Yes      | Ordered level descriptions, lowest first. Minimum 2; maximum **10** with Jev models. |

```json
"frustration": {
  "type": "score",
  "instructions": "How frustrated is the customer?",
  "criteria": ["Calm", "Frustrated", "Very angry"]
}
```

**Answer**

```json
"frustration": {
  "type": "score",
  "score": 1.05,
  "legend": { "0": "Calm", "1": "Frustrated", "2": "Very angry" },
  "probabilities": { "0": 0.0, "1": 0.95, "2": 0.05 },
  "confidence": 0.92
}
```

| Field           | Description                                                                    |
| --------------- | ------------------------------------------------------------------------------ |
| `score`         | Probability-weighted value across levels (0-indexed); can fall between levels. |
| `legend`        | Level index → your level description.                                          |
| `probabilities` | Level index → probability (sums to 1).                                         |
| `confidence`    | 0–1 certainty derived from the distribution.                                   |

***

#### Asking multiple questions in one request

Batch every question you need about the same state into a single call. Each answer comes back under its own key. `gemini-decision-1` accepts up to 1,000 questions per request.

```python
import os
import requests

payload = {
    "model": "typesafe/jev-latest",
    "state": "Help! My payouts have been failing for 3 days.",
    "questions": {
        "is_urgent": {
            "type": "noul",
            "instructions": "Does this convey urgency?",
        },
        "department": {
            "type": "choice",
            "instructions": "Which team should handle this?",
            "criteria": {
                "billing": "Payments, invoicing, refunds",
                "technical": "Bugs, outages, integrations",
                "sales": "Pricing, upgrades, new accounts",
            },
        },
        "frustration": {
            "type": "score",
            "instructions": "How frustrated is the customer?",
            "criteria": ["Calm", "Frustrated", "Very angry"],
        },
    },
}

resp = requests.post(
    "https://api.fastrouter.ai/api/v1/systemone",
    headers={"Authorization": f"Bearer {os.environ['FASTROUTER_API_KEY']}"},
    json=payload,
    timeout=30,
)
resp.raise_for_status()
answers = resp.json()["answers"]

if answers["is_urgent"]["noul"] > 0.8:
    print(f"Escalate to {answers['department']['choice']} "
          f"(frustration {answers['frustration']['score']:.2f})")
```

***

#### Structured state

`state` can be structured JSON, which is useful for conversations, records, or application state.

```json
{
  "model": "typesafe/jev-latest",
  "state": {
    "channel": "support_chat",
    "messages": [
      { "role": "customer", "text": "I was charged twice for my subscription." },
      { "role": "agent", "text": "Sorry about that — let me check." },
      { "role": "customer", "text": "It's been a week. I want a refund today." }
    ]
  },
  "questions": {
    "refund_requested": {
      "type": "noul",
      "instructions": "Has the customer explicitly asked for a refund in `messages`?"
    }
  }
}
```

Refer to fields of the state by name in backticks (e.g. `` `messages` ``) to point a question at a specific part of it.

#### Structured instructions

When a question needs extra context or reference data, pass `instructions` as an object: put the question in one field and the data in others, and reference those fields in backticks.

```json
{
  "model": "typesafe/jev-latest",
  "state": "Resume: John Smith, Oakland CA. Senior SWE at Google (2019–2025)...",
  "questions": {
    "is_duplicate": {
      "type": "noul",
      "instructions": {
        "potential_duplicate": {
          "name": "John Smith",
          "location": "Oakland, California",
          "last_employer": "Google"
        },
        "question": "Is the resume for the same person as `potential_duplicate`?"
      }
    }
  }
}
```

***

#### Images

With Jev models, an image can go anywhere text can go: in the `state`, in a question's `instructions`, or in its `criteria`. List each image once in the top-level `assets` map and write its reference string where the image belongs.

```json
{
  "model": "typesafe/jev-latest",
  "state": { "photo": "a3f1c2e8-found" },
  "assets": {
    "a3f1c2e8-found": { "type": "image", "format": "jpg", "data_base64": "/9j/4AAQSkZJRg..." }
  },
  "questions": {
    "has_collar": {
      "type": "noul",
      "instructions": "Is the dog in `photo` wearing a collar?"
    }
  }
}
```

Send the `TypeSafe-Danger-Zone-Images: true` header with every request that contains an image. See [Images](images.md) for placement in instructions and criteria, model support, a full Python example, and limits.

***

#### Response formats

By default, a SystemOne-format request returns the **SystemOne** format shown above, and a Gemini-format request returns the Gemini format. Set `response_format` to `"decisions"` to receive the same answers in the **OpenAI Decisions** format. This is useful if your code already parses [Decisions](https://claude.ai/chat/decisions-endpoint.md) responses.

| `response_format`       | `answers` shape                                      | Choice/score `probabilities`                                         |
| ----------------------- | ---------------------------------------------------- | -------------------------------------------------------------------- |
| `"systemone"` (default) | Map keyed by question id                             | Map of option or level index → probability, plus `legend` for scores |
| `"decisions"`           | Array; each answer carries `name` (your question id) | Array of `{ value, label?, probability }`                            |

`response_format` changes only the **response** shape. The request body still uses the SystemOne schema (`state`, a `questions` map, `criteria`).

**Request**

```bash
curl https://api.fastrouter.ai/api/v1/systemone \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "typesafe/jev-latest",
    "state": "Help! My payouts have been failing for 3 days.",
    "response_format": "decisions",
    "questions": {
      "frustration": {
        "type": "score",
        "instructions": "How frustrated is the customer?",
        "criteria": ["Calm", "Frustrated", "Very angry"]
      }
    }
  }'
```

**Response**

```json
{
  "answers": [
    {
      "type": "score",
      "name": "frustration",
      "score": 1.05,
      "probabilities": [
        { "value": 0, "label": "Calm", "probability": 0.0 },
        { "value": 1, "label": "Frustrated", "probability": 0.95 },
        { "value": 2, "label": "Very angry", "probability": 0.05 }
      ],
      "confidence": 0.92
    }
  ],
  "model": "jev-1.13.0",
  "usage": { "input_tokens": 304, "output_tokens": 18 }
}
```

**SystemOne vs. Decisions field mapping**

<table data-header-hidden data-search="false"><thead><tr><th></th><th></th><th></th></tr></thead><tbody><tr><td>Concept</td><td><code>/systemone</code> request</td><td><code>/decisions</code> request</td></tr><tr><td>Content to evaluate</td><td><code>state</code></td><td><code>input</code></td></tr><tr><td>Question list</td><td><code>questions</code> map</td><td><code>questions</code> array</td></tr><tr><td>Question identifier</td><td>Map key</td><td><code>name</code></td></tr><tr><td>Yes/no question</td><td><code>noul</code></td><td><code>predicate</code></td></tr><tr><td>Choice options</td><td><code>criteria</code> map (option → description)</td><td><code>choices[]</code> (<code>value</code>, <code>description</code>)</td></tr><tr><td>Score levels</td><td><code>criteria</code> array</td><td><code>levels[]</code> (<code>label</code>, <code>description</code>)</td></tr><tr><td>Images</td><td><code>assets</code> map, referenced by string</td><td><code>input_image</code> parts in <code>input</code> (data URLs)</td></tr></tbody></table>

| Answer field (yes/no) | SystemOne format | Decisions format |
| --------------------- | ---------------- | ---------------- |
| Answer type           | `"noul"`         | `"predicate"`    |
| Probability of yes    | `noul`           | `probability`    |

***

#### Response body

| Field                 | Type                 | Description                                                                  |
| --------------------- | -------------------- | ---------------------------------------------------------------------------- |
| `model`               | string               | The exact model version that ran the evaluation.                             |
| `answers`             | map\<string, Answer> | One answer per question, under the keys you supplied.                        |
| `usage.input_tokens`  | integer              | Input tokens consumed, including image tokens.                               |
| `usage.output_tokens` | integer              | Output tokens consumed. Always `0` for `gpt-6-luna` and `gemini-decision-1`. |

Usage is billed to your FastRouter account and shown in the Activity Log. See [Decision Models → Pricing](https://claude.ai/chat/README.md#pricing).

***

#### Best practices

* **Describe the edges.** Add `criteria` to noul questions when "yes" is ambiguous; it noticeably sharpens results.
* **Use thresholds, not just the top answer.** `noul`, `probabilities`, and `confidence` let you route low-confidence cases to a human or a stronger model.
* **Re-check thresholds when you change models.** Probabilities from different models are not interchangeable. Pin a Jev version (`typesafe/jev-1.14.0`) once thresholds are tuned.
* **Batch questions.** Asking several questions about one state in a single request is cheaper and faster than separate calls.
* **Keep keys meaningless to the model.** Keys are never seen by the model; all meaning must live in `instructions` and `criteria`.

#### Common use cases

* **Support triage:** urgency (noul), department routing (choice), sentiment (score).
* **Content moderation:** policy-violation checks with calibrated probabilities.
* **Image QA and matching:** compare a photo against a known reference picture (Jev models).
* **LLM output grading:** score responses from other models on accuracy, tone, or completeness.
* **Deduplication and matching:** compare records using structured instructions.
* **Agent guardrails:** check agent state before allowing an action.
