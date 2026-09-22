---
description: >-
  Evaluate text or structured data against typed yes/no, choice, and score
  questions using TypeSafe's Jev models through FastRouter.
---

# TypeSafe Jev — Structured Evaluation

***

## Introduction

FastRouter now supports TypeSafe's **Jev** models through the `systemone` evaluation endpoint. Instead of prompting a chat model and parsing free-form text, you send a piece of content (the **state**) along with a set of **typed questions**, and get back **structured, probability-calibrated answers** — one per question.

Use it for classification, routing, triage, moderation, quality scoring, deduplication, and any workflow where you need a reliable, machine-readable decision rather than prose.

**Why use Jev through FastRouter**

* **One API key** — use your existing FastRouter key; no separate TypeSafe account required.
* **Unified billing and usage** — Jev usage appears alongside all your other model spend.
* **Observability** — every request is visible in the Activity Log with tokens, latency, and cost.
* **Structured by design** — answers are typed (`noul`, `choice`, `score`) and include probabilities and confidence, so no output parsing is needed.

***

## Endpoint

```http
POST https://api.fastrouter.ai/api/v1/systemone
Authorization: Bearer <FASTROUTER_API_KEY>
Content-Type: application/json
```

## Available models

| Model                 | Description                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `typesafe/jev-latest` | TypeSafe's flagship evaluation model (alias that always points to the latest Jev release). |

The response's `model` field reports the exact version that served the request (for example `jev-1.13.0`).

***

### Quickstart

#### cURL

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

#### Python

```python
import os
import requests

response = requests.post(
    "https://api.fastrouter.ai/api/v1/systemone",
    headers={
        "Authorization": f"Bearer {os.environ['FASTROUTER_API_KEY']}",
        "Content-Type": "application/json",
    },
    json={
        "model": "typesafe/jev-latest",
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

#### Node.js / TypeScript

```typescript
const response = await fetch("https://api.fastrouter.ai/api/v1/systemone", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FASTROUTER_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "typesafe/jev-latest",
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

#### Response

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

***

## Core Concepts

| Concept       | What it is                                                                                              |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| **State**     | The content being evaluated — a string, or structured JSON (object/array) such as a chat log or record. |
| **Questions** | A map of named, typed questions. You choose each key.                                                   |
| **Answers**   | One typed answer per question, returned under the same key you used.                                    |

Question keys are identifiers only — they are **not** sent to the model and do not affect inference. Put everything the model needs into `instructions` and `criteria`.

***

## Request Body

| Field       | Type                      | Required | Description                                                             |
| ----------- | ------------------------- | -------- | ----------------------------------------------------------------------- |
| `model`     | string                    | Yes      | The Jev model to use, e.g. `typesafe/jev-latest`.                       |
| `state`     | string \| object \| array | Yes      | The content to evaluate.                                                |
| `questions` | map\<string, Question>    | Yes      | Named questions. Each value is a `noul`, `choice`, or `score` question. |

Every question has a `type` and `instructions`. `instructions` (and criteria values) can be a string, object, or array — see [Structured instructions](https://claude.ai/chat/1394a175-27d7-41bf-9c69-db45cb8377c5#structured-instructions).

***

## Question Types

#### Noul — yes/no

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

#### Choice — pick one option

Selects one option from a set you define and returns the full probability distribution.

| Field          | Type                                            | Required | Description                                                                         |
| -------------- | ----------------------------------------------- | -------- | ----------------------------------------------------------------------------------- |
| `type`         | `"choice"`                                      | Yes      |                                                                                     |
| `instructions` | string \| object \| array                       | Yes      | What the model should decide.                                                       |
| `criteria`     | map\<string, string \| object \| array \| null> | Yes      | Option → description. Use `null` for self-explanatory options. Max **255** options. |

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

#### Score — rate on a rubric

Rates the state along an ordered scale and returns a probability-weighted score.

| Field          | Type                              | Required | Description                                                          |
| -------------- | --------------------------------- | -------- | -------------------------------------------------------------------- |
| `type`         | `"score"`                         | Yes      |                                                                      |
| `instructions` | string \| object \| array         | Yes      | What the model should rate.                                          |
| `criteria`     | array\<string \| object \| array> | Yes      | Ordered level descriptions, lowest first. Minimum 2, maximum **10**. |

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

## Asking Multiple Questions In One Request

Batch every question you need about the same state into a single call. Each answer comes back under its own key.

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

## Structured State

`state` can be structured JSON — useful for conversations, records, or application state.

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

## Structured Instructions

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

## Response Body

| Field                 | Type                 | Description                                           |
| --------------------- | -------------------- | ----------------------------------------------------- |
| `model`               | string               | The exact model version that ran the evaluation.      |
| `answers`             | map\<string, Answer> | One answer per question, under the keys you supplied. |
| `usage.input_tokens`  | integer              | Input tokens consumed.                                |
| `usage.output_tokens` | integer              | Output tokens consumed.                               |

Usage is billed to your FastRouter account and shown in the Activity Log.

***

## Best Practices

* **Describe the edges.** Add `criteria` to noul questions when "yes" is ambiguous; it noticeably sharpens results.
* **Use thresholds, not just the top answer.** `noul`, `probabilities`, and `confidence` let you route low-confidence cases to a human or a stronger model.
* **Batch questions.** Asking several questions about one state in a single request is cheaper and faster than separate calls.
* **Keep keys meaningless to the model.** Keys are never seen by the model — all meaning must live in `instructions` and `criteria`.

## Common Use Cases

* **Support triage** — urgency (noul), department routing (choice), sentiment (score).
* **Content moderation** — policy-violation checks with calibrated probabilities.
* **LLM output grading** — score responses from other models on accuracy, tone, or completeness.
* **Deduplication and matching** — compare records using structured instructions.
* **Agent guardrails** — check agent state before allowing an action.
