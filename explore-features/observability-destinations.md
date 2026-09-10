---
description: >-
  Own your observability data. Stream request traces from FastRouter straight
  into your own Kafka cluster.
icon: up-from-line
---

# Observability Destinations

> **Beta:** Observability Destinations is currently available to select customers. To request access, contact: [support@fastrouter.ai](mailto:support@fastrouter.ai).

## Overview

FastRouter records latency, tokens, cost, and the full request and response for every call you route through it. Observability let you keep that data in your own infrastructure: connect a Kafka cluster (or ClickHouse database - coming soon), and FastRouter streams each request trace to it.

* **Unlimited retention in your stack.** Traces live in storage you control, for as long as your own policies allow.
* **Query with your own tooling.** Analyze traces with the dashboards, warehouses, and pipelines your team already runs.
* **Credentials stay encrypted.** Connection credentials are stored encrypted by FastRouter.

**Common use cases:**

* **Long-term audit and compliance** by retaining every request and response beyond the Activity Log window
* **Custom analytics** by joining LLM cost and latency data with your own product and business data
* **Real-time pipelines** that feed traces into alerting, anomaly detection, or data quality jobs you already operate

***

## Supported Destinations

| Destination             | How traces are delivered                                   |
| ----------------------- | ---------------------------------------------------------- |
| Kafka                   | Each trace is produced as a message to a topic you choose. |
| ClickHouse: Coming Soon | Each trace is written to your ClickHouse database.         |

***

## How It Works

1. You add a destination with its connection details and, optionally, filters.
2. Once the destination is saved, FastRouter forwards the trace for every request that matches your filters.
3. From there, retention, access control, and querying are governed entirely by your own infrastructure.

Traces continue to appear in the FastRouter [Dashboard & Activity Log](quickstart.md#activity-log) as usual. Destinations are an additional copy, not a replacement.

<figure><img src="../.gitbook/assets/Screenshot 2026-09-10 at 11.05.27 AM.png" alt=""><figcaption><p>Access Observability from the Monitor section of the sidebar</p></figcaption></figure>

***

## Prerequisites **For Kafka:**

* One or more brokers reachable from FastRouter over the network
* A listener configured for `SASL_SSL` with the `PLAIN` mechanism
* An existing topic for traces. FastRouter does not create topics.
* A SASL user with permission to produce to that topic

> Create a dedicated user for FastRouter with write access to the traces topic only. This keeps the credential's blast radius small and makes rotation straightforward.

***

## Add a Kafka Destination

1. In the FastRouter dashboard, go to **Observability** and click **Add Destination**.
2. Under **Destination Type**, select **Kafka**.
3. Fill in the **Kafka Connection** and **Authentication** fields described below.
4. Optionally, set **Filters** to limit which traces are forwarded.
5. Click **Test Connection**, then **Send Test Trace** to verify the setup.
6. Click **Create Destination**. Traces start flowing once saved.

**Kafka Connection**

| Field             | Required | Description                                                              | Example                         |
| ----------------- | -------- | ------------------------------------------------------------------------ | ------------------------------- |
| Bootstrap Servers | Yes      | `host:port` of your Kafka broker. Separate multiple brokers with commas. | `kafka-broker.example.com:9093` |
| Topic             | Yes      | The topic traces are produced to. It must already exist.                 | `fastrouter-traces`             |

Listing more than one broker in **Bootstrap Servers** lets FastRouter discover your cluster even if one broker is temporarily unavailable.

**Authentication**

| Field             | Required | Description                                                                                    | Example            |
| ----------------- | -------- | ---------------------------------------------------------------------------------------------- | ------------------ |
| Security Protocol | Yes      | How FastRouter connects and authenticates. See the options below.                              | `SASL_SSL (PLAIN)` |
| Username          | Yes      | The SASL username FastRouter authenticates with.                                               | `fastrouter-user`  |
| Password          | Yes      | The SASL password for that user. Stored encrypted. Use the eye icon to reveal it while typing. | —                  |

**Security protocol options:**

| Option             | Description                                                   |
| ------------------ | ------------------------------------------------------------- |
| `SASL_SSL (PLAIN)` | Credentials are sent over TLS using the PLAIN SASL mechanism. |

***

## Filters

Filters are optional and apply to every destination type. Leave both set to all to stream every request.

| Filter  | Default      |
| ------- | ------------ |
| Project | All Projects |
| API Key | All API Keys |

When filters are set, a trace is forwarded only if it matches both.

***

## Test Before You Save

The Add Destination form includes two checks. Run both before clicking **Create Destination**.

| Action          | What it verifies                                                                                 |
| --------------- | ------------------------------------------------------------------------------------------------ |
| Test Connection | FastRouter can reach your destination and authenticate with the credentials you entered.         |
| Send Test Trace | End-to-end delivery. FastRouter sends a sample trace so you can confirm it arrives on your side. |

To confirm the test trace arrived in Kafka, read from the topic with a consumer that uses the same kind of credentials:

Create a `client.properties` file:

```properties
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="YOUR_USERNAME" password="YOUR_PASSWORD";
```

Then consume from the topic:

```bash
kafka-console-consumer.sh \
  --bootstrap-server kafka-broker.example.com:9093 \
  --topic fastrouter-traces \
  --consumer.config client.properties \
  --from-beginning
```

***

## Trace Format

Each exported trace is one record per request routed through FastRouter. Fields ending in `Nanos` are durations in nanoseconds.

If you use Tracing, the `traceparent` header you sent is included twice: once as the full value and once split into its components. You can use these fields to regroup multi-step workflows on your side.

**Request**

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td><code>transactionId</code></td><td>Unique identifier for the transaction in FastRouter.</td></tr><tr><td><code>requestId</code></td><td>Unique identifier for the API request.</td></tr><tr><td><code>fr_task_id</code></td><td>FastRouter task ID for asynchronous jobs, such as image and video generation.</td></tr><tr><td><code>stream</code></td><td>Whether the request was a streaming request.</td></tr><tr><td><code>statusCode</code></td><td>HTTP status code returned for the request.</td></tr><tr><td><code>errorFromProvider</code></td><td>Error returned by the upstream provider, if the request failed.</td></tr><tr><td><code>storage_url</code></td><td>Location of the stored request and response payload.</td></tr><tr><td><code>tags</code></td><td>Dynamic tags attached to the request.</td></tr></tbody></table>

**Organization & Access**

| Field       | Description                                                          |
| ----------- | -------------------------------------------------------------------- |
| `orgId`     | ID of your FastRouter organization.                                  |
| `projectId` | ID of the project the request belongs to.                            |
| `apiKey`    | The FastRouter API key hash used for the request.                    |
| `isByokKey` | Whether the request was routed through your own provider key (BYOK). |

**Model & Routing**

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td><code>modelId</code></td><td>ID of the model that served the request.</td></tr><tr><td><code>provider</code></td><td>Provider that served the request.</td></tr><tr><td><code>autoRouter</code></td><td>Whether the request used Automatic Model Selection (<code>fastrouter/auto</code>).</td></tr><tr><td><code>strategyId</code></td><td>ID of the provider routing strategy applied.</td></tr><tr><td><code>strategyType</code></td><td>Type of provider routing strategy applied, such as lowest price or lowest latency.</td></tr><tr><td><code>service_tier_input</code></td><td>Service tier requested, such as flex.</td></tr><tr><td><code>service_tier_applied</code></td><td>Service tier actually applied by the provider.</td></tr></tbody></table>

**Timing & Performance**

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td><code>queryTime</code></td><td>Timestamp of the request.</td></tr><tr><td><code>hour</code></td><td>Hour bucket of the request, useful for partitioning and aggregation.</td></tr><tr><td><code>requestReceivedAtRouter</code></td><td>Timestamp when FastRouter received the request.</td></tr><tr><td><code>responseFromRouter</code></td><td>Timestamp when FastRouter returned the response.</td></tr><tr><td><code>responseTimeNanos</code></td><td>Total response time, in nanoseconds.</td></tr><tr><td><code>timeToFirstTokenNanos</code></td><td>Time to first token, in nanoseconds.</td></tr><tr><td><code>timeToLastTokenNanos</code></td><td>Time to last token, in nanoseconds.</td></tr><tr><td><code>tokenPerMinute</code></td><td>Output throughput, in tokens per minute.</td></tr></tbody></table>

**Token Usage**

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td><code>inputTokenSize</code></td><td>Total input tokens.</td></tr><tr><td><code>outputTokenSize</code></td><td>Total output tokens.</td></tr><tr><td><code>textInputTokens</code></td><td>Text input tokens.</td></tr><tr><td><code>textOutputTokens</code></td><td>Text output tokens.</td></tr><tr><td><code>audioInputTokens</code></td><td>Audio input tokens.</td></tr><tr><td><code>audioOutputTokens</code></td><td>Audio output tokens.</td></tr><tr><td><code>imageTokens</code></td><td>Image tokens.</td></tr><tr><td><code>videoTokens</code></td><td>Video tokens.</td></tr><tr><td><code>reasoningTokens</code></td><td>Reasoning tokens generated by the model.</td></tr><tr><td><code>input_tokens_cached</code></td><td>Input tokens read from the prompt cache.</td></tr><tr><td><code>output_tokens_cached</code></td><td>Output tokens served from cache.</td></tr><tr><td><code>cachedTextTokens</code></td><td>Cached text tokens.</td></tr><tr><td><code>cachedAudioTokens</code></td><td>Cached audio tokens.</td></tr></tbody></table>

**Web Search**

| Field               | Description                                                            |
| ------------------- | ---------------------------------------------------------------------- |
| `searchContextSize` | Search context size used for web search, such as low, medium, or high. |
| `citationTokens`    | Tokens consumed by citations in web search results.                    |

**Cost**

| Field         | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| `creditsUsed` | FastRouter credits charged for the request.                  |
| `byokCost`    | Provider cost of the request when using your own key (BYOK). |

**Tracing & Session**

<table data-search="false"><thead><tr><th>Field</th><th>Description</th></tr></thead><tbody><tr><td><code>input_user_id</code></td><td>User identifier supplied with the request.</td></tr><tr><td><code>input_session_id</code></td><td>Session identifier supplied with the request.</td></tr><tr><td><code>input_trace_id</code></td><td>Trace identifier supplied with the request.</td></tr><tr><td><code>trace_parent</code></td><td>Full <code>traceparent</code> header value.</td></tr><tr><td><code>trace_parent_version</code></td><td>Version field of the <code>traceparent</code> header (<code>00</code>).</td></tr><tr><td><code>trace_parent_trace_id</code></td><td>Trace ID from the <code>traceparent</code> header (32 hex characters).</td></tr><tr><td><code>trace_parent_parent_id</code></td><td>Caller span ID from the <code>traceparent</code> header (16 hex characters).</td></tr><tr><td><code>trace_parent_flags</code></td><td>Flags field of the <code>traceparent</code> header.</td></tr></tbody></table>

**Prompt**

| Field               | Description                                       |
| ------------------- | ------------------------------------------------- |
| `prompt_id`         | ID of the Prompt Library prompt, if one was used. |
| `prompt_version_id` | Version of the Prompt Library prompt used.        |
| `prompt_type`       | Type of prompt.                                   |
| `prompt_content`    | Content of the prompt.                            |

**Prompt Compression**

| Field                           | Description                             |
| ------------------------------- | --------------------------------------- |
| `compressedRequest`             | Whether Prompt Compression was applied. |
| `compressionEngine`             | Compression engine used.                |
| `compressionTokensBefore`       | Input tokens before compression.        |
| `compressionTokensAfter`        | Input tokens after compression.         |
| `compressionEstimatedCostSaved` | Estimated cost saved by compression     |

***

## Security & Data Handling

* **Credentials are encrypted.** Destination credentials are stored encrypted by FastRouter.
* **Traffic is encrypted in transit.** With `SASL_SSL`, both credentials and trace payloads are sent over TLS.
* **Traces contain full payloads.** Prompts, completions, and any personal data inside them are forwarded as-is. Restrict access to your topic or table accordingly, and apply your own retention and deletion policies.
* **Use least privilege.** Give the FastRouter user permission to write to the traces topic or table only.

***

## Troubleshooting

| Symptom                              | What to check                                                                                                                                                         |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Test Connection fails to connect     | Confirm each `host:port` is correct and that the port is your `SASL_SSL` listener. Check that firewalls or security groups allow inbound connections from FastRouter. |
| Test Connection fails authentication | Verify the username and password, and that the PLAIN mechanism is enabled on the broker listener.                                                                     |
| Send Test Trace fails                | Make sure the topic exists (FastRouter does not auto-create it) and that the user has permission to write to it.                                                      |
| Test trace sent but not visible      | Check your consumer is reading the right topic, from the earliest offset, with valid credentials.                                                                     |
| No traces after saving               | Confirm the destination was saved with **Create Destination**, and that your Project and API Key filters match the traffic you're sending.                            |

***
