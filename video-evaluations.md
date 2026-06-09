---
description: >-
  Evaluate AI-generated videos at scale using LLM-based judges, with automated
  scoring across motion, sync, quality, and prompt adherence.
icon: video
---

# Video Evaluations

#### Introduction

FastRouter's Video Evaluations feature lets you assess the quality of AI-generated videos at scale. By importing video generation logs, defining LLM-based judging criteria, and running evaluations against those outputs, you can systematically measure video quality across dimensions like motion fidelity, audio-visual sync, cinematic quality, and adherence to the original prompt.

Video Evals work within the same Custom Evaluations infrastructure as text and image evals — the same judge configuration, the same scoring rubrics, and the same results dashboard — extended to support multimodal video output.

***

#### Key Benefits

* Evaluate AI-generated video outputs automatically using an LLM judge.
* Import video generation logs directly from your FastRouter activity — no manual uploads required.
* Use the same custom criteria and Auto Grader setup as text evaluations.
* Drill down into per-video judge reasoning to understand exactly what scored well or poorly.

***

#### Creating a Video Evaluation

Navigate to the **Evaluations** section in your FastRouter dashboard and click **Create Evaluation**.

<figure><img src=".gitbook/assets/Evaluation Dashboard.png" alt=""><figcaption><p>Custom Evaluations</p></figcaption></figure>

**Step 1 — Name Your Evaluation**

Provide a descriptive name (e.g., `Video Compliance Evaluation` or `Product-Video-Quality-Check-v2`).

<figure><img src=".gitbook/assets/New Evaluation.png" alt=""><figcaption><p>Name Your Evaluation</p></figcaption></figure>

**Step 2 — Import Video Logs**

Click **Import Data**. In the Import Test Data dialog, select the **Videos** tab.

<figure><img src=".gitbook/assets/Import Test Data - Video.png" alt=""><figcaption></figcaption></figure>

Configure the following fields:

* **Date Range** _(required)_: Select the date range covering the video generations you want to evaluate.
* **Model** _(required)_: Choose the video generation model whose outputs you want to import (e.g., `google/veo3.1-lite`).
* **Project**: Optionally filter by project. Select a project to narrow the available API keys, or leave as "All Projects" to see all keys.
* **Key**: Optionally filter by a specific API key used during generation.
* **Input contains**: Search for specific text in the generation input to narrow down which logs are imported.
* **Sampling rate (%)**: Set a percentage of matching logs to import (1–100%). Useful for large log sets — start with a smaller sample to validate your setup before scaling.

> ℹ️ Video file logs are available for import approximately **2 hours** after they are generated. If your recent video logs aren't showing up, try again later.

Click **Import** to load the video generation logs as your evaluation dataset.

**Step 3 — Add Evaluation Metrics**

Click **Add Metric** and configure an **Auto Grader** (LLM-based judge) for your video outputs.

* **Judge Model**: Select a capable multimodal model (e.g., `gemini-3.1-flash-lite-preview`) that can process video as input.
*   **System Prompt**: Describe the evaluator's role and scoring approach. Example:

    ```
    You are an expert AI response evaluator tasked with assessing model outputs
    for quality and effectiveness. Evaluate video outputs across three dimensions:
    1. Major errors, safety concerns, or failure to perform the core task.
    2. Minor issues such as artifacts, animation inconsistencies, or audio-visual sync problems.
    3. Suggestions for higher quality animation, better motion dynamics, and tighter sound integration.
    ```
* **Scoring**: Define a numeric scale (e.g., 0–10) or pass/fail threshold. The judge will return both a score and structured reasoning per dimension.
* **Variables**: Reference the video output in your judge prompt using `{{sample.output}}`. Use `{{item.input}}` to pass the original generation prompt to the judge for context.

**Step 4 — Select Evaluation API Key**

Choose an API key from your account. This key will be used for all LLM judge calls during the evaluation.

**Step 4 — Run**

Click **Run** to start the evaluation. FastRouter will apply your judge asynchronously to each video in the dataset and return scored results.

<figure><img src=".gitbook/assets/Import Preview.png" alt=""><figcaption></figcaption></figure>

***

#### Viewing Results

Access results from the **Evaluations** listing page by clicking your evaluation.

* **Data view**: See each video row with its generation input, a video preview thumbnail, Auto Grader score, latency, and cost.
*

    <figure><img src=".gitbook/assets/Content Preview.png" alt=""><figcaption><p>Content Preview</p></figcaption></figure>
* **Report view**: Aggregated metrics across all rows — average score, pass rate, latency distribution, and cost.

<figure><img src=".gitbook/assets/Run Details.png" alt=""><figcaption><p>Report Details</p></figcaption></figure>

* **Judge Reasoning**: Click any individual row score to expand the full LLM judge reasoning — broken down by evaluation dimension (e.g., safety check → minor issues → improvement suggestions).

<figure><img src=".gitbook/assets/Judge Feedback.png" alt=""><figcaption><p>Judge Feedback</p></figcaption></figure>

**Example output for a tiger image-to-video eval (google/veo3.1-lite):**

| Metric            | Value          |
| ----------------- | -------------- |
| Auto Grader Score | Pass: 5.5 / 10 |
| Latency           | 1,115 ms       |
| Cost              | μ$400,000      |
| Video Length      | 8 seconds      |

Judge reasoning summary:

1. **Major errors / safety**: None identified.
2. **Minor issues**: Animation was extremely subtle (near-static); audio present but not closely synchronized with the visual action.
3. **Improvements**: Increase motion complexity (eye blinking, ear movement, water ripples); tighten audio-visual sync to specific visual moments.

***

#### Tips & Best Practices

* **Start with a small sample**: Use the sampling rate slider to import 10–20% of your logs first. Validate your judge prompt on a handful of videos before scaling to your full dataset.
* **Use a capable judge model**: Video evaluation requires a multimodal LLM that can process video frames. Choose models that support video input explicitly.
* **Be specific in your rubric**: Vague judge prompts produce inconsistent scores. Break your evaluation into named dimensions (e.g., motion quality, prompt adherence, audio sync) and score each separately.
* **Allow 2 hours post-generation**: Video logs take approximately 2 hours to become available for import. Plan your eval runs accordingly.
* **Monitor costs**: Video judge calls can be more expensive than text, especially with longer clips. Start with shorter videos and efficient judge models.

***

#### Relationship to Custom Evaluations

Video Evals are an extension of FastRouter's [Custom Evaluations](https://claude.ai/chat/custom-evaluations.md) feature. The same infrastructure — dataset management, run comparison, judge configuration, and results dashboard — applies to both. The key difference is the data source: instead of importing chat completion logs or CSV files, you import video generation logs via the **Videos** tab in the Import Test Data dialog.

All judge configuration options available for text evals (scoring rubrics, variable interpolation, multi-criteria graders) are fully supported for video evals.
