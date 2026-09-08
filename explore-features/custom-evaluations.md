---
description: >-
  FastRouter's Custom Evaluations lets you benchmark and compare AI models on
  your own data using LLM-based judges to automatically score accuracy, latency,
  and cost.
icon: star-exclamation
---

# Custom Evaluations

## **Introduction**

FastRouter's Custom Evaluations feature allows you to assess and compare the performance of AI models on your datasets. By importing chat completion logs or uploading a dataset, generating outputs from the models you want to compare, and defining evaluation criteria with LLM-based judges, you can quantitatively measure aspects like accuracy, relevance, latency, and cost. This is ideal for benchmarking models, optimizing prompts, and ensuring high-quality responses in production.

Evaluations are managed through the FastRouter dashboard, where you can create, run, and analyze evaluations asynchronously. Results include detailed metrics and scores, helping you make data-driven decisions.

Custom Evaluations supports four data sources, all through the same workflow:

| Source               | Use it when                                                                                  |
| -------------------- | -------------------------------------------------------------------------------------------- |
| **Chat Completions** | You want to replay real production traffic against other models                              |
| **Files**            | You have a curated JSONL or CSV test set — see [Dataset Evaluations](dataset-evaluations.md) |
| **Images**           | You want to score generated images — see [Image Evaluations](image-evaluations.md)           |
| **Videos**           | You want to score generated videos — see [Video Evaluations](video-evaluations.md)           |

This page walks through the chat completions flow end to end.

***

## **Key Benefits**

* Automated judging using LLM evaluators for scalable assessments.
* Compare several models against the same inputs in a single run, with the model from your logs kept as the baseline.
* Support for custom criteria tailored to your use case (e.g., factual accuracy, creativity, conciseness, schema compliance).
* Integration with your API keys for secure, cost-effective processing.
* Visual dashboards for easy comparison of runs and metrics.

***

## **Before You Start**

Evaluations built from chat completions run against traffic you have already sent through FastRouter — either via the API or interactively from **Model Playground**.

<figure><img src="../.gitbook/assets/0. Chat Response.png" alt=""><figcaption><p>A chat completion in Model Playground</p></figcaption></figure>

The example used throughout this page is an SEO URL-classification task: the model receives a URL and must return strict JSON with a normalized URL, locale, category, content type, keywords, and confidence. Structured tasks like this are the easiest to evaluate well, because a judge can check the output field by field instead of forming an opinion about prose.

***

## **Creating a New Evaluation**

Navigate to the **Evaluations** section in your FastRouter dashboard and click **Create Evaluation**.

<figure><img src="../.gitbook/assets/1. New Eval.png" alt=""><figcaption><p>Custom Evaluations: New Evaluation</p></figcaption></figure>

**1. Name Your Evaluation**

Provide a descriptive name (e.g., `Chat Eval` or `SEO-URL-Classification-Benchmark`).

**2. Import Test Data**

Click **Import Data** and select the **Chat Completions** tab.

<figure><img src="../.gitbook/assets/2. Import Test Data.png" alt=""><figcaption><p>Custom Evaluations: Import Test Data</p></figcaption></figure>

Configure the following:

* **Date Range** _(required)_: The period covering the requests you want to evaluate.
* **Model** _(required)_: The model whose logged traffic you want to import (e.g., `moonshotai/kimi-k3`).
* **Project** and **Key**: Optionally narrow to a specific project or API key. Selecting a project filters the available keys; leave as "All Projects" to see all keys.
* **Input contains** / **Output contains**: Filter to logs whose request or response contains specific text — useful for isolating one task type out of mixed traffic.
* **Sampling rate (%)**: Import a percentage of matching logs (1–100%). Start small to validate your setup before scaling.

The dialog shows a live **Total rows** count as you adjust filters. Click **Import** to load the logs as your evaluation dataset.

**3. Review the Data Preview**

After importing, the dataset appears as **Imported from Chat Completions input**, and the source model is added automatically under **Models to Compare** as the baseline. The **Data (preview)** panel shows each row's input alongside the output that model actually produced in production.

<figure><img src="../.gitbook/assets/3. Data Preview.png" alt=""><figcaption><p>Custom Evaluations: Data Preview</p></figcaption></figure>

Check this preview before continuing — it is the quickest way to confirm you imported the traffic you intended.

**4. Add Models to Compare**

Click **Add Models** to choose the models you want to evaluate against the same inputs.

<figure><img src="../.gitbook/assets/4. Add Models.png" alt=""><figcaption><p>Custom Evaluations: Add Models</p></figcaption></figure>

Filter the catalog by category, context length, input pricing, or creator, or search by name. Each selected model becomes its own run in the results, generating fresh responses for every row in the dataset. The model imported from your logs stays as the **Baseline**, so you are comparing candidates against what production actually returned rather than against a fresh re-run.

> ℹ️ Prices in the picker are shown per 1M tokens for text models and per unit for media models, so you can weigh cost before committing to a run.

**5. Add Evaluation Metrics**

Click **Add Metric** and choose a testing criteria type:

* **Model Scorer** — uses a model to produce a numeric score within a range you define. Available as **Auto grader** (ready-made rubric) or **Custom grader** (your own prompt).
* **Model Labeler** — uses a model to classify each row and determine which items pass. Available as **Criteria match** or **Custom labeler**.

For task-specific benchmarking, **Model Scorer → Custom grader** gives the most control.

<figure><img src="../.gitbook/assets/5. Custom Grader.png" alt=""><figcaption><p>Custom Evaluations: Edit Test Criteria</p></figcaption></figure>

Configure the judge:

* **Model** _(required)_: The judge model. Use good reasoning models that can generate structured outputs as a judge.
*   **System**: The judge's instructions and rubric. Framing the judge as an auditor rather than a second attempt at the task produces more reliable scores:

    ```
    You are a strict evaluator of SEO URL-classification outputs. You will judge ONE candidate
    response against a fixed task and a gold rubric. You are not the candidate; do not redo the
    task. Your job is to find failures.

    === GOLD RUBRIC FOR THIS URL ===
    Use these as ground truth. "Acceptable" lists every value that earns full credit; anything else
    is a fail for that criterion unless the candidate's reasoning gives a defensible URL-grounded
    justification.
    C1 normalized_url  Must NOT contain: ref=, utm_campaign=, #reviews  Acceptable: ...
    ```
*   **User** _(required)_: The prompt template sent to the judge, built from variables:

    ```
    === ORIGINAL TASK GIVEN TO THE CANDIDATE ==={{item.input}}
    === CANDIDATE OUTPUT === {{sample.output}}
    ```
* **Variables**: Use `{{curly braces}}` to insert values. `{{item.input}}` is the original request, `{{sample.output}}` is the response being judged.
* **Range** and **Pass Threshold**: Define the numeric scale and the score at which a row counts as a pass.

You can add multiple metrics to score different dimensions separately — for example one grader for schema compliance and another for keyword quality.

**6. Select an Evaluation Key**

Choose an API key from your account. This key is used for both response generation and judge calls during the evaluation.

**7. Run the Evaluation**

Click **Run**. FastRouter shows a credit utilization estimate — the number of requests, the estimated cost, and your current balance — before starting. Confirm to proceed, and the evaluation generates outputs for each model and applies the judges asynchronously.

> ⚠️ A run makes `rows × models` generation calls plus the same number of judge calls. Check the estimate before running a large dataset against several models. Baseline requests are not re-run.

***

## **Viewing Evaluation Results**

Open a completed evaluation from the Evaluations listing page. Use the **Report** / **Data** toggle in the top right to switch views.

**Report view** — One row per model run with its Request ID, p50 and p95 latency, cost, and grader score. The **Test Criteria** panel below shows the judge prompt, its range and pass threshold, the judge model, and the total judge spend.

<figure><img src="../.gitbook/assets/6. Report Details.png" alt=""><figcaption><p>Custom Evaluations: Report Details</p></figcaption></figure>

Example results for the SEO URL-classification benchmark:

| Run                          | Source                        | p50 Latency | p95 Latency | Cost    | Custom Grader |
| ---------------------------- | ----------------------------- | ----------- | ----------- | ------- | ------------- |
| `moonshotai/kimi-k3`         | Imported from logs (Baseline) | 26,563 ms   | 28,906 ms   | $0.0848 | 50%           |
| `google/gemini-3.8-flash`    | Generated                     | 15,677 ms   | 16,588 ms   | $0.0290 | 0%            |
| `z-ai/glm-5.3-flash`         | Generated                     | 42,427 ms   | 43,247 ms   | $0.0005 | 0%            |
| `deepseek/deepseek-v4-flash` | Generated                     | 9,847 ms    | 13,539 ms   | $0.0002 | 0%            |

Total judge score cost for the evaluation: $0.1802.

Read this table as a whole rather than by score alone. The cheaper models are dramatically faster and cheaper but none of them passed the strict rubric, while the baseline passed only half its rows — a signal that either the task needs a stronger model, the prompt needs tightening, or the rubric is stricter than the use case requires.

**Data view** — One row per test case with the baseline output and each generated output side by side, including score, latency, and cost. Switch between **Single View** and **Compare**, and use the selector to choose which runs are shown.

<figure><img src="../.gitbook/assets/7. Compare Mode.png" alt=""><figcaption><p>Custom Evaluations: Compare mode</p></figcaption></figure>

**Content Preview** — Click the expand arrow on any row to open the full input and outputs side by side. Use the **Input** / **Output** tabs to switch between the request that was sent and the responses that came back, and the pin control to keep a run in view while scrolling across others.

<figure><img src="../.gitbook/assets/8. Content Preview.png" alt=""><figcaption><p>Custom Evaluations: Content Preview</p></figcaption></figure>

**Judge Reasoning** — Click any score to open **Criteria Results** and see the judge's reasoning criterion by criterion, along with the grader configuration that produced the verdict.

<figure><img src="../.gitbook/assets/9. Reasoning Score Details.png" alt=""><figcaption><p>Custom Evaluations: Judge Reasoning</p></figcaption></figure>

For the example rubric, the judge reports a result per named criterion — `C1 normalized_url`, `C2 recommended_canonical`, `C3 locale`, `C4 primary_category`, `C5 secondary_category`, `C6 content_type` — with the value it found and the credit awarded. That granularity is what lets you tell a model that got one field wrong from one that misunderstood the task.

***

## **Tips & Best Practices**

* **Start small:** Begin with a small sample size (10–50 rows) to test your setup before scaling to larger datasets.
* **Keep the baseline honest:** The run imported from your logs reflects real production behaviour. Comparing candidates against it is more meaningful than comparing them only against each other.
* **Name your criteria:** Rubrics broken into labelled criteria (C1, C2, C3…) produce judge reasoning you can act on. A single overall score tells you something failed but not what.
* **Tell the judge not to redo the task:** Judges that attempt the task themselves tend to grade against their own answer rather than your rubric.
* **Diverse criteria:** Use multiple judges for comprehensive evaluations (e.g., one for schema validity, another for keyword quality).
* **Judge calibration:** Test your judge prompt on a handful of rows first. A 0% pass rate often means the rubric is stricter than intended rather than that every model failed.
* **Watch p95, not just p50:** A model can look fast on median latency and still have a tail that hurts in production.
* **Cost management:** Monitor the estimate in the setup phase. Use efficient models for judges, and remember that adding a model multiplies generation cost across every row.

