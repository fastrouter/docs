---
description: >-
  Create reusable bundles of reasoning, flex tier and prompt compression, and
  apply them to virtual model aliases, evaluation runs and API keys.
icon: filter
---

# Optimization Settings

### Introduction

An **optimization setting** is a named, reusable bundle of three controls that change how a request is run:

| Control                | What it does                                                 | Where it applies                          |
| ---------------------- | ------------------------------------------------------------ | ----------------------------------------- |
| **Reasoning**          | How much a model thinks before it answers                    | Models that support reasoning controls    |
| **Flex tier**          | Runs the request on the provider's cheaper, slower tier      | OpenAI and Google models with a flex tier |
| **Prompt compression** | Shrinks input tokens before the request reaches the provider | Every model                               |

You configure a setting once, then apply it wherever you need it:

* **Virtual model alias targets**: each model inside an alias can have its own setting
* **Evaluation candidates**: compare the same model with and without a setting
* **API keys**: a default for every request made with the key

Any part of a setting that a model doesn't support is skipped, so one setting can safely be applied to an alias that mixes providers.

<figure><img src="../.gitbook/assets/Optimization Setting.png" alt=""><figcaption><p><em>Optimize › Optimization Settings</em></p></figcaption></figure>

***

### Create a setting

1. In the sidebar, go to **Optimize › Optimization Settings**.
2. Click **Create Setting**.
3. Fill in the details and configure the three sections described below.
4. Click **Create setting**.

<figure><img src="../.gitbook/assets/Screenshot 2026-10-16 at 5.03.51 PM.png" alt=""><figcaption><p>Create Optimization Setting</p></figcaption></figure>

#### Details

| Field           | Required | Notes                                                                                        |
| --------------- | -------- | -------------------------------------------------------------------------------------------- |
| **Name**        | Yes      | Display name shown in dropdowns, e.g. `Cheap batch summaries`                                |
| **Identifier**  | Yes      | Lowercase letters, numbers and hyphens. Filled in from the name. **Can't be changed later.** |
| **Description** | No       | Tell teammates when to use this setting                                                      |

#### Reasoning

Choose how much models think before they answer.

| Mode              | Behavior                                                                 |
| ----------------- | ------------------------------------------------------------------------ |
| **Model default** | Nothing is sent. Each model uses its own reasoning default.              |
| **Effort**        | Sends a reasoning level (None, Minimal, Low, Medium, High). Recommended. |
| **Token budget**  | Sends an exact thinking-token budget                                     |

<figure><img src="../.gitbook/assets/Optimization Setting - Reasoning.png" alt=""><figcaption><p>Optimization Setting with Reasoning Effort</p></figcaption></figure>

**How Effort is translated per provider**

| Provider / model family | What FastRouter sends                                                              |
| ----------------------- | ---------------------------------------------------------------------------------- |
| OpenAI, Grok            | `reasoning_effort`                                                                 |
| Gemini 3                | `thinkingLevel` at the chosen level                                                |
| Claude 4.6 and later    | Adaptive thinking at the chosen effort. Billing follows the tokens actually used.  |
| Claude 4.5 and earlier  | Converted to a budget of 80% of `max_tokens`, kept between 1,024 and 24,576 tokens |
| Gemini 2.5              | Not applied, since it takes a token budget                                         |
| Newer Qwen models       | Turns thinking on. Cap the combined output with `max_completion_tokens`.           |

> **Note:** If a model doesn't accept the level you chose, FastRouter sends the nearest level it does accept. For example, None and Minimal run as Low on GPT-6. Check the provider's model page for the levels each model accepts.

> **Tip:** Effort is the portable choice. Use **Token budget** only when you need an exact number on Gemini 2.5 or older Claude models.

#### Flex tier

Turn on the toggle to send requests to the provider's flex tier.

* Around **50% cheaper** than the standard tier
* Responses can take noticeably longer
* Requests can be rejected when provider capacity is tight

Flex tier applies only to models and providers that offer one (currently **OpenAI and Google**). Everything else runs on the standard tier.

Use flex for work that can wait: batch jobs, background enrichment, offline evaluations. Avoid it for user-facing, latency-sensitive traffic.

<figure><img src="../.gitbook/assets/Optimization Setting - Flex Tier.png" alt=""><figcaption><p>Optimization Setting with Flex tier</p></figcaption></figure>

#### Prompt compression

Reduce input tokens before the request reaches the provider. Works on every model.

| Engine        | Behavior                                                                                |
| ------------- | --------------------------------------------------------------------------------------- |
| **Off**       | Messages are sent to the provider unchanged                                             |
| **Headroom**  | Lossless. Restructures system prompts, rules and tool schemas without changing meaning. |
| **Caveman**   | _TODO: one-line description_                                                            |
| **LLMLingua** | _TODO: one-line description_                                                            |
| **All**       | Runs the available engines together for the largest reduction                           |

Each engine has optional tuning fields. Leave a field empty to use the engine's default.

| Field                        | Example                   | Meaning                                                       |
| ---------------------------- | ------------------------- | ------------------------------------------------------------- |
| **Target ratio**             | `0.6`                     | Target size of the compressed prompt relative to the original |
| **Min tokens to compress**   | `64`                      | Prompts smaller than this are left alone                      |
| **Keep recent turns**        | `2`                       | Most recent conversation turns that are never compressed      |
| **Compress user messages**   | Engine default / On / Off | Whether user messages are compressed                          |
| **Compress system messages** | Engine default / On / Off | Whether system messages are compressed                        |
| **Protect analysis context** | Engine default / On / Off | Keeps context the model needs for analysis intact             |

<figure><img src="../.gitbook/assets/Optimization Setting - Prompt Compression.png" alt=""><figcaption><p>Optimization Setting with Prompt compression</p></figcaption></figure>

Compression applies to **Chat Completions** and **Messages API** requests. If compression can't run, the original messages are sent, so a request never fails because of compression.

***

### Apply a setting

A setting does nothing until you apply it. You can apply the same setting in any number of places.

#### To a virtual model alias target

Each model in a virtual model alias can have its own setting. This lets one alias, for example, run GPT-6.1 on flex while keeping GPT-6 on the standard tier.

1. Go to **Setup › Virtual Models** and create or edit an alias.
2. In the **Strategy** step, find the model in **You have selected N models**.
3. Pick a setting from the dropdown next to the model (it defaults to **No optimization setting**).
4. Save the alias.

<figure><img src="../.gitbook/assets/Create Virtual Model Alias.png" alt=""><figcaption><p>Optimization Setting applied to a Virtual Model Alias</p></figcaption></figure>

The setting applies whenever the alias routes a request to that model, whatever the routing strategy.

#### To an evaluation candidate

Use settings in evaluations to measure the quality and cost impact of reasoning, flex and compression before rolling them out.

1. Go to **Optimize › Evaluations** and click **New Evaluation**.
2. Under **Models to Compare**, pick a setting from the dropdown next to each candidate.
3. Add metrics and click **Run**.

<figure><img src="../.gitbook/assets/Optimization Setting in an Eval Run (1).png" alt=""><figcaption><p>Optimization Setting applied to an Evaluation Run</p></figcaption></figure>

#### As an API key default

A key's default setting applies to every request made with that key, unless something more specific applies (see Precedence).

1. Go to **Manage › Keys** and create or edit a key.
2. Under **Default Optimization Setting**, pick a setting.
3. Click **Save**.

<figure><img src="../.gitbook/assets/Optimization Setting in a Key.png" alt=""><figcaption><p>Optimization Setting applied to a Key</p></figcaption></figure>

***

### Precedence

When more than one setting could apply to a request, FastRouter picks the **single most specific setting** and uses it **as a whole**. Settings are not merged with each other.

From highest to lowest priority:

| Priority | Source                         | Applies when                                                                        |
| -------- | ------------------------------ | ----------------------------------------------------------------------------------- |
| 1        | **Virtual model alias target** | The request goes through an alias, and the target the alias routed to has a setting |
| 2        | **API key default**            | The key that made the request has a default setting                                 |
| 3        | **Evaluation candidate**       | The request is part of an evaluation run, and the candidate has a setting           |

If none of these apply, the request runs with no optimization setting.

#### Examples

**Alias target beats the key default**

* Key `prod-backend` has default setting `balanced` (Effort: Medium, compression: Headroom).
* Alias `summarizer` routes to GPT-6.1 Sol with setting `flex-batch` (flex on, Effort: Low).

A request with `prod-backend` to `summarizer` uses **`flex-batch` as a whole**: flex on, Effort Low, and **no compression**, because `balanced` is not merged in.

**Key default with a request override**

Key `prod-backend` has default `balanced` (Effort: Medium, compression: Headroom). The request is sent directly to a model and includes:

```json
{
  "model": "openai/gpt-6.1-sol",
  "messages": [{ "role": "user", "content": "Summarize this ticket..." }],
  "service_tier": "flex"
}
```

The request runs with Effort Medium and Headroom compression from `balanced`, and on the flex tier from the request.

### FAQ

**Can I apply a setting to a whole alias instead of each target?** Not today. Settings are applied per target so mixed-provider aliases can be tuned model by model.

**Does flex tier work with fallbacks?** If a flex request is rejected for capacity, the alias's routing strategy handles it like any other failed request.

**Will compression change my model's answers?** Headroom is lossless. Other engines remove tokens and can affect output; run an evaluation with and without the setting before rolling it out.
