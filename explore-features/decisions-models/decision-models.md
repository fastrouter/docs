---
description: >-
  Get typed yes/no probabilities, choices, and rubric scores from OpenAI, Google
  Gemini, and TypeSafe decision models through FastRouter.
icon: gavel
---

# Decision Models

### Introduction

Decision models answer **closed questions** about a piece of content instead of generating text. You send the content (text, images, or both) together with a set of **typed questions**, and get back one structured answer per question: the probability that a condition is true, one option from a fixed set, or a score against a rubric.

Use them to classify content, route requests, moderate images, triage issues, grade other models' output, and guard agent actions: anywhere your application needs a decision rather than prose.

FastRouter exposes decision models through two endpoints. Each accepts a different request schema, and **every decision model works on both endpoints**:

| Endpoint                 | Request schema                                | Default response format | Guide              |
| ------------------------ | --------------------------------------------- | ----------------------- | ------------------ |
| `POST /api/v1/decisions` | OpenAI Decisions (`input`, `questions` array) | Decisions               | Decisions endpoint |
| `POST /api/v1/systemone` | TypeSafe SystemOne (`state`, `questions` map) | SystemOne               | SystemOne endpoint |

Pick the endpoint whose request schema matches your code, then switch models by changing `model`. No other change to the request is needed.

Both endpoints also accept Google's **Gemini Decisions request format**, with any decision model. See [Request formats](decision-models.md#request-formats).

**Why use decision models through FastRouter**

* **One API key:** use your existing FastRouter key for OpenAI, Gemini, and TypeSafe models, with no separate provider accounts.
* **One schema, any model:** send a Decisions-shaped or SystemOne-shaped request to any supported model.
* **Switchable response format:** receive answers in either format with `response_format`, whichever endpoint you call.
* **Unified billing and observability:** every request appears in the Activity Log with tokens, latency, and cost, alongside the rest of your model spend.

***

### Supported models

| Model                 | Provider | Input        | Notes                                                                              |
| --------------------- | -------- | ------------ | ---------------------------------------------------------------------------------- |
| `gpt-6-luna`          | OpenAI   | Text, images | OpenAI's decision model. OpenAI's Decisions API is in public beta.                 |
| `gemini-decision-1`   | Google   | Text         | Google's decision model. 262,144-token context; up to 1,000 questions per request. |
| `typesafe/jev-latest` | TypeSafe | Text, images | Alias that always points to the latest Jev release.                                |
| `typesafe/jev-1.14.0` | TypeSafe | Text, images | Pinned Jev 1.14 release.                                                           |
| `typesafe/jev-1.13.0` | TypeSafe | Text, images | Pinned Jev 1.13 release.                                                           |

\{% hint style="info" %\} **Pin a version once you have tuned thresholds.** `typesafe/jev-latest` moves to each new Jev release, which can shift probabilities. If your routing depends on thresholds set against labeled examples, use a pinned model such as `typesafe/jev-1.14.0`. The response's `model` field always reports the exact version that served the request. \{% endhint %\}

Image input is covered in [Images](images.md).

***

### Choosing an endpoint

| If you…                                                                                       | Use                                                                  |
| --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Already parse OpenAI Decisions requests or responses                                          | [`/decisions`](decisions-endpoint.md)                                |
| Already use TypeSafe's SystemOne schema or SDK                                                | [`/systemone`](systemone-endpoint.md)                                |
| Want to send structured JSON state (chat logs, records) and point questions at fields by name | [`/systemone`](systemone-endpoint.md)                                |
| Want to put images inside a question's instructions or criteria (Jev models)                  | [`/systemone`](systemone-endpoint.md)                                |
| Are starting fresh with plain text or a single image                                          | Either. [`/decisions`](decisions-endpoint.md) is the simpler schema. |
| Already build requests for Google's Gemini Decisions API                                      | Either. Both accept the Gemini format, with any model.               |

***

### Request formats

FastRouter accepts three request formats. Every format works with every decision model.

| Request format | Shape                                                            | Accepted on                   | Default response format |
| -------------- | ---------------------------------------------------------------- | ----------------------------- | ----------------------- |
| Decisions      | `input` + `questions` **array** (`predicate`, `choice`, `score`) | `/decisions`                  | Decisions               |
| SystemOne      | `state` + `questions` **map** (`noul`, `choice`, `score`)        | `/systemone`                  | SystemOne               |
| Gemini         | `input` + `questions` **map** (`noul`, `choice`, `score`)        | `/decisions` and `/systemone` | Gemini                  |

The Gemini format is the one Google's Gemini Decisions API uses. It combines `input` for the content with a `questions` map keyed by question name. Each question has a `type`, `instructions`, and `criteria`:

* **`noul`:** `criteria` is optional and can include `true_description` and `false_description`.
* **`choice`:** `criteria` maps option IDs to descriptions.
* **`score`:** `criteria` is an ordered array of level descriptions, lowest first.

If your code already builds Gemini requests, send them to FastRouter unchanged, with any model:

```json
{
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
}
```

Change `model` to `gpt-6-luna` or `typesafe/jev-latest` to send the same request to another model. The response comes back in the Gemini format unless you set `response_format`.

**Gemini-format response**

```json
{
  "answers": {
    "is_damaged": {
      "type": "noul",
      "noul": 0.97,
      "confidence": 0.97
    },
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

Answers are keyed by question name, as in the SystemOne format. The differences from SystemOne are that `noul` answers also carry `confidence` (equal to `max(noul, 1 - noul)`), and `usage` reports `total_input_tokens`, `input_tokens_by_modality`, `total_output_tokens`, and `total_tokens`. A `score` answer carries `score`, `legend`, `probabilities` per level, and `confidence`.

***

### Question types

Both endpoints support the same three question types under different names.

| Concept | `/decisions` type | `/systemone` type | Main result                                                               |
| ------- | ----------------- | ----------------- | ------------------------------------------------------------------------- |
| Yes/no  | `predicate`       | `noul`            | Probability (0 to 1) that the condition is true.                          |
| Choice  | `choice`          | `choice`          | One option from the set you supply, with a full probability distribution. |
| Score   | `score`           | `score`           | Probability-weighted average of 0-indexed levels, with a distribution.    |

Use `choice` for categories with no order and `score` for ordered levels. Both return a `confidence` value from 0 to 1 derived from the distribution.

***

### Response formats

By default, **the response format follows the request format**, whichever model serves the request and whichever endpoint you call:

| Request format | Default response format |
| -------------- | ----------------------- |
| Decisions      | Decisions               |
| SystemOne      | SystemOne               |
| Gemini         | Gemini                  |

Set `response_format` to override the default:

| `response_format` | Accepted aliases         | `answers` shape                   | Choice/score `probabilities`                                         |
| ----------------- | ------------------------ | --------------------------------- | -------------------------------------------------------------------- |
| `"decisions"`     | `decision`, `openai`     | Array; each answer carries `name` | Array of `{ value, label?, probability }`                            |
| `"systemone"`     | `system_one`, `typesafe` | Map keyed by question name        | Map of option or level index → probability, plus `legend` for scores |

For example, a Gemini-format request sent to `typesafe/jev-latest` returns a Gemini-format response, and the same request with `"response_format": "decisions"` returns a Decisions-format response.

`response_format` changes only the **response**. The request body must still use the endpoint's own format or the Gemini format.

#### Field mapping

<table data-search="false"><thead><tr><th>Concept</th><th>Decisions format</th><th>SystemOne format</th><th>Gemini format</th></tr></thead><tbody><tr><td>Content to evaluate</td><td><code>input</code></td><td><code>state</code></td><td><code>input</code></td></tr><tr><td>Question list</td><td><code>questions</code> array</td><td><code>questions</code> map</td><td><code>questions</code> map</td></tr><tr><td>Question identifier</td><td><code>name</code></td><td>Map key</td><td>Map key</td></tr><tr><td>Yes/no question</td><td><code>predicate</code></td><td><code>noul</code></td><td><code>noul</code></td></tr><tr><td>Yes/no criteria</td><td>—</td><td><code>criteria.true</code>, <code>criteria.false</code></td><td><code>criteria.true_description</code>, <code>criteria.false_description</code></td></tr><tr><td>Choice options</td><td><code>choices[]</code> (<code>value</code>, <code>description</code>)</td><td><code>criteria</code> map (option → description)</td><td><code>criteria</code> map (option → description)</td></tr><tr><td>Score levels</td><td><code>levels[]</code> (<code>label</code>, <code>description</code>)</td><td><code>criteria</code> array</td><td><code>criteria</code> array</td></tr><tr><td>Images</td><td><code>input_image</code> parts in <code>input</code> (data URLs)</td><td><code>assets</code> map, referenced anywhere in the request</td><td>Not supported</td></tr><tr><td>Yes/no answer field</td><td><code>probability</code></td><td><code>noul</code></td><td><code>noul</code> and <code>confidence</code></td></tr><tr><td>Token usage fields</td><td><code>input_tokens</code>, <code>output_tokens</code>, <code>total_tokens</code>, <code>cost</code></td><td><code>input_tokens</code>, <code>output_tokens</code></td><td><code>total_input_tokens</code>, <code>input_tokens_by_modality</code>, <code>total_output_tokens</code>, <code>total_tokens</code></td></tr></tbody></table>

***

### Pricing

| Model               | Input                                              | Output                 |
| ------------------- | -------------------------------------------------- | ---------------------- |
| `gpt-6-luna`        | $0.10 per 1M tokens                                | None (always 0 tokens) |
| `gemini-decision-1` | $0.10 per 1M tokens                                | None (always 0 tokens) |
| `typesafe/jev-*`    | See the model page in the FastRouter model catalog | See the model page     |

Images count as input tokens. There are no cache-read or cache-write charges. Responses from `/decisions` report the exact charge in `usage.cost`, and every request's cost appears in the Activity Log.

***

### Best practices

* **Write questions around observable criteria.** "Is there a visible crack?" is better than "Is this product bad?"
* **One concern per question.** Split compound checks into separate questions in the same request.
* **Make options distinct.** Choices should not overlap, and adjacent score levels should have clearly different criteria.
* **Include a fallback option** such as `"other"` when your choices might not cover every input, and send those results to a review queue.
* **Batch independent questions** about the same content in one request. It is cheaper and faster than separate calls.
* **Use thresholds, not just the top answer.** Set thresholds on labeled examples from your own application, weighing the cost of false positives against false negatives. Route low-confidence results to a human or a stronger model.
* **Compare models on your own data.** Because every model accepts the same request, you can run one labeled set through `gpt-6-luna`, `gemini-decision-1`, and Jev by changing only `model`.
* **Use Structured Outputs instead** when you need a generated object (extracted fields, written explanations) rather than a probability, choice, or score.

### Common use cases

* **Support triage:** urgency (yes/no), department routing (choice), severity or frustration (score).
* **Image moderation and QA:** damage, policy violations, or product category from photos.
* **Content moderation:** policy checks with calibrated probabilities.
* **LLM output grading:** score other models' responses for accuracy, tone, or completeness.
* **Deduplication and matching:** compare records or photos against a known reference.
* **Agent guardrails:** check a proposed action or agent state before allowing it.
