---
description: >-
  FastRouter can return Reasoning Tokens (also known as thinking tokens) for
  supported models.
icon: lightbulb-message
---

# Reasoning Tokens

### **Overview**

FastRouter can return Reasoning Tokens (also known as _thinking tokens_) for supported models. These tokens represent the model's internal reasoning process and can significantly improve output quality for complex tasks such as planning, math, tool use, and multi-step analysis.

* Reasoning tokens are **enabled by default** for supported models.
* The model decides whether to generate reasoning tokens unless explicitly controlled.
* When returned, reasoning tokens appear in the **`reasoning` field** of each message.
* You can **limit**, **control**, or **exclude** reasoning tokens using the `reasoning` parameter.

***

### **Supported Models**

Reasoning tokens are currently supported by:

* **Gemini thinking models** (includes Gemini 2.5 and Gemini 3 series)
* **Anthropic models** (via `reasoning.max_tokens`)
* **OpenAI o-series models**
* **Grok models**

***

### **How Reasoning Tokens Appear in Responses**

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

### **Controlling Reasoning Tokens**

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

### **Reasoning Effort Levels**

**Supported By**

* **OpenAI o-series**
* **Grok models**
* **Google Gemini 3 models** (mapped to `thinkingLevel`)

**Effort Options**

| Effort     | Token Allocation        |
| ---------- | ----------------------- |
| `max`      | \~90% of `max_tokens`   |
| `xhigh`    | \~85% of `max_tokens`   |
| `high`     | \~80% of `max_tokens`   |
| `trending` | \~80% of `max_tokens`   |
| `medium`   | \~50% of `max_tokens`   |
| `low`      | \~20% of `max_tokens`   |
| `minimal`  | \~10% of `max_tokens`   |
| `none`     | 0% — reasoning disabled |

> **Note:** `trending` currently maps to the same allocation as `high`. It is provided as a separate, forward-compatible label and may be tuned independently in the future — don't assume it will always equal `high`.

Example:

```json
"reasoning": {
 "effort": "high"
}
```

***

### **Reasoning Max Tokens**

**Supported By**

* **Gemini 2.5 thinking models** (via `thinkingBudget`)
* **Anthropic models**

Example:

```json
"reasoning": {
 "max_tokens": 2000
}
```

***

### **Google Gemini Reasoning Behavior**

Google Gemini models support reasoning tokens, but the API used depends on the model generation.

**Gemini 2.5 Models — `thinkingBudget` API**

Gemini 2.5 thinking models use Google's `thinkingBudget` API. With FastRouter, you control this using `reasoning.max_tokens`, which is passed through as the thinking budget.

Example:

```json
"reasoning": {
 "max_tokens": 2000
}
```

**Gemini 3 Models — `thinkingLevel` API**

Gemini 3 models (such as `google/gemini-3.1-pro-preview` and `google/gemini-3-flash-preview`) use Google's newer `thinkingLevel` API instead of the older `thinkingBudget` API used by Gemini 2.5 models.

FastRouter maps the `reasoning.effort` parameter to Google's `thinkingLevel` values as follows:

| FastRouter reasoning.effort | Google thinkingLevel  |
| --------------------------- | --------------------- |
| `max`                       | `high`                |
| `xhigh`                     | `high`                |
| `high`                      | `high`                |
| `trending`                  | `high`                |
| `medium`                    | `medium`              |
| `low`                       | `low`                 |
| `minimal`                   | `minimal`             |
| `none`                      | _(thinking disabled)_ |

> Google's `thinkingLevel` API only exposes four levels (`minimal`, `low`, `medium`, `high`). FastRouter's finer-grained effort levels above `high` (`xhigh`, `max`, `trending`) all map to Google's `high` on Gemini 3 models — the additional granularity only takes effect on models that consume a numeric token budget (Anthropic, Gemini 2.5).

Example:

```json
"reasoning": {
 "effort": "high"
}
```

**Token Consumption is Determined by Google**

When using `thinkingLevel`, the actual number of reasoning tokens consumed is determined internally by Google. There are no publicly documented token limit breakpoints for each level. For example, setting `effort: "low"` might result in several hundred reasoning tokens depending on the complexity of the task. This is expected behavior and reflects how Google implements thinking levels internally.

***

### **Anthropic-Specific Reasoning Behavior**

When using **Anthropic models**:

**Rules**

* `reasoning.max_tokens`
* Used directly
* Minimum: **1024 tokens**
* `reasoning.effort`
* Converted into a reasoning token budget
* Reasoning tokens are:
* **Minimum:** 1024 tokens (except `none`, see below)
* **Maximum:** 24,576 tokens

**Budget Formula**

```
budget_tokens = max(
 min(max_tokens × effort_ratio, 24576),
 1024
)
```

Where:

| Effort     | effort\_ratio |
| ---------- | ------------- |
| `max`      | 0.9           |
| `xhigh`    | 0.85          |
| `high`     | 0.8           |
| `trending` | 0.8           |
| `medium`   | 0.5           |
| `low`      | 0.2           |
| `minimal`  | 0.1           |
| `none`     | 0             |

> **`none` is a special case:** an `effort_ratio` of `0` would otherwise still floor to the 1024-token minimum under the formula above. Instead, `effort: "none"` disables reasoning entirely (equivalent to omitting `reasoning` / setting `enabled: false`) — no reasoning budget is allocated and no reasoning tokens are generated or billed.

**Important Constraint**

> **`max_tokens` must be strictly greater than the reasoning budget**, otherwise the model will not have enough tokens to produce a final answer. This applies to all effort levels except `none`.

**Adaptive Thinking (Newer Anthropic Models)**

Newer Anthropic models support **adaptive thinking**, where the model dynamically decides how much of its reasoning budget to actually use for a given request, rather than always consuming the full `budget_tokens` allocated.

* FastRouter still computes and sends `budget_tokens` (via the formula above) to the provider as an upper bound.
* The model adaptively spends anywhere from a small fraction of that budget up to the full amount, depending on task complexity — a simple prompt at `effort: "max"` may consume far fewer reasoning tokens than the 90%-of-`max_tokens` ceiling suggests.
* Billing reflects **actual reasoning tokens generated**, not the requested budget — the budget is a ceiling, not a guarantee of spend.
* This means observed reasoning-token usage on adaptive models will typically be lower, and more variable, than the static budget formula alone would imply.

***

### **Excluding Reasoning Tokens**

You can instruct the model to reason internally **without returning reasoning tokens**.

```json
"reasoning": {
 "exclude": true
}
```

* The model still performs reasoning
* Reasoning tokens are **not included** in the response
* Works across **all models**
* Note: this is distinct from `effort: "none"` — `exclude` still allocates and bills a reasoning budget (subject to adaptive thinking behavior on supported models), it just withholds the reasoning text from the response. `effort: "none"` skips reasoning altogether.

***

### **Token Usage & Billing**

* Reasoning tokens are counted as **output tokens**
* They are billed the same way as regular output tokens
* Enabling reasoning increases token usage but often improves: Accuracy, Coherence, Tool-calling correctness
* On adaptive-thinking Anthropic models, billed reasoning tokens reflect actual usage, which may be well below the computed budget ceiling
