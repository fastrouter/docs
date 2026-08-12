---
description: >-
  Weekly, evidence-backed recommendations that reduce your LLM spend without
  giving up output quality.
icon: lightbulb-on
---

# Insights

***

## Introduction to Insights

**Insights** analyses your organization's traffic week and surfaces concrete, ranked recommendations to cut spend while maintaining output quality.

Each recommendation is scoped to a real project, key, and model in your account, comes with a projected weekly saving, and shows the full calculation behind that number — including the raw request evidence it was derived from. Nothing is applied automatically: Insights tells you what to change and what it's worth, and you decide whether to act.

Find it in the dashboard under **Optimize → Insights**.

### The Insights page

#### Summary bar

Across the top of the page:

* **Projected savings** — the combined weekly value if you applied every recommendation in the current run.
* **Per-category totals** — projected savings and recommendation count for Caching, Flex Tier, and Model Switch.

Use the category totals to decide where to spend your review time: a single large Flex Tier recommendation is often worth more than a dozen small ones elsewhere.

#### Filtering and sorting

Filter the list by category using the **All / Caching / Flex Tier / Model Switch** tabs, and reorder it with the **Sort by** control. The default sort is savings, highest first.

#### Recommendation cards

Every card in the list shows:

| Element              | Description                                                                                                  |
| -------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Title**            | The action, in plain language — e.g. _Move `google/gemini-3-flash-preview` traffic to the Flex service tier_ |
| **Category badge**   | What kind of optimization this is                                                                            |
| **Scope chips**      | The project, API key, and model the recommendation applies to                                                |
| **Supporting line**  | The headline evidence — percentage off, quality delta, or request volume                                     |
| **Projected impact** | Weekly savings, or estimated cost savings for eval-backed recommendations                                    |
| **Preview**          | Opens the full detail panel                                                                                  |

***

### Recommendation detail

Selecting **Preview** opens a side panel with the complete reasoning behind a recommendation:

1. **Projected impact** — the headline weekly figure, plus generation date and measurement window.
2. **Scope** — project, key, and model.
3. **Why this recommendation?** — a short explanation of the pattern that was detected and what it currently costs you.
4. **The calculation** — the savings table or projection panel, showing current cost, modelled cost, and the difference.
5. **How was this calculated?** — the methodology in full, including formulas where relevant.
6. **Warnings** — trade-offs you should weigh before applying, such as Flex tier latency.
7. **Evidence** — the underlying request and token volumes, broken down by provider.
8. **How to apply** — the specific change to make.

<figure><img src=".gitbook/assets/Screenshot 2026-08-12 at 6.10.21 PM.png" alt=""><figcaption><p>The Insights page ranks every recommendation by projected weekly savings, with the scope — project, key, and model — shown on each card.</p></figcaption></figure>

***

### How it works

Insights runs on a weekly cadence over a rolling 7-day measurement window.

| Measure                | Description                                                           |
| ---------------------- | --------------------------------------------------------------------- |
| **Cadence**            | Recommendations refresh weekly                                        |
| **Measurement window** | Trailing 7 days (e.g. `Aug 5 – 12`)                                   |
| **Scope**              | Analysed per organization, broken down by project, API key, and model |
| **Ranking**            | Sorted by projected weekly savings, highest first                     |
| **Action**             | Manual — you review and apply the change yourself                     |

The page header always shows the **Last generated** date and the **Measurement window** the current set was computed from, so you know exactly which traffic produced the numbers you're looking at. Use the refresh control in the top-right to pull the latest run.

Because each run reflects the previous week's traffic, recommendations naturally appear and disappear as your workloads change. A recommendation that vanishes usually means the underlying pattern stopped — the traffic moved, the key went quiet, or you already applied the change.

***

### Recommendation types

Insights currently produces different categories of recommendation. Each has its own detection logic, its own savings model, and its own trade-offs. These include:

#### Model Switch

**What it detects:** A use case where a different model would be cheaper — and, in most cases, score better — on your actual traffic.

**How the saving is modelled:** Insights samples real requests from the use case and replays them against alternative models. Both output sets are scored by an LLM judge to produce a quality delta. The weekly impact reprices the sampled requests at the recommended model's rates and projects that across the scope's traffic.

```
PER SAMPLED REQUEST  →  WEEKLY IMPACT
   −$1.2074 (79%)    →    −$1,064.50
```

**What you'll see:** An **Evaluation summary** with the use case, current model, best alternative, quality change (as a before → after pair), estimated cost savings, and an **Eval ID** linking to the full evaluation run so you can inspect the replayed outputs yourself.

<figure><img src=".gitbook/assets/Screenshot 2026-08-12 at 6.10.57 PM.png" alt=""><figcaption><p>Model switch recommendations link out to the evaluation run that produced the quality delta, so you can read the replayed outputs yourself.</p></figcaption></figure>

**Trade-offs:** The quality delta comes from an automated LLM judge on a sample, not from your own rubric or your users. Treat a positive delta as a strong signal to run your own evaluation, not as a finished verdict — open the linked eval and read the outputs before switching a production key.

→ See [Evaluations](https://docs.fastrouter.ai/custom-evaluations) to run your own comparison.

> **Model Switch is opt-in and incurs cost.** Generating these recommendations means replaying sampled requests against alternative models and scoring both output sets with an LLM judge — real inference that is billed to your account. Enable it under **Settings** on the Insights page; Caching and Flex Tier recommendations are generated from your existing usage data and cost nothing.

#### Caching

**What it detects:** A system prompt prefix that repeats across many requests on the same key, where prompt caching is not currently in use.

**How the saving is modelled:** Requests on the key are bucketed into hourly windows across the 7-day measurement period and grouped by system-prompt hash. For every hour containing a repeated prefix, the first request is priced as a cache write and the remaining requests as cache reads, using the provider's published cache multipliers. That modelled cost is compared against what you actually paid, and the saving shown is the sum across every hourly bucket in the window.

```
// per hour, per repeated prefix
actual  = requests × prefix_tokens × input_rate
cached  = prefix_tokens × write_rate
        + (requests − 1) × prefix_tokens × read_rate
saving  = actual − cached
```

**What you'll see:** Number of repeated prefixes, requests on those prefixes, current cost, modelled cost with caching, potential saving, and the resulting cost reduction as a percentage.

<figure><img src=".gitbook/assets/Screenshot 2026-08-12 at 6.13.41 PM.png" alt=""><figcaption><p>A caching recommendation breaks out the modelled cost line by line, so you can see exactly which prefix volume the saving comes from.</p></figcaption></figure>

**Trade-offs:** Caching is generally safe — the model, the prompt, and the output are unchanged. Savings depend heavily on how tightly your repeated requests cluster in time, since cache entries expire; a prefix repeated 20,000 times but spread thinly across a week will save less than the same volume concentrated into bursts.

→ See [Prompt Caching](https://claude.ai/docs/prompt-caching) for how to enable it.

#### Flex Tier

**What it detects:** High-volume traffic running on a provider's standard service tier where the same model is available on a cheaper Flex tier.

**How the saving is modelled:** Every eligible request from the measurement window is repriced at the provider's published Flex rates, holding token counts constant. The projection panel shows standard cost, Flex cost, and the difference as your weekly impact.

```
STANDARD COST − FLEX COST = WEEKLY IMPACT
   $3,236.62  −  $1,641.62  =  −$1,595.00
```

**What you'll see:** A `same model, same output` marker, the standard-vs-Flex cost breakdown, a plain-language summary of the reduction, and an **Evidence** table listing the upstream provider, request count, and input/output token volumes the projection was built from.

<figure><img src=".gitbook/assets/Screenshot 2026-08-12 at 6.11.20 PM.png" alt=""><figcaption><p>Flex recommendations carry a <code>same model, same output</code> marker alongside an explicit warning about slower processing and queuing.</p></figcaption></figure>

**Trade-offs:** Flex runs the _same model_ and produces the _same quality of output_, but with slower processing and occasional queuing. Skip this one for any key that serves user-facing, latency-sensitive traffic — it's best suited to batch jobs, background enrichment, evals, and offline pipelines.

**How to apply:** Append `:flex` to the model slug on the key, or send it per request.

***

### Interpreting the numbers

A few things worth knowing before you take a projection to your finance team:

* **Projections are models, not guarantees.** Every figure is computed by repricing last week's traffic under different assumptions. If next week's volume or prompt mix differs, your realised saving will differ too.
* **Savings are not always additive.** Two recommendations can target the same traffic — for example a caching recommendation and a model switch on the same key. The **Projected savings** total assumes all recommendations are applied, so treat it as an upper bound rather than a forecast.
* **Small numbers are still real signals.** A recommendation worth cents per week on a low-volume key may be worth a great deal once that workload scales. Check the cost _reduction percentage_ alongside the absolute figure.
* **Quality deltas come from an automated judge.** For Model Switch recommendations, validate against your own evaluation criteria before changing production traffic.

***

### Settings

The **Settings** control on the Insights page lets you configure how recommendations are generated for your organization. Configuration is organization-wide and applies to the next weekly run.

This is where you enable **Model Switch** recommendations. They are off by default because generating them consumes billable inference — sampled requests are replayed against candidate models and judged. Caching and Flex Tier recommendations run on your existing usage data and are always on.

<figure><img src=".gitbook/assets/Screenshot 2026-08-12 at 6.28.17 PM.png" alt=""><figcaption><p>Enable Model Switch recommendations from Settings</p></figcaption></figure>

***

### FAQ

**Does FastRouter apply these changes automatically?** No. Insights is advisory. Every change — enabling caching, appending `:flex`, switching a model — is one you make yourself.

**Does generating Insights cost anything?** Caching and Flex Tier recommendations are computed from usage data you already have and are free. Model Switch recommendations replay sampled requests against alternative models and score them with an LLM judge, which is billed as normal inference — that's why they must be enabled explicitly in Settings.

**Why did a recommendation disappear?** Each run only reflects the last 7 days of traffic. If the pattern stopped, the key went idle, or you already applied the change, the recommendation drops out of the next run.

**Can I get recommendations more often than weekly?** Recommendations refresh on a weekly cadence. Use the refresh control to pull the most recent completed run.
