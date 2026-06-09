---
description: >-
  Alerts help you monitor your API usage and performance in real-time. Set up
  custom thresholds to get notified when metrics exceed expected values or
  change compared to historical baselines.
icon: siren-on
---

# Alerts

### Overview

Alerts evaluate your selected metrics at regular intervals and notify you when conditions are met. Each alert can have two severity levels:

* **Warning** — Early indicator that a metric is trending toward a problem
* **Critical** — Immediate attention required; metric has exceeded acceptable limits

Alerts are scoped to specific Projects, API Keys, and Models, giving you granular control over what you monitor.

<figure><img src=".gitbook/assets/Screenshot 2026-04-02 at 5.35.54 PM.png" alt=""><figcaption><p>Select metric, name and scope</p></figcaption></figure>

<figure><img src=".gitbook/assets/Screenshot 2026-04-02 at 5.35.44 PM.png" alt=""><figcaption><p>Set threshold and subscribe to alert</p></figcaption></figure>

***

### Available Metrics

#### Performance

| Metric                  | Description                                                                               | Unit  |
| ----------------------- | ----------------------------------------------------------------------------------------- | ----- |
| **Response Time**       | Median (p50) end-to-end latency from request receipt to response completion               | ms    |
| **Time to First Token** | Median (p50) time from request submission until the first response token begins streaming | ms    |
| **Throughput**          | Median request throughput for streaming responses (`stream = true`)                       | req/s |

#### Reliability

| Metric          | Description                                                  | Unit   |
| --------------- | ------------------------------------------------------------ | ------ |
| **Error Count** | Total number of failed requests within the evaluation window | errors |
| **Error Rate**  | Percentage of requests that failed                           | %      |

#### Usage & Cost

| Metric                | Description                                                      | Unit     |
| --------------------- | ---------------------------------------------------------------- | -------- |
| **Token Consumption** | Total tokens consumed (input + output)                           | tokens   |
| **Daily Spend**       | Cumulative spending for the current day (resets at midnight UTC) | $        |
| **Monthly Spend**     | Cumulative spending for the current month (resets on the 1st)    | $        |
| **Total Requests**    | Total number of API requests                                     | requests |

***

### Scoping Alerts

Each alert can be scoped to monitor specific subsets of your traffic:

#### Projects

Select which projects to include in the alert evaluation. You can choose:

* **All Projects** — Monitor aggregate metrics across your entire organization
* **Specific Projects** — Monitor one or more selected projects

#### API Keys

Select which API keys to include:

* **All Keys** — Monitor all API keys within the selected projects
* **Specific Keys** — Monitor one or more selected API keys

#### Models

Select which models to include:

* **All Models** — Monitor requests to any model
* **Specific Models** — Monitor requests to selected models only (e.g., only GPT-4.1 and Claude 4.5 Sonnet)

> **Example**: You could create an alert that monitors Response Time only for your "Production" project, only for requests using a particular API key, and only for GPT-4.1 requests.

***

### Alert Types

#### Static Value

Triggers when the metric crosses a fixed threshold.

**Use cases:**

* Response Time > 2000ms
* Error Rate > 5%
* Daily Spend > $500

**Configuration:**

| Field              | Description                                     |
| ------------------ | ----------------------------------------------- |
| Condition          | `Above` or `Below`                              |
| Warning Threshold  | Value that triggers a warning (optional)        |
| Critical Threshold | Value that triggers a critical alert (required) |

#### Percentage Change

Triggers when the metric changes by a specified percentage compared to a historical baseline.

**Use cases:**

* Response Time increased 50% vs same time yesterday
* Request volume dropped 30% vs same time last week
* Error Count spiked 100% vs previous hour

**Configuration:**

| Field                  | Description                                      |
| ---------------------- | ------------------------------------------------ |
| Comparison Period      | Historical baseline to compare against           |
| Condition              | `Above` or `Below`                               |
| Warning Threshold (%)  | Percentage change that triggers a warning        |
| Critical Threshold (%) | Percentage change that triggers a critical alert |

***

### Evaluation Interval

The evaluation interval determines how often the alert checks your metrics. Choose based on how quickly you need to detect issues:

| Interval   | Best For                         | Trade-off                                            |
| ---------- | -------------------------------- | ---------------------------------------------------- |
| **5 min**  | Error Rate, Error Count          | Fastest detection, may be noisy for volatile metrics |
| **15 min** | Response Time, TTFT, Throughput  | Balances speed and noise reduction                   |
| **30 min** | Response Time, Token Consumption | Smooths transient spikes                             |
| **1 hour** | Spend metrics, Usage patterns    | Good for slow-changing metrics                       |
| **Daily**  | Daily Spend, Monthly Spend       | End-of-day summaries                                 |

#### Recommendations by Metric

| Metric                   | Recommended Interval |
| ------------------------ | -------------------- |
| Error Rate / Error Count | 5 min                |
| Response Time / TTFT     | 15–30 min            |
| Throughput               | 15 min               |
| Token Consumption        | 30–60 min            |
| Daily/Monthly Spend      | 1 hour or Daily      |

***

### Comparison Periods

When using **Percentage Change** alerts, you compare the current value against a historical baseline. The comparison period determines which historical window to use.

| Comparison Period         | What It Compares                   | Best For                                           |
| ------------------------- | ---------------------------------- | -------------------------------------------------- |
| **Previous Period**       | The immediately preceding interval | Detecting sudden spikes                            |
| **Same time 1 hour ago**  | Same interval, 1 hour earlier      | Intra-day patterns                                 |
| **Same time 1 day ago**   | Same interval, 24 hours earlier    | Daily patterns (e.g., business hours vs off-hours) |
| **Same time 1 week ago**  | Same interval, 7 days earlier      | Weekly patterns (e.g., weekday vs weekend)         |
| **Same time 1 month ago** | Same interval, 30 days earlier     | Monthly patterns, seasonal trends                  |

#### How Comparison Works

The system compares two time windows of equal length:

```
Current Window:     [T - interval, T]
Previous Window:    [T - interval - offset, T - offset]
```

**Example:** Alert runs at 3:00 PM with a 15-minute interval, comparing to "Same time 1 day ago"

```
Current Window:     2:45 PM – 3:00 PM today
Previous Window:    2:45 PM – 3:00 PM yesterday
```

#### Percentage Change Formula

```
% Change = ((Current Value - Previous Value) / Previous Value) × 100
```

**Example:**

* Current Response Time: 450ms
* Previous Response Time: 300ms
* % Change: ((450 - 300) / 300) × 100 = **50%**

If your alert threshold is "Above 40%", this would trigger.

***

### Thresholds

Each alert supports two threshold levels:

#### Warning Threshold

An early indicator that the metric is trending toward a problem. Useful for:

* Getting advance notice before issues become critical
* Allowing time to investigate before escalation
* Tracking trends that may need attention

#### Critical Threshold

The primary alert trigger indicating immediate attention is needed. This is the main threshold that should reflect your SLA or operational limits.

#### Condition Direction

| Condition | Meaning                                 | Typical Use                      |
| --------- | --------------------------------------- | -------------------------------- |
| **Above** | Alert when metric exceeds threshold     | Response Time, Error Rate, Spend |
| **Below** | Alert when metric falls below threshold | Throughput, Request Volume       |

> **Tip:** Set your Warning threshold at \~50-70% of your Critical threshold to give yourself response time.

***

### Notification Behavior

#### Notification Channels

* **Email** — Send alerts to specified email addresses
* **Organization Owners** — Automatically notify all org owners
* **Project Members** — Automatically notify all members of affected projects

#### Alert States

Alerts transition between three states:

```
        threshold breached
    ┌───────────────────────────┐
    │                           ▼
 ┌──┴──┐                   ┌─────────┐
 │ OK  │                   │ FIRING  │
 └──┬──┘                   └────┬────┘
    ▲                           │
    └───────────────────────────┘
        below threshold (reset)
```

***

### Examples

#### Example 1: High Response Time Alert

**Goal:** Get notified when API response times are slow

| Setting             | Value                                    |
| ------------------- | ---------------------------------------- |
| Metric              | Response Time                            |
| Scope               | Production project, All keys, All models |
| Alert Type          | Static Value                             |
| Condition           | Above                                    |
| Warning Threshold   | 1500 ms                                  |
| Critical Threshold  | 3000 ms                                  |
| Evaluation Interval | 15 min                                   |

**Behavior:** Every 15 minutes, calculates the median response time. If it exceeds 1500ms, a warning is sent. If it exceeds 3000ms, a critical alert is sent.

***

#### Example 2: Error Rate Spike Detection

**Goal:** Detect sudden increases in error rate compared to normal

| Setting             | Value                              |
| ------------------- | ---------------------------------- |
| Metric              | Error Rate                         |
| Scope               | All projects, All keys, All models |
| Alert Type          | Percentage Change                  |
| Comparison Period   | Same time 1 day ago                |
| Condition           | Above                              |
| Warning Threshold   | 50%                                |
| Critical Threshold  | 100%                               |
| Evaluation Interval | 5 min                              |

**Behavior:** Every 5 minutes, compares the current error rate to the same 5-minute window yesterday. If today's error rate is 50% higher, a warning is sent. If it's 100% higher (doubled), a critical alert is sent.

***

#### Example 3: Daily Spend Limit

**Goal:** Get notified before exceeding daily budget

| Setting             | Value                                    |
| ------------------- | ---------------------------------------- |
| Metric              | Daily Spend                              |
| Scope               | Production project, All keys, All models |
| Alert Type          | Static Value                             |
| Condition           | Above                                    |
| Warning Threshold   | $400                                     |
| Critical Threshold  | $500                                     |
| Evaluation Interval | 1 hour                                   |

**Behavior:** Every hour, checks the cumulative daily spend. Sends a warning at $400 and a critical alert at $500.

***

#### Example 4: Traffic Drop Detection

**Goal:** Detect if request volume suddenly drops (may indicate an outage)

| Setting             | Value                                    |
| ------------------- | ---------------------------------------- |
| Metric              | Total Requests                           |
| Scope               | Production project, All keys, All models |
| Alert Type          | Percentage Change                        |
| Comparison Period   | Same time 1 hour ago                     |
| Condition           | Below                                    |
| Warning Threshold   | 30%                                      |
| Critical Threshold  | 50%                                      |
| Evaluation Interval | 5 min                                    |

**Behavior:** Every 5 minutes, compares request count to the same period 1 hour ago. If traffic drops by 30%, a warning is sent. If it drops by 50%, a critical alert is sent.

***

### Best Practices

1. **Start with Critical thresholds only** — Add Warning thresholds once you understand your baseline metrics.
2. **Use appropriate intervals** — Don't use 5-minute intervals for metrics that naturally fluctuate; you'll get too many false positives.
3. **Leverage Percentage Change for anomalies** — Static thresholds work well for known limits, but percentage change is better for detecting unusual patterns.
4. **Scope alerts appropriately** — Create separate alerts for Production vs Staging environments rather than one alert for everything.
5. **Set up resolved notifications** — Knowing when an issue is resolved is as important as knowing when it started.
6. **Document your thresholds** — Keep a record of why you chose specific threshold values so future team members understand the rationale.

***

### FAQ

**Q: What happens if there's no data in the evaluation window?**

A: The alert maintains its current state. If there were zero requests in the window, metrics like Error Rate cannot be calculated, so the alert remains unchanged.

**Q: Can I have multiple alerts for the same metric?**

A: Yes. You might have one alert for Production and another for Staging, or different thresholds for different models.

**Q: When do cumulative metrics (Daily Spend, Monthly Spend) reset?**

A: Daily Spend resets at midnight UTC. Monthly Spend resets on the 1st of each month at midnight UTC.

**Q: What timezone are alerts evaluated in?**

A: All alert evaluations use UTC. "Same time 1 day ago" means the same UTC time yesterday.

**Q: Can I pause an alert without deleting it?**

A: Yes. You can pause an alert from the Alerts list page. Paused alerts retain their configuration but do not evaluate or send notifications.
