---
description: >-
  FastRouter can return Reasoning Tokens (also known as thinking tokens) for
  supported models.
icon: lightbulb-message
---

# Reasoning Tokens

## Reasoning Tokens

#### Overview

FastRouter can return Reasoning Tokens (also known as _thinking tokens_) for supported models. These tokens represent the model's internal reasoning process and can significantly improve output quality for complex tasks such as planning, math, tool use, and multi-step analysis.

* Reasoning tokens are **enabled by default** for supported models.
* The model decides whether to generate reasoning tokens unless explicitly controlled.
* When returned, reasoning tokens appear in the **`reasoning` field** of each message.
* You can **limit**, **control**, or **exclude** reasoning tokens using the `reasoning` parameter.

***

#### Supported Models

**Reasoning Token Support**

Reasoning tokens are currently supported by:

* **Gemini thinking models** (includes Gemini 2.5 and Gemini 3 series)
* **Anthropic models** (via `reasoning.max_tokens`)
* **OpenAI o-series models**
* **Grok models**

***

#### How Reasoning Tokens Appear in Responses

When enabled, reasoning tokens appear as structured blocks in the response:

```json
{
 "type": "reasoning",
 "reasoning": {
 "text": "The model is considering multiple constraints before responding..."
 }
}
```

If excluded, the model still reasons internally—but the reasoning is **not returned**.

***

#### Controlling Reasoning Tokens

You can control reasoning behavior using the `reasoning` object in your request.

**General Structure**

```json
{
 "model": "your-model",
 "messages": [],
 "reasoning": {
 "effort": "high",
 "max_tokens": 2000,
 "exclude": false,
 "enabled": true
 }
}
```

> ⚠️ Use **either** `effort` **or** `max_tokens` — not both.

***

#### Reasoning Effort Levels

**Supported By**

* **OpenAI o-series**
* **Grok models**
* **Google Gemini 3 models** (mapped to `thinkingLevel`)

**Effort Options**

| Effort   | Token Allocation        |
| -------- | ----------------------- |
| `high`   | \\\~80% of `max_tokens` |
| `medium` | \\\~50% of `max_tokens` |
| `low`    | \\\~20% of `max_tokens` |

Example:

```json
"reasoning\": {
 "effort": "high"
}
```

***

#### Reasoning Max Tokens

**Supported By**

* **Gemini 2.5 thinking models** (via `thinkingBudget`)
* **Anthropic models**

Example:

```json
\"reasoning\": {
 \"max_tokens\": 2000
}
```

***

#### Google Gemini Reasoning Behavior

Google Gemini models support reasoning tokens, but the API used depends on the model generation.

**Gemini 2.5 Models — `thinkingBudget` API**

Gemini 2.5 thinking models use Google's `thinkingBudget` API. With FastRouter, you control this using `reasoning.max_tokens`, which is passed through as the thinking budget.

Example:

```json
\"reasoning\": {
 \"max_tokens\": 2000
}
```

**Gemini 3 Models — `thinkingLevel` API**

Gemini 3 models (such as `google/gemini-3.1-pro-preview` and `google/gemini-3-flash-preview`) use Google's newer `thinkingLevel` API instead of the older `thinkingBudget` API used by Gemini 2.5 models.

FastRouter maps the `reasoning.effort` parameter directly to Google's `thinkingLevel` values:

| FastRouter reasoning.effort | Google thinkingLevel |
| --------------------------- | -------------------- |
| `"minimal"`                 | `"minimal"`          |
| `"low"`                     | `"low"`              |
| `"medium"`                  | `"medium"`           |
| `"high"`                    | `"high"`             |

Example:

```json
"reasoning": {
 "effort": "high"
}
```

**Token Consumption is Determined by Google**

When using `thinkingLevel`, the actual number of reasoning tokens consumed is determined internally by Google. There are no publicly documented token limit breakpoints for each level. For example, setting `effort: \"low\"` might result in several hundred reasoning tokens depending on the complexity of the task. This is expected behavior and reflects how Google implements thinking levels internally.

***

#### Anthropic-Specific Reasoning Behavior

When using **Anthropic models**:

**Rules**

* `reasoning.max_tokens`
* Used directly
* Minimum: **1024 tokens**
* `reasoning.effort`
* Converted into a reasoning token budget
* Reasoning tokens are:
* **Minimum:** 1024 tokens
* **Maximum:** 32,000 tokens

**Budget Formula**

```
budget_tokens = max(
 min(max_tokens × effort_ratio, 32000),
 1024
)
```

Where:

* `high` → 0.8
* `medium` → 0.5
* `low` → 0.2

**Important Constraint**

> **`max_tokens` must be strictly greater than the reasoning budget**, otherwise the model will not have enough tokens to produce a final answer.

***

#### Excluding Reasoning Tokens

You can instruct the model to reason internally **without returning reasoning tokens**.

```json
"reasoning": {
 "exclude": true
}
```

* The model still performs reasoning
* Reasoning tokens are **not included** in the response
* Works across **all models**

***

#### Token Usage & Billing

* Reasoning tokens are counted as **output tokens**
* They are billed the same way as regular output tokens
* Enabling reasoning increases token usage but often improves: Accuracy, Coherence, Tool-calling correctness
