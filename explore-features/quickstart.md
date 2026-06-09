---
description: >-
  The Dashboard and Activity Log are essential for monitoring, analyzing, and
  optimizing your API usage with large language models (LLMs).
icon: bullseye-arrow
---

# Dashboard

### Overview

The Dashboard and Activity Log in FastRouter provide comprehensive monitoring and analytics for API interactions involving LLMs. The Activity Log offers real-time tracking of requests and responses, while the Dashboard delivers interactive visualizations for deeper insights into usage, performance, and costs.

{% embed url="https://www.youtube.com/watch?v=sj6FYlSutOI" %}

### Dashboard Metrics

* **Requests**: Total number of API calls routed via FastRouter.
* **Total Cost**: Aggregated dollar value for all requests routed.
* **Total Prompt Tokens**: Cumulative count of input tokens across requests.
* **Total Completion Tokens**: Cumulative count of output tokens returned/used.
* **Averages**: Includes average input/output tokens per request and average cost.

### Dashboard Charts

* **Requests per Day**: Daily breakdown of routed requests.
* **Cost per Day**: Trend of API usage cost over time.
* **All Keys**: Usage per key alias. Click to filter per key selected.
* **All Models**: Usage per model. Click to filter per model selected.
* **All Providers**: Usage per providers that received the routed traffic. Click to filter per provider selected.
* **Errors**: Error occurrences over time by type and provider.
* **Model Response Time per Day**: Trend of average model response time per request.
* **Model Response Time Quantiles**: Breakdown (P50, P90, P99) of latency distributions.
* **Time to First Token per Day**: Measures responsiveness; how quickly the first token is received.
* **Request Distribution by Country**: Measures requests received by originating location.

### Activity Log

The **Activity Log** provides a real-time view of all API requests and responses—allowing you to monitor, troubleshoot, and audit usage effectively.

> **Note:** Activity Log request and response data is available only if **Content Logging** is not disabled for the corresponding key.

**Overview**

The Activity Log streams every request and response associated with that key, helping teams analyze performance, debug issues, and ensure compliance.

**Key Features**

* Click any log entry to view full request and response details.
* Quickly filter logs by key, model, or specific text in inputs/outputs.
* Adjust visible columns in the log using the column visibility toggle to focus on the most relevant information.

