---
description: >-
  Evaluate AI-generated videos at scale using LLM-based judges, with automated
  scoring across motion, sync, quality, and prompt adherence.
icon: video
---

# Video Evaluations

## **Introduction**

FastRouter's Video Evaluations feature lets you assess the quality of AI-generated videos at scale. By importing video generation logs, defining LLM-based judging criteria, and running evaluations against those outputs, you can systematically measure video quality across dimensions like visual fidelity, scene flow, on-screen text accuracy, audio-visual sync, and adherence to the original prompt.

Video Evals work within the same Custom Evaluations infrastructure as text and image evals; the same judge configuration, the same scoring rubrics, and the same results dashboard extended to support multimodal video output.

***

## **Key Benefits**

* Evaluate AI-generated video outputs automatically using a multimodal LLM judge.
* Import video generation logs directly from your FastRouter activity.
* Use the same Auto grader and Custom grader setup as text and image evaluations.
* Catch failures that are hard to spot at a glance: a duplicated logo, a typo in the final frame, a scene that skips a beat with per-dimension judge reasoning for every video.

***

## **Before You Start**

Video evals run against videos you have already generated through FastRouter. You can generate them via the API, or interactively from **Model Playground** by switching to the **Video** tab and selecting a video model (e.g. `x-ai/grok-imagine-video-1.5`).

<figure><img src="../.gitbook/assets/Video Playground.png" alt=""><figcaption><p>Generating a video in Model Playground</p></figcaption></figure>

Video prompts are usually long and structured. The example used throughout this page:

```
Create a 8-second luxury jewelry ad for Star Diamonds, promoting a new diamond earring range.
Elegant, cinematic, premium aesthetic with black velvet, soft champagne-gold lighting, sparkling
highlights, and smooth transitions. Photorealistic diamonds, accurate earring symmetry, crisp
product details, refined typography, no distorted faces or jewelry.

0–2 sec: Macro shot of diamond earrings emerging from darkness, slowly rotating as light catches
each facet. On-screen copy: "A New Light Has Arrived"
2–4 sec: Elegant model wearing the earrings, turning her head subtly against a sophisticated
black-and-gold background. On-screen copy: "Made to Make You Shine"
4–6 sec: Fast, graceful montage of three distinct designs—classic studs, diamond drops, and
contemporary hoops—each sparkling in close-up. On-screen copy: "Discover the New Earring Collection"
6–8 sec: Hero product arrangement on black velvet, followed by the Star Diamonds logo and a soft
diamond-flare finish. On-screen copy: "Star Diamonds" CTA: "Find Your Perfect Pair"

Audio: Sophisticated cinematic shimmer with a gentle rising beat and a delicate crystal chime on
the final logo reveal.
```

Writing the prompt as timed beats pays off twice: the generation model has a clearer target, and you can reuse the same structure as your judge's checklist.

***

## **Creating a Video Evaluation**

Navigate to the **Evaluations** section in your FastRouter dashboard and click **Create Evaluation**.

<figure><img src="../.gitbook/assets/New Evaluation.png" alt=""><figcaption><p>New Evaluation</p></figcaption></figure>

**Step 1 — Name Your Evaluation**

Provide a descriptive name (e.g. `Video Eval` or `Jewellery-Ad-Compliance-v2`).

**Step 2 — Import Video Logs**

Click **Import Data**. In the Import Test Data dialog, select the **Videos** tab.

<figure><img src="../.gitbook/assets/Import Test Data.png" alt=""><figcaption><p>Import Test Data — Videos tab</p></figcaption></figure>

Configure the following fields:

* **Date Range** _(required)_: Select the date range covering the video generations you want to evaluate.
* **Model** _(required)_: Choose the video generation model whose outputs you want to import (e.g. `x-ai/grok-imagine-video-1.5`).
* **Project**: Optionally filter by project. Select a project to narrow the available API keys, or leave as "All Projects" to see all keys.
* **Key**: Optionally filter by a specific API key used during generation.
* **Input contains**: Search for specific text in the generation prompt to narrow down which logs are imported.
* **Sampling rate (%)**: Set a percentage of matching logs to import (1–100%). Useful for large log sets — start with a smaller sample to validate your setup before scaling.

> ℹ️ Video file logs are available for import approximately **2 hours** after they are generated. If your recent video logs aren't showing up, try again later.

The dialog shows a live **Total rows** count as you adjust the filters. Click **Import** to load the video generation logs as your evaluation dataset.

Once imported, the dataset appears as **Imported from Video logs**, and the source model is added automatically under **Models to Compare**. The **Data (preview)** panel on the right shows each row's prompt alongside its video output.

<figure><img src="../.gitbook/assets/Sample Overview.png" alt=""><figcaption><p>New Evaluation with imported video logs</p></figcaption></figure>

**Step 3 — Add Evaluation Metrics**

Click **Add Metric** and configure your judge. Two presets are available:

* **Auto grader**: a general-purpose quality judge with a ready-made rubric — good for a quick first pass.
* **Custom grader**: your own prompt, model, and scoring scale — recommended when you have a specific brief to check the video against.

<figure><img src="../.gitbook/assets/Custom Grader.png" alt=""><figcaption><p>Configuring a Custom grader</p></figcaption></figure>

Configure the following:

* **Model** _(required)_: Select a multimodal model that accepts video input (e.g. `google/gemini-3.1-pro-preview`). A text-only or image-only model cannot score video.
*   **System**: Describe the evaluator's role, the dimensions to check, and how to score them. The example used here:

    ```
    You are a video evaluator. Watch the video and check if it matches these requirements:
    1. Visual Quality: Is it a luxury jewelry ad with black velvet, gold lighting, and symmetrical,
       distortion-free diamonds?
    2. Scene Flow: Does it progress smoothly from a macro shot, to a model, to a product montage,
       and end on a hero product layout?
    3. Text & Typo Check: Read all on-screen text. Check for any typos, spelling mistakes, or
       grammatically incorrect copy.
    Score from 1 - 7. Give an overall fail score of 2 or below if any of the requirements fails to
    meet the standard.
    ```
* **Variables**: Use `{{curly braces}}` to interpolate values into your prompt. `{{item.input}}` passes the original generation prompt to the judge, which is what you need for prompt-adherence checks. The generated video itself is attached to the judge call automatically. The valid variable list is shown under each prompt box.
* **User** _(required)_: The user-turn template sent to the judge, typically the user input placeholder followed by the response to evaluate.
* **Scoring**: Define the numeric **Range** (e.g. `0–7`) and a **Pass Threshold** (e.g. `3.5`). Rows scoring at or above the threshold are marked Pass; everything below is Fail.

**Step 4 — Select Evaluation API Key**

Choose an API key from your account. This key is used for all LLM judge calls during the evaluation.

**Step 5 — Run**

Click **Run**. FastRouter first shows a **Credit Utilization Estimate** with the number of selected requests, the estimated cost, and your current balance.

Click **Proceed with Evaluation** to start the run. The judge is applied asynchronously to each video in the dataset and results are returned as they complete.

> ℹ️ The estimate is based on the selected judge model and an average prompt size — actual costs vary with clip length and resolution. If credits run out mid-run, the evaluation may not complete.

***

## **Viewing Results**

Access results from the **Evaluations** listing page by clicking your evaluation. Use the **Report** / **Data** toggle in the top right to switch views.

**Report view** — Aggregated metrics across all rows: the run and its Request ID, latency percentiles, generation cost, and the overall score for each grader. The **Test Criteria** panel below shows the judge prompt, its range and pass threshold, the judge model, and the total judge spend for the run.

<figure><img src="../.gitbook/assets/Report Details.png" alt=""><figcaption><p>Report Details</p></figcaption></figure>

> ℹ️ Latency shows **N/A** for imported video generations. Video models generate asynchronously, so per-request latency isn't captured the way it is for chat or image runs — use cost and score to compare video models.

**Data view** — One row per video with its generation input on the left and the graded output on the right, including the score, cost, and a video attachment. Use **Single View** to inspect one run. Search, sort, and fullscreen controls help with larger datasets.

<figure><img src="../.gitbook/assets/Sample Overview (1).png" alt=""><figcaption><p>Runs Overview — Data view</p></figcaption></figure>

**Content Preview** — Click the expand arrow on any row to open an inline video player next to the full input parameters, with the grader verdict and cost alongside. Play the clip here to check the judge's call yourself.

<figure><img src="../.gitbook/assets/Content Preview.png" alt=""><figcaption><p>Content Preview</p></figcaption></figure>

**Judge Reasoning** — Click the score to open **Criteria Results** and see the judge's per-dimension breakdown, along with the grader configuration that produced it.

<figure><img src="../.gitbook/assets/Judge Reasoning.png" alt=""><figcaption><p>Judge Feedback</p></figcaption></figure>

**Example output for the Star Diamonds ad eval (`x-ai/grok-imagine-video-1.5`):**

| Metric                   | Value                                                                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| Custom Grader            | Fail: 2 (Range 0–7, Pass ≥ 3.5)                                                                                              |
| Latency                  | N/A                                                                                                                          |
| Original Generation Cost | μ$1,120,000.0000 ($1.12). **Note:** This cost is only incurred during the original generation and not during the evaluation. |
| Total Judge Score Cost   | $0.0220                                                                                                                      |
| Pass Rate                | 0%                                                                                                                           |
| Video Length             | 8 seconds (720p, 16:9, audio on)                                                                                             |

Judge reasoning summary:

1. **Visual Quality** — Has the intended luxury aesthetic with velvet and gold lighting, but fails the distortion check due to visual artifacts overlapping the model's face.
2. **Scene Flow** — Passes. Follows the intended sequence from macro shot to model to montage to hero arrangement.
3. **Text Check** — Mostly spelled correctly, but fails due to poorly formatted and duplicated "Star Diamonds" text in the final scene.

Two of three dimensions failed, and the rubric instructed the judge to cap the overall score at 2 whenever any requirement misses the standard; so the row is marked **Fail** even though scene flow was clean.

***

## **Tips & Best Practices**

* **Start with a small sample**: Use the sampling rate slider to import 10–20% of your logs first. Validate your judge prompt on a handful of videos before scaling to the full dataset.
* **Use a capable multimodal judge**: Video evaluation requires a model that can process video frames and audio. Choose models that support video input explicitly.
* **Mirror your prompt structure in your rubric**: If the generation prompt is written as timed beats, make the judge check those same beats. Named dimensions (visual quality, scene flow, text accuracy) produce far more consistent scores than a single open-ended quality question.
* **Ask the judge to read on-screen text**: Typos, duplicated logos, and malformed CTAs in the final frame are the most common — and most embarrassing — failures in generated ad content, and they are easy to miss on a first watch.
* **Be deliberate with harsh aggregation rules**: "Fail the whole video if any requirement misses" is the right call for a compliance gate but collapses pass rates to 0%. For tracking quality over time, prefer an average across dimensions so you can see movement between runs.
* **Allow 2 hours post-generation**: Video logs take approximately 2 hours to become available for import. Plan your eval runs accordingly.

***

## **Relationship to Custom Evaluations**

Video Evals are an extension of FastRouter's [Custom Evaluations](custom-evaluations.md) feature. The same infrastructure — dataset management, run comparison, judge configuration, and results dashboard — applies to both. The key difference is the data source: instead of importing chat completion logs or CSV files, you import video generation logs via the **Videos** tab in the Import Test Data dialog.

All judge configuration options available for text evals (scoring rubrics, variable interpolation, multi-criteria graders) are fully supported for video evals.
