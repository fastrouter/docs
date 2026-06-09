---
description: >-
  FastRouter’s Custom Evaluations lets you benchmark and compare AI models on
  your own data—using LLM-based judges to automatically score accuracy, latency,
  and cost.
icon: star-exclamation
---

# Custom Evaluations

### **Introduction**

FastRouter's Custom Evaluations feature allows you to assess and compare the performance of AI models on your datasets. By importing chat completion logs or datasets, generating model outputs (runs), and defining evaluation criteria with LLM-based judges, you can quantitatively measure aspects like accuracy, relevance, latency, and cost. This is ideal for benchmarking models, optimizing prompts, and ensuring high-quality responses in production.

Evaluations are managed through the FastRouter dashboard, where you can create, run, and analyze evaluations asynchronously. Results include detailed metrics and scores, helping you make data-driven decisions.

{% embed url="https://youtu.be/ISDIMfOA6J4" %}

***

### **Key Benefits**

* Automated judging using LLM evaluators for scalable assessments.
* Support for custom criteria tailored to your use case (e.g., factual accuracy, creativity, conciseness).
* Integration with your API keys for secure, cost-effective processing.
* Visual dashboards for easy comparison of runs and metrics.

***

### **Creating a New Evaluation**

To start, navigate to the "Evaluations" section in your FastRouter dashboard and click "New Evaluation."

<figure><img src=".gitbook/assets/New Evaluation (1).png" alt=""><figcaption><p>Custom Evaluations: Create Evaluation</p></figcaption></figure>

1. **Name Your Evaluation:** Provide a descriptive name (e.g., "Math Query Benchmark").
2. **Import Test Data:** Upload or import chat completion logs or datasets. You can:
   * Select a project and model (e.g., "Anthropic Claude 4.5").
   * Filter by date range.
   * Choose input/output text to filter rows.
   * Set a sample size (e.g., 10%) to evaluate a subset of your data for efficiency.

<figure><img src=".gitbook/assets/Import Test Data.png" alt=""><figcaption><p>Custom Evaluations: Import Test Data</p></figcaption></figure>

3. **Add Runs:** Select model runs to generate outputs (e.g., "anthropic/claude-4.5"). You can add multiple runs for side-by-side comparison.

<figure><img src=".gitbook/assets/Add Run.png" alt=""><figcaption><p>Custom Evaluations: Add Run</p></figcaption></figure>

4. **Add Test Criteria:** Create one or more evaluation criteria using an LLM judge.

* Click "Add Test Criteria" and choose a type (e.g., Model Scorer for quantitative scoring).
* Configure the LLM judge: Select a model (e.g., "openai/gpt-5"), system prompt (e.g., "You are an expert AI response evaluator..."), and user prompt template (e.g., "Rate the response on \[criteria] from 1-10") with any variables.
* To access any values for evaluation by the LLM judge in the input or generated output, you can use the variables: _\{{item.input\}}, \{{item.column\_name\}} or \{{sample.output\}}._
* Define scoring rubrics, such as pass/fail thresholds or numeric scales.

<figure><img src=".gitbook/assets/Add Test Criteria.png" alt=""><figcaption><p>Custom Evaluations: Add Test Criteria</p></figcaption></figure>

<figure><img src=".gitbook/assets/Test Criteria.png" alt=""><figcaption><p>Custom Evaluations: Edit Test Criteria</p></figcaption></figure>



5. **Select an Evaluation Key:** Choose an API key from your account to handle generation and evaluation requests. This key will be used for all API calls during the evaluation.

<figure><img src=".gitbook/assets/New Evaluation 2.png" alt=""><figcaption><p>Custom Evaluations: Configure Evaluation Key</p></figcaption></figure>

6. **Run the Evaluation:** Click "Run" to start processing. The evaluation will generate outputs for each run and apply the judges asynchronously.

***

### **Viewing Evaluation Results**

Once the evaluation is complete, access the results of a particular custom evaluation from the Evaluations listing page.

* **Overview:** See a summary of runs, including model names, request IDs, input samples, generated outputs, testing criteria scores (e.g., Latency, Cost), and overall scores.

<figure><img src=".gitbook/assets/Evaluation Run Overview.png" alt=""><figcaption><p>Custom Evaluations: Evaluation Runs Overview</p></figcaption></figure>

* **Comparison and Analysis:** Compare multiple runs side-by-side. Metrics include:
  * **Score:** Aggregated from your criteria (e.g., 7/10 for accuracy).
  * **Latency:** Time to generate responses.
  * **Cost:** Token-based billing.
  * Custom metrics based on your judges.
* **Detailed Metrics:** For each run, view aggregated stats like average score, latency (in ms), cost, and pass rate. Drill down into individual responses for judge reasoning.
* **Detailed Metrics:** For each run, view aggregated stats like average score, latency (in ms), cost, and pass rate. Drill down into individual responses for judge reasoning.

<figure><img src=".gitbook/assets/Evaluation Run Details.png" alt=""><figcaption><p>Custom Evaluations: Evaluation Run Details</p></figcaption></figure>

* **Judge Reasoning:** For each test criterion and score, you can drill down into the individual responses for details of the judge reasoning.

<figure><img src=".gitbook/assets/Judge Reasoning.png" alt=""><figcaption><p>Custom Evaluations: Judge Reasoning</p></figcaption></figure>

***

### **Tips & Best Practices**

* **Start Small:** Begin with a small sample size (e.g., 10-50 rows) to test your setup before scaling to larger datasets.
* **Diverse Criteria:** Use multiple judges for comprehensive evaluations (e.g., one for factual accuracy, another for response conciseness).
* **Judge Calibration:** Test your LLM judge prompts on sample data to ensure unbiased and consistent scoring.
* **Cost Management:** Monitor estimated costs in the setup phase. Use efficient models for judges to minimize expenses.
