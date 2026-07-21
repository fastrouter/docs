---
description: >-
  Multi-model deliberation — ask a panel of models the same prompt in parallel,
  then have a judge model compare their answers into a structured analysis.
icon: arrows-to-circle
---

# Blend — Multi-Model Deliberation

### Overview

Blend asks a panel of models the same prompt in parallel, then a judge model compares their answers into a structured analysis: agreements, disagreements, coverage gaps, standout insights, missing considerations, and confidence notes.

Instead of trusting a single model, Blend gathers multiple independent perspectives on your prompt and has a judge model analyze how they compare. The judge does **not** merge the answers — it evaluates them, so you can see where models agree, where they conflict, and what each one uniquely contributed.

* **Parallel panel calls** — the same prompt runs against every panel model at once.
* **Structured judge analysis** — a single JSON object comparing the answers.
* **Aggregated usage & cost** — panel + judge tokens and spend, summed and itemized.
* **Reuses routing, BYOK & credits** — sub-calls run through the normal gateway path.

### Two ways to use Blend

| Method                               | How to trigger                                        | When to use                                                                                                                            |
| ------------------------------------ | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **Model alias** — `fastrouter/blend` | Set `"model": "fastrouter/blend"`.                    | Blend always runs and returns a human-readable summary as the completion content. Best when you want Blend to _be_ the whole response. |
| **Server tool** — `fastrouter:blend` | Attach a `fastrouter:blend` tool to a normal request. | The outer model decides when to call it; the structured analysis is fed back as a tool result so the model writes the final answer.    |

***

### 1. Model alias — `fastrouter/blend`

Send a normal chat completion request with the model set to `fastrouter/blend`:

```bash
curl https://api.fastrouter.ai/v1/chat/completions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "fastrouter/blend",
    "messages": [
      { "role": "user", "content": "Survey the strongest arguments for and against EV cars." }
    ],
    "analysis_models": ["anthropic/claude-sonnet-5", "openai/gpt-5.2", "google/gemini-3-pro"]
  }'
```

The assistant content is a markdown summary: a `## Panel responses` section (one subsection per model) followed by `## Analysis` containing the structured JSON. `analysis_models` is optional — see Panel & judge selection.

### 2. Server tool — `fastrouter:blend`

Attach the hosted `fastrouter:blend` tool to a request that targets any normal model. Blend parameters go inside the tool's `parameters` object.

```bash
curl https://api.fastrouter.ai/v1/chat/completions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "x-ai/grok-4.20-beta",
    "messages": [
      { "role": "user", "content": "Assess the case for and against banning internal combustion engine vehicles by 2035." }
    ],
    "tools": [
      {
        "type": "fastrouter:blend",
        "parameters": {
          "analysis_models": [
            "google/gemini-3-flash-preview",
            "z-ai/glm-5.2",
            "moonshotai/kimi-k2.7-code"
          ],
          "model": "anthropic/claude-sonnet-5"
        }
      }
    ]
  }'
```

**What happens under the hood:**

1. The gateway replaces the hosted tool with a callable function named `fastrouter_blend` (providers reject unknown hosted tool types).
2. The outer model runs normally and decides whether to call `fastrouter_blend`.
3. When it does, the gateway runs the Blend pipeline, injects the structured `BlendResult` JSON as the tool result, and re-calls the model.
4. The model uses that analysis to write the final answer.

> **Model-driven.** With the server tool, Blend runs only if the outer model chooses to call `fastrouter_blend`. For simple prompts a model may answer directly and skip it. Use the **model alias** `fastrouter/blend` at the top level if you want Blend to always run.

***

### Parameters

For the **alias** these go at the top level of the request body. For the **tool** they go inside the tool's `parameters` object.

| Parameter         | Type       | Required | Description                                                                                                                                   |
| ----------------- | ---------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `analysis_models` | `string[]` | No       | The panel models that answer in parallel. Capped at 5. If omitted, the panel is auto-selected from the router's top candidates (capped at 3). |
| `model`           | `string`   | No       | The judge model that compares the panel answers. Defaults to the top-ranked panel model when omitted.                                         |

Everything else in your request — the `messages`, plus any other tools, sampling params, or `response_format` — is forwarded to each panel model. Only the model id is swapped, the Blend tool is stripped, and streaming is turned off for the sub-calls.

### Panel & judge selection

**Panel models**

* **Supplied:** your `analysis_models` are used as-is, in order, de-duplicated, capped at `MAX_BLEND_ANALYSIS_MODELS = 5`. Supplying 2 uses exactly 2.
* **Auto:** if you supply none, the panel is derived from the router's classifier top candidates, capped at `MAX_BLEND_AUTO_MODELS = 3`.

**Judge model**

* The `model` override when provided.
* Otherwise, the top-ranked panel model.

***

### Response format

#### Model alias — human-readable content

The completion content is markdown, structured like:

```markdown
## Panel responses

### anthropic/claude-sonnet-5
<that model's full answer>

### openai/gpt-5.2
<that model's full answer>

**Tool calls**    ← only shown if a panel answered with tool calls
[ { "id": "call_1", "type": "function", "function": { ... } } ]

## Analysis
<the structured judge JSON>
```

#### Server tool — structured tool result

The Blend pipeline returns a `BlendResult` JSON object as the tool result. The model then writes the final natural-language answer from it. The object looks like:

```json
{
  "status": "ok",
  "analysis": { /* judge JSON, see below */ },
  "responses": [
    {
      "model": "google/gemini-3-flash-preview",
      "content": "…",
      "tool_calls": [ /* present only if the panel emitted them */ ],
      "usage": { "prompt_tokens": 0, "completion_tokens": 0, "total_tokens": 0, "cost": 0 }
    }
  ],
  "failed_models": [ { "model": "…", "error": "…" } ],
  "panel_models": ["…"],
  "judge_model": "anthropic/claude-sonnet-5",
  "trace_id": "blend-…",
  "usage": { "prompt_tokens": 0, "completion_tokens": 0, "total_tokens": 0, "cost": 0 },
  "judge_usage": { "prompt_tokens": 0, "completion_tokens": 0, "total_tokens": 0, "cost": 0 }
}
```

| Field                          | Description                                                            |
| ------------------------------ | ---------------------------------------------------------------------- |
| `status`                       | `"ok"` or `"error"`.                                                   |
| `analysis`                     | The judge's structured comparison JSON. Omitted if the judge degraded. |
| `responses`                    | The successful panel answers (`content` and/or `tool_calls`).          |
| `failed_models`                | Panel models that errored, with the error message.                     |
| `panel_models` / `judge_model` | Which models were used (observability).                                |
| `trace_id`                     | Shared id correlating every panel + judge sub-call in the run.         |
| `usage` / `judge_usage`        | See Usage & cost.                                                      |

#### Judge output (the `analysis` JSON)

The judge returns only this object — it analyzes, it does not merge:

```json
{
  "agreements": ["points where most or all responses aligned"],
  "disagreements": [
    {
      "topic": "the point of disagreement",
      "positions": [
        { "model": "model identifier", "position": "that model's stance" }
      ]
    }
  ],
  "coverage_gaps": [
    {
      "point": "a relevant point only some responses addressed",
      "covered_by": ["model identifiers that covered it"]
    }
  ],
  "standout_insights": [
    { "model": "model identifier", "insight": "something notable only this model raised" }
  ],
  "missing_considerations": ["important aspects no response addressed"],
  "confidence_notes": ["where responses seemed uncertain, hedged, or speculative"]
}
```

***

### Usage & cost

The `usage` field is the sum of every panel call plus the judge call — `prompt_tokens`, `completion_tokens`, `total_tokens`, and `cost`. The `judge_usage` field isolates just the judge call (its numbers are already included in the total).

Panel usage is counted **even for panels that failed** to produce usable content, because the tokens were still spent.

### Errors & graceful degradation

* **Judge fails / returns non-JSON but at least one panel succeeded** → `status` stays `"ok"`, `analysis` is omitted, and the panel responses are still returned.
* **No panel produced output** → `status: "error"` with a `failure_reason`.

| `failure_reason`          | Meaning                                                            |
| ------------------------- | ------------------------------------------------------------------ |
| `all_panels_failed`       | Every panel model errored.                                         |
| `blend_invocation_capped` | Blend was already invoked earlier in this turn (one run per turn). |
| `rate_limited`            | A rate limit was hit.                                              |
| `unexpected_error`        | An unexpected internal error (e.g. no models available).           |

***

### Limits & notes

* **Max panel size:** 5 supplied models, 3 auto-selected.
* Blend runs **at most once per turn** (recursion protection).
* The server-tool path is **non-streaming and top-level only**.
* Panel models receive your other tools too, so a panel may answer with `tool_calls`; those are captured and included in what the judge compares.
* The `fastrouter:blend` server tool is model-driven; use `fastrouter/blend` when you want Blend to always run.
