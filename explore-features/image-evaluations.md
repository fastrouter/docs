---
description: >-
  FastRouter's Image Evaluations feature lets you assess the quality of
  AI-generated images at scale.
icon: image-circle-check
---

# Image Evaluations

## **Introduction**

By importing image generation logs, defining LLM-based judging criteria, and running evaluations against those outputs, you can systematically measure image quality across dimensions like photorealism, material and lighting accuracy, physical plausibility, fine detail, and adherence to the original prompt.

Image Evals work within the same Custom Evaluations infrastructure as text and video evals — the same judge configuration, the same scoring rubrics, and the same results dashboard — extended to support image output.

***

## **Key Benefits**

* Evaluate AI-generated image outputs automatically using a multimodal LLM judge.
* Import image generation logs directly from your FastRouter activity — no manual uploads required.
* Use the same Auto Grader and Custom grader setup as text evaluations.
* Drill down into per-image judge reasoning to understand exactly what scored well or poorly.

***

## **Before You Start**

Image evals run against images you have already generated through FastRouter. You can generate them via the API, or interactively from **Model Playground** by switching to the **Image** tab and selecting an image model (e.g. `x-ai/grok-imagine-image-quality`).

<figure><img src="../.gitbook/assets/0. Image Playground.png" alt=""><figcaption><p>Generating images in Model Playground</p></figcaption></figure>

Every generation is recorded in **Activity Log**. Open a log entry to confirm the request landed and to preview the generated image before you build an evaluation around it.

<figure><img src="../.gitbook/assets/1. Activity Log.png" alt=""><figcaption><p>Image generation log in Activity Log</p></figcaption></figure>

***

## **Creating an Image Evaluation**

Navigate to the **Evaluations** section in your FastRouter dashboard and click **Create Evaluation**.

**Step 1 — Name Your Evaluation**

Provide a descriptive name (e.g. `Image Evaluation` or `Jewellery-Ad-Realism-v2`).

**Step 2 — Import Image Logs**

Click **Import Data**. In the Import Test Data dialog, select the **Images** tab.

<figure><img src="../.gitbook/assets/2. Create Image Evaluation.png" alt=""><figcaption><p>Import Test Data — Images tab</p></figcaption></figure>

Configure the following fields:

* **Date Range** _(required)_: Select the date range covering the image generations you want to evaluate.
* **Model** _(required)_: Choose the image generation model whose outputs you want to import (e.g. `x-ai/grok-imagine-image-quality`).
* **Project**: Optionally filter by project. Select a project to narrow the available API keys, or leave as "All Projects" to see all keys.
* **Key**: Optionally filter by a specific API key used during generation.
* **Input contains**: Search for specific text in the generation prompt to narrow down which logs are imported.
* **Sampling rate (%)**: Set a percentage of matching logs to import (1–100%). Useful for large log sets — start with a smaller sample to validate your setup before scaling.

The dialog shows a live **Total rows** count as you adjust the filters, so you can confirm the dataset size before importing. Click **Import** to load the image generation logs as your evaluation dataset.

Once imported, the dataset appears as **Imported from Image logs**, and the source model is added automatically under **Models to Compare** as the baseline run. The **Data (preview)** panel on the right shows each row's prompt alongside its image output.

<figure><img src="../.gitbook/assets/3. Import Logs.png" alt=""><figcaption><p>New Evaluation with imported image logs</p></figcaption></figure>

**Step 3 — Add Evaluation Metrics**

Click **Add Metric** and configure your judge. Two presets are available:

* **Auto grader**: a general-purpose quality judge with a ready-made rubric — good for a quick first pass.
* **Custom grader**: your own prompt, model, and scoring scale — recommended when you have domain-specific criteria (product realism, brand compliance, anatomy, typography, and so on).

<figure><img src="../.gitbook/assets/4. Judge Prompt.png" alt=""><figcaption><p>Configuring a Custom grader</p></figcaption></figure>

Configure the following:

* **Model** _(required)_: Select a multimodal model that accepts image input (e.g. `google/gemini-3.1-pro-preview`). The judge cannot score images if the selected model is text-only.
*   **System**: Describe the evaluator's role, the dimensions to check, and how to score them. Example:

    ```
    Act as a jewelry expert. Check this AI earring ad for realism. Score it from 1 to 7
    on these points and list any errors. Even if there is a single error, drop the
    overall score to 2:
    Diamond Sparkle: Does the diamond reflect light naturally, or does it look like flat glass/plastic?
    Metal Shine: Do the gold or platinum surfaces look like real metal reflecting studio light?
    Gravity and Fit: Does the earring hang naturally from the earlobe, or does it look floating and fake?
    Sharp Details: Are the tiny metal prongs holding the diamond sharp and distinct, or blurry and melted?
    Camera Focus: Is there a natural macro blur on the background, or is the whole image artificially sharp?
    ```
* **Variables**: Use `{{curly braces}}` to interpolate dataset values into your prompt. `{{item.input}}` passes the original generation prompt to the judge, which is what you need for prompt-adherence checks and `{{sample.output}}` refers to the output imported from the logs.
* **User** _(required)_: The user-turn template sent to the judge, typically the user input placeholder.
* **Scoring**: Define the numeric **Range** (e.g. `0–7`) and a **Pass Threshold** (e.g. `5`). Rows scoring at or above the threshold are marked Pass; everything below is Fail.

**Step 4 — Select Evaluation API Key**

Choose an API key from your account. This key is used for all LLM judge calls during the evaluation.

**Step 5 — Run**

Click **Run**. FastRouter first shows a **Credit Utilization Estimate** with the number of selected requests, the estimated cost, and your current balance.

Click **Proceed with Evaluation** to start the run. The judge is applied asynchronously to each image in the dataset and results are returned as they complete.

> ℹ️ The estimate is based on the selected judge model and an average prompt size — actual costs vary with image resolution and reasoning length. If credits run out mid-run, the evaluation may not complete.

***

## **Viewing Results**

Access results from the **Evaluations** listing page by clicking your evaluation. Use the **Report** / **Data** toggle in the top right to switch views.

**Report view** — Aggregated metrics across all rows: the run and its Request ID, latency percentiles (p50, p95), generation cost, and the overall score for each grader. The **Test Criteria** panel below shows the judge prompt, its range and pass threshold, the judge model, and the total judge spend for the run.

<figure><img src="../.gitbook/assets/6. Report.png" alt=""><figcaption><p>Report Details</p></figcaption></figure>

**Data view** — One row per image with its generation input on the left and the graded output on the right, including the score, latency, cost, and an image thumbnail. Use **Single View** to inspect one run, or **Compare** to place multiple model runs side by side. Search, sort, and fullscreen controls help with larger datasets.

<figure><img src="../.gitbook/assets/7. Data.png" alt=""><figcaption><p>Runs Overview — Data view</p></figcaption></figure>

**Content Preview** — Click the expand arrow on any row to open the full-size image next to its input parameters, with the grader verdict, latency, and cost alongside.

<figure><img src="../.gitbook/assets/8. Preview.png" alt=""><figcaption><p>Content Preview</p></figcaption></figure>

**Judge Reasoning** — Click the score to open **Criteria Results** and see the judge's per-dimension breakdown, along with the grader configuration that produced it.

<figure><img src="../.gitbook/assets/9. Scoring.png" alt=""><figcaption><p>Judge Feedback</p></figcaption></figure>

**Example output for a jewellery ad eval (`x-ai/grok-imagine-image-quality`):**

| Metric                 | Value                           |
| ---------------------- | ------------------------------- |
| Custom Grader          | Fail: 2 (Range 0–7, Pass ≥ 5.1) |
| Latency                | 4,767 ms                        |
| Generation Cost        | μ$50,000.0000 ($0.05)           |
| Total Judge Score Cost | $0.0243                         |
| Pass Rate              | 0%                              |

**Judge reasoning summary:**

1. **Diamond sparkle** — Flares are digital and exaggerated. Score: 3.
2. **Metal shine** — Surfaces look flat and illustrative. Score: 4.
3. **Gravity and fit** — Earrings float artificially on the ear. Score: 3.
4. **Sharp details** — Metal prongs are blurry and merged. Score: 3.
5. **Camera focus** — Depth of field ruined by artificial overlays. Score: 4.

Because the rubric instructed the judge to drop the overall score to 2 on any single error, the row was marked **Fail** despite individual dimensions scoring 3–4.

***

## **Tips & Best Practices**

* **Start with a small sample**: Use the sampling rate slider to import 10–20% of your logs first. Validate your judge prompt on a handful of images before scaling to the full dataset.
* **Use a capable multimodal judge**: Image evaluation requires a model that can process images as input. Text-only models will not produce meaningful scores.
* **Be specific in your rubric**: Vague judge prompts produce inconsistent scores. Break your evaluation into named dimensions (e.g. material realism, physical plausibility, prompt adherence) and score each separately.
* **Watch out for harsh aggregation rules**: Instructions like "drop the overall score to 2 if there is any error" make pass rates collapse to 0%. That's useful for strict compliance gates, but for quality tracking prefer an average across dimensions so you can see movement between runs.
* **Set the pass threshold deliberately**: The threshold defines what "good enough" means for your use case. Pick it before you look at the first set of scores so it isn't fitted to one run.
* **Monitor costs**: Image judge calls are more expensive than text, especially at higher resolutions. The credit estimate before each run tells you what to expect.

***

## **Relationship to Custom Evaluations**

Image Evals are an extension of FastRouter's [Custom Evaluations](custom-evaluations.md) feature. The same infrastructure — dataset management, run comparison, judge configuration, and results dashboard — applies to both. The key difference is the data source: instead of importing chat completion logs or JSON files, you import image generation logs via the **Images** tab in the Import Test Data dialog.

All judge configuration options available for text evals (scoring rubrics, variable interpolation, multi-criteria graders) are fully supported for image evals.
