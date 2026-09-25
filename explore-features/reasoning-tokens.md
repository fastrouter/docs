---
description: >-
  FastRouter can return Reasoning Tokens (also known as thinking tokens) for
  supported models.
icon: lightbulb-message
---

# Reasoning Tokens

## **Overview**

FastRouter can return Reasoning Tokens (also known as _thinking tokens_) for supported models. These tokens represent the model's internal reasoning process and can significantly improve output quality for complex tasks such as planning, math, tool use, and multi-step analysis.

* Reasoning is **enabled by default** for supported models.
* The model decides how much to reason unless you explicitly control it.
* Reasoning content, where the provider returns it, appears in the **`reasoning_details`** array of each message (Chat Completions) or as **`thinking` content blocks** (Messages).
* You can **limit**, **control**, or **exclude** reasoning using the `reasoning` parameter.

***

## **Supported Models**

Reasoning controls differ per model, not just per provider. FastRouter sends whichever control the selected model accepts.

<table data-search="false"><thead><tr><th>Provider</th><th>Models</th><th>Control FastRouter sends</th><th>Accepted values</th></tr></thead><tbody><tr><td><strong>OpenAI</strong></td><td>GPT-6 family, GPT-5.x family, o-series</td><td><code>reasoning_effort</code></td><td>Model-dependent subset of <code>none</code>, <code>minimal</code>, <code>low</code>, <code>medium</code>, <code>high</code>, <code>xhigh</code>, <code>max</code></td></tr><tr><td><strong>xAI</strong></td><td>Grok 4.5, Grok 4.6 (and later)</td><td><code>reasoning_effort</code></td><td><code>low</code>, <code>medium</code>, <code>high</code> (default), plus <code>xhigh</code> on Grok 4.6 and later</td></tr><tr><td><strong>Google</strong></td><td>Gemini 3 series</td><td><code>thinkingLevel</code></td><td><code>minimal</code>, <code>low</code>, <code>medium</code>, <code>high</code> — per model</td></tr><tr><td><strong>Google</strong></td><td>Gemini 2.5 series</td><td><code>thinkingBudget</code></td><td>Token budget, per-model range</td></tr><tr><td><strong>Anthropic</strong></td><td>Claude 4.6 and later, including the Claude 5 family</td><td>Adaptive thinking with <code>effort</code></td><td><code>low</code>, <code>medium</code>, <code>high</code>, <code>xhigh</code>, <code>max</code> — per model</td></tr><tr><td><strong>Anthropic</strong></td><td>Claude 4.5 and earlier</td><td>Extended thinking with <code>budget_tokens</code></td><td>1,024 tokens minimum</td></tr><tr><td><strong>Qwen</strong></td><td>Newer Qwen models, such as Qwen3.8 Flash</td><td><code>enable_thinking</code></td><td>On or off. Cap the combined output with <code>max_completion_tokens</code></td></tr></tbody></table>

Models that are not listed ignore the `reasoning` parameter. FastRouter drops it rather than failing the request.

> **Check the model page before pinning a value.** Accepted levels, defaults, budget ranges and whether reasoning can be turned off are set per model by the provider, and they change with each release. The table above gives the shape of the control; the provider's own model description is the source of truth for the exact values a given model accepts.

***

## **Controlling Reasoning Tokens**

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

`effort` is the portable option. It works on every model in the table above, including Anthropic models, where FastRouter maps it to the model's native control. Use `max_tokens` only when you need an exact token budget on Gemini 2.5 or on Claude 4.5 and earlier.

***

## **Reasoning Effort Levels**

**Effort Options**

<table data-search="false"><thead><tr><th>Effort</th><th>Intent</th></tr></thead><tbody><tr><td><code>max</code></td><td>Maximum reasoning depth</td></tr><tr><td><code>xhigh</code></td><td>Near-maximum depth, for long-running or hard tasks</td></tr><tr><td><code>high</code></td><td>Deep reasoning for complex work</td></tr><tr><td><code>trending</code></td><td>Currently the same as <code>high</code></td></tr><tr><td><code>medium</code></td><td>Balanced default for most workloads</td></tr><tr><td><code>low</code></td><td>Light reasoning, faster and cheaper</td></tr><tr><td><code>minimal</code></td><td>Smallest reasoning pass a model will accept</td></tr><tr><td><code>none</code></td><td>Reasoning disabled</td></tr></tbody></table>

> **Note:** `trending` currently maps to the same allocation as `high`. It is provided as a separate, forward-compatible label and may be tuned independently in the future — don't assume it will always equal `high`.

**When a model doesn't accept a level**

Providers publish the accepted levels on each model's description page, so check there first when a specific level matters to you.

Providers expose different subsets of these levels, so FastRouter maps an unsupported level to the nearest level the selected model accepts, and reports the substitution in the Activity Log. Examples:

| Request             | Model              | What is sent                                                        |
| ------------------- | ------------------ | ------------------------------------------------------------------- |
| `effort: "none"`    | GPT-6 Astra        | `low` — the model rejects `none`                                    |
| `effort: "minimal"` | GPT-6 Astra        | `low` — the model rejects `minimal`                                 |
| `effort: "xhigh"`   | Grok 4.5           | `high`                                                              |
| `effort: "max"`     | Gemini 3 Flash     | `thinkingLevel: high`                                               |
| `effort: "none"`    | Grok, Gemini 3 Pro | Lowest level the model supports, since reasoning cannot be disabled |

**Where effort maps to a token budget**

On Claude 4.5 and earlier, `effort` is converted into a token budget using this formula:

```
budget_tokens = max(
 min(max_tokens × effort_ratio, 24576),
 1024
)
```

Where:

<table data-search="false"><thead><tr><th>Effort</th><th>effort_ratio</th></tr></thead><tbody><tr><td><code>max</code></td><td>0.9</td></tr><tr><td><code>xhigh</code></td><td>0.85</td></tr><tr><td><code>high</code></td><td>0.8</td></tr><tr><td><code>trending</code></td><td>0.8</td></tr><tr><td><code>medium</code></td><td>0.5</td></tr><tr><td><code>low</code></td><td>0.2</td></tr><tr><td><code>minimal</code></td><td>0.1</td></tr><tr><td><code>none</code></td><td>0</td></tr></tbody></table>

> **`none` is a special case:** an `effort_ratio` of `0` would otherwise still floor to the 1024-token minimum. Instead, `effort: "none"` disables reasoning entirely (equivalent to omitting `reasoning` / setting `enabled: false`) — no reasoning budget is allocated and no reasoning tokens are generated or billed.

> **Important:** where a budget applies, your request's `max_tokens` must be strictly greater than the reasoning budget, otherwise the model has no room left for a final answer.

Claude 4.6 and later do **not** use this formula. FastRouter passes the effort level through to Anthropic's adaptive thinking instead. See [Anthropic-Specific Reasoning Behavior](reasoning-tokens.md#anthropic-specific-reasoning-behavior).

***

## **Reasoning Max Tokens**

**Supported By**

* **Gemini 2.5 thinking models** (sent as `thinkingBudget`)
* **Claude 4.5 and earlier** (sent as `budget_tokens`, minimum 1,024)

Example:

```json
"reasoning": {
 "max_tokens": 2000
}
```

On models that take a level rather than a budget (OpenAI, Grok, Gemini 3, Claude 4.6 and later), `max_tokens` cannot be applied. FastRouter converts it to the closest effort level.

***

## **OpenAI Reasoning Behavior**

FastRouter sends `reasoning.effort` as OpenAI's `reasoning_effort`. Accepted values vary by model, so check the model page before pinning a level:

| Model        | Accepted effort values                   | Default   |
| ------------ | ---------------------------------------- | --------- |
| GPT-6 family | `low`, `medium`, `high`, `xhigh`, `max`  | Per model |
| GPT-5.2      | `none`, `low`, `medium`, `high`, `xhigh` | `none`    |
| GPT-5.1      | `none`, `low`, `medium`, `high`          | `none`    |
| GPT-5        | `minimal`, `low`, `medium`, `high`       | `medium`  |
| o-series     | `low`, `medium`, `high`                  | `medium`  |

The GPT-6 family does not accept `none` or `minimal`. FastRouter sends `low` instead of failing the request.

***

## **xAI Reasoning Behavior**

* Grok 4.6 and later accept `low`, `medium`, `high` and `xhigh`.
* Grok 4.5 accepts `low`, `medium` and `high`. `xhigh` runs as `high`.
* Reasoning cannot be disabled on these models, so `effort: "none"` is sent as the lowest supported level.
* On multi-agent Grok models, `effort` controls **how many agents** collaborate rather than reasoning depth. FastRouter does not send effort to these models, because the cost impact is different in kind. Set agent count in your request to the provider's native parameter instead.

***

#### **Google Gemini Reasoning Behavior**

Google Gemini models support reasoning, but the API used depends on the model generation.

**Gemini 2.5 Models — `thinkingBudget` API**

Gemini 2.5 thinking models use Google's `thinkingBudget` API. With FastRouter, you control this using `reasoning.max_tokens`, which is passed through as the thinking budget. Budget ranges are per model: Gemini 2.5 Pro cannot disable thinking, while Flash and Flash-Lite can.

```json
"reasoning": {
 "max_tokens": 2000
}
```

**Gemini 3 Models — `thinkingLevel` API**

Gemini 3 models use Google's newer `thinkingLevel` API instead of `thinkingBudget`. FastRouter maps `reasoning.effort` as follows:

| FastRouter reasoning.effort | Google thinkingLevel                            |
| --------------------------- | ----------------------------------------------- |
| `max`                       | `high`                                          |
| `xhigh`                     | `high`                                          |
| `high`                      | `high`                                          |
| `trending`                  | `high`                                          |
| `medium`                    | `medium`                                        |
| `low`                       | `low`                                           |
| `minimal`                   | `minimal`                                       |
| `none`                      | _(thinking disabled where the model allows it)_ |

> Google's `thinkingLevel` API exposes four levels (`minimal`, `low`, `medium`, `high`), and individual models support only a subset. Pro models, for example, don't accept `minimal` and can't turn thinking off. FastRouter sends the nearest supported level.

**Token Consumption is Determined by Google**

When using `thinkingLevel`, the actual number of reasoning tokens consumed is determined internally by Google. There are no publicly documented token limit breakpoints for each level. For example, setting `effort: "low"` might result in several hundred reasoning tokens depending on the complexity of the task. This is expected behavior and reflects how Google implements thinking levels internally.

***

## **Anthropic-Specific Reasoning Behavior**

**Claude 4.6 and later, including the Claude 5 family**

These models use **adaptive thinking**: Claude decides whether and how much to think on each request, guided by an effort level. FastRouter maps `reasoning.effort` to Anthropic's `effort` and enables adaptive thinking.

* Fixed token budgets are deprecated on Claude 4.6 and **rejected** on Claude Opus 4.7 and later and the Claude 5 family. On those models, `reasoning.max_tokens` is converted to the nearest effort level.
* Billing reflects **actual reasoning tokens generated**. A simple prompt at `effort: "max"` may use far fewer tokens than the level suggests.
* Some models cannot turn reasoning off. Claude Fable 5 always thinks, and Claude Opus 5 accepts a disabled state only at `high` effort or lower.
* Thinking text is **omitted from responses by default** on Claude Opus 4.7 and later. FastRouter requests summarized thinking when `exclude` is `false`, so reasoning content is returned where the model supports it.

**Claude 4.5 and earlier**

These models use extended thinking with a fixed budget:

* `reasoning.max_tokens` is used directly, with a minimum of **1,024** tokens.
* `reasoning.effort` is converted into a budget with the formula above, clamped between **1,024** and **24,576** tokens.
* Your request's `max_tokens` must be strictly greater than the reasoning budget.

***

## **Qwen Reasoning Behavior**

Newer Qwen models, such as Qwen3.8 Flash, use a simple on/off thinking switch rather than a level or a budget. FastRouter sends it as `enable_thinking`.

```json
{
 "model": "qwen3.8-flash",
 "messages": [],
 "max_completion_tokens": 200,
 "enable_thinking": true
}
```

**Use `max_completion_tokens`, not `max_tokens`**

On these models the cap applies to the **total output: reasoning tokens and the final answer combined**. Set it high enough that the answer still fits after the model has finished thinking. A cap that is too low can return reasoning with a truncated answer, or no answer at all, and you are still billed for the reasoning tokens generated.

Because thinking is a toggle here, an `effort` level only decides whether thinking is on. There is no separate reasoning budget to set. Check the model's description on the provider's site for its thinking defaults and any per-model limits.

***

## **How Reasoning Appears in Responses**

What comes back depends on the endpoint and on how much the provider exposes.

**Chat Completions**

Reasoning content is returned in `message.reasoning_details`, an array of typed blocks:

```json
"reasoning_details": [
 {
 "type": "reasoning.text",
 "text": "The model is considering multiple constraints before responding...",
 "format": "anthropic-claude-v1",
 "index": 0
 },
 {
 "type": "reasoning.encrypted",
 "data": "gAAAAAB...",
 "format": "openai-responses-v1",
 "index": 1
 }
]
```

| Block type            | Meaning                                                                   |
| --------------------- | ------------------------------------------------------------------------- |
| `reasoning.text`      | Readable reasoning, returned by providers that expose it                  |
| `reasoning.summary`   | A provider-generated summary of the reasoning                             |
| `reasoning.encrypted` | Opaque reasoning state. Pass it back on the next turn to preserve context |

Reasoning token counts appear in `usage.completion_tokens_details.reasoning_tokens`.

**Messages**

Reasoning is returned as `thinking` content blocks alongside `text` blocks, each with a `signature` you should pass back on later turns. Counts appear in `usage.output_tokens_details.thinking_tokens`.

```json
"content": [
 { "type": "thinking", "thinking": "...", "signature": "CAQS..." },
 { "type": "text", "text": "..." }
]
```

***

## **Excluding Reasoning Tokens**

You can instruct the model to reason internally **without returning reasoning content**.

```json
"reasoning": {
 "exclude": true
}
```

* The model still performs reasoning
* Reasoning content is **not included** in the response
* Works across **all models**
* Note: this is distinct from `effort: "none"` — `exclude` still allocates and bills reasoning, it just withholds the content. `effort: "none"` skips reasoning altogether, on models that allow it.

***

## **Token Usage & Billing**

* Reasoning tokens are counted as **output tokens**
* They are billed the same way as regular output tokens
* Enabling reasoning increases token usage but often improves: Accuracy, Coherence, Tool-calling correctness
* On adaptive-thinking models, billed reasoning tokens reflect actual usage, which may be well below the level you requested
* Reasoning tokens still count against your output limit, so leave room for the final answer
