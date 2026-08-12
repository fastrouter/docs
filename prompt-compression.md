---
description: >-
  Prompt Compression intelligently shrinks prompts before they're sent to an AI
  model, reducing token usage while maintaining response quality. It helps lower
  costs and maximize available context.
icon: compress
---

# Prompt Compression

#### Overview

Opt-in compression for chat requests. Add one block to your request body and FastRouter compresses your messages before they reach the provider — cutting input tokens without changing the response.

* **Off by default** — nothing happens unless you opt in
* **Opt-in per request** — one block in the body
* **Fail-open** — if compression can't run, your original messages are sent unchanged

#### How it works

```
Client ──► FastRouter gateway ──► compression ──► Provider (OpenAI / Anthropic / …)
```

You send a normal chat request plus a small `optimize` block. The gateway compresses eligible messages, forwards the compressed payload upstream, and returns the normal provider response with `X-FastRouter-Compression-*` headers reporting savings.

#### Enabling it

Compression runs only when your request opts in — the body includes an `optimize.compress` block. Without it, the request flows through completely unchanged.

#### Choosing an engine

Pick based on your content, not the implementation:

| Your content                               | Engine value           | What it does                                                                                | Type     |
| ------------------------------------------ | ---------------------- | ------------------------------------------------------------------------------------------- | -------- |
| Strict system prompts, rules, tool schemas | `"headroom"` (default) | Structural compression — restructures JSON/repetitive payloads without dropping information | Lossless |
| Chat prose, verbose instructions           | `"caveman"`            | Rule-based compaction — strips filler and redundancy, preserves all facts and constraints   | Lossy    |
| RAG chunks, docs, transcripts              | `"llmlingua"`          | ML token pruning — a small model scores and drops low-information tokens                    | Lossy    |
| Maximum savings on mixed content           | `"all"`                | Full pipeline: headroom → llmlingua → caveman                                               | Mixed    |

`engine` accepts a single value, a preset (`"both"`/`"hybrid"` = headroom → caveman, `"all"` = the full pipeline), or an explicit list like `["headroom", "llmlingua"]`. Application order is always **headroom → llmlingua → caveman**.

> **Try before you buy:** set `"mode": "audit"` to measure what you _would_ save without changing a single byte of your request. Stats are still returned in the response headers.

#### Request format

```json
{
  "model": "openai/gpt-5.5",
  "messages": [ ... ],
  "optimize": {
    "compress": {
      "engine": "llmlingua",
      "llmlingua_rate": 0.75
    }
  }
}
```

The `compress` object is an open key/value bag — any engine parameter is forwarded as-is. Omit the block entirely and no compression happens.

#### Supported routes

| Surface                 | Endpoints                                                                              | Coverage                             |
| ----------------------- | -------------------------------------------------------------------------------------- | ------------------------------------ |
| OpenAI chat completions | `POST /v1/chat/completions`, `POST /api/v1/chat/completions`, `POST /chat/completions` | Streaming, non-streaming & SDK paths |
| Anthropic Messages      | `POST /v1/messages`, `POST /api/v1/messages`                                           | Streaming & non-streaming            |

Native Responses / Gemini handlers are not covered yet.

#### Parameters

**General**

| Param         | Type                | Default      | Description                                                                     |
| ------------- | ------------------- | ------------ | ------------------------------------------------------------------------------- |
| `engine`      | string \| string\[] | `"headroom"` | Which compressor(s) to run.                                                     |
| `mode`        | string              | `"optimize"` | `"optimize"` applies transforms; `"audit"` only observes (still returns stats). |
| `provider`    | string              | inferred     | Provider hint for token counting.                                               |
| `cache_align` | bool                | `false`      | Improve provider prompt-cache hits (does not reduce tokens).                    |

**Lossless structural (`engine: "headroom"`)**

| Param                           | Type  | Description                                |
| ------------------------------- | ----- | ------------------------------------------ |
| `target_ratio`                  | float | Desired compression ratio target.          |
| `min_tokens_to_compress`        | int   | Skip blocks smaller than this.             |
| `compress_user_messages`        | bool  | Compress user-role messages.               |
| `compress_system_messages`      | bool  | Compress system-role messages.             |
| `protect_recent` / `keep_turns` | int   | Leave the N most recent turns untouched.   |
| `protect_analysis_context`      | bool  | Protect analysis/reasoning context blocks. |

**Rule-based prose (`engine: "caveman"`)**

| Param           | Type      | Default   | Description                                 |
| --------------- | --------- | --------- | ------------------------------------------- |
| `caveman_level` | string    | `"light"` | `"light"`, `"semantic"`, or `"aggressive"`. |
| `caveman_roles` | string\[] | all roles | Restrict to given roles, e.g. `["user"]`.   |

Never drops negations, modals, quantifiers, code, URLs, paths, numbers, quoted strings, or ALL-CAPS codes.

**ML prose (`engine: "llmlingua"`)**

| Param                    | Type        | Default   | Description                                                       |
| ------------------------ | ----------- | --------- | ----------------------------------------------------------------- |
| `llmlingua_rate`         | float (0–1) | `0.75`    | Fraction of tokens to keep. Higher = gentler. `0.5` = aggressive. |
| `llmlingua_target_token` | int         | —         | Absolute token budget (overrides rate).                           |
| `llmlingua_roles`        | string\[]   | all roles | Restrict to given roles, e.g. `["user"]`.                         |

> The first `llmlingua` request loads the ML model (\~70s). Later requests are fast. `headroom` and `caveman` have no load cost.

***

### Examples by engine

Each example below is a full request body. Long message content is shortened to `… (long prose)` for readability — swap in your real payload. After the request runs, confirm compression with the `X-FastRouter-Compression-*` response headers.

#### `headroom` — structural / lossless

**Best for** strict system prompts, repeated rule blocks, and tool/function schemas. `headroom` restructures repetitive and machine-readable payloads without dropping information, so it's the safe default when correctness matters. Here it targets a system prompt whose rule block is repeated verbatim, plus a tool schema — both highly compressible with zero information loss.

```bash
{
    "model": "openai/gpt-4.1-mini",
    "stream": false,
    "max_tokens": 500,
    "messages": [
        {
            "role": "system",
            "content": "You are a data-analysis assistant."
        },
        {
            "role": "user",
            "content": "Summarize failures, pending, revenue, and regions."
        },
        {
            "role": "assistant",
            "content": null,
            "tool_calls": [
                {
                    "id": "call_transactions_001",
                    "type": "function",
                    "function": {
                        "name": "get_transactions",
                        "arguments": "{\"start_date\":\"2026-07-01\",\"end_date\":\"2026-07-17\",\"limit\":10000}"
                    }
                }
            ]
        },
        {
            "role": "tool",
            "tool_call_id": "call_transactions_001",
            "content": "[{\"transaction_id\": \"txn-000001\", \"customer_id\": \"customer-001\", \"region\": \"US\", \"amount\": 137.25, \"currency\": \"USD\", \"status\": \"failed\", \"provider\": \"adyen\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000002\", \"customer_id\": \"customer-002\", \"region\": \"GB\", \"amount\": 174.5, \"currency\": \"USD\", \"status\": \"pending\", \"provider\": \"checkout\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000003\", \"customer_id\": \"customer-003\", \"region\": \"DE\", \"amount\": 211.75, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"razorpay\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000004\", \"customer_id\": \"customer-004\", \"region\": \"SG\", \"amount\": 249.0, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000005\", \"customer_id\": \"customer-005\", \"region\": \"IN\", \"amount\": 286.25, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000006\", \"customer_id\": \"customer-006\", \"region\": \"US\", \"amount\": 323.5, \"currency\": \"USD\", \"status\": \"failed\", \"provider\": \"adyen\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000007\", \"customer_id\": \"customer-007\", \"region\": \"GB\", \"amount\": 360.75, \"currency\": \"USD\", \"status\": \"pending\", \"provider\": \"checkout\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000008\", \"customer_id\": \"customer-008\", \"region\": \"DE\", \"amount\": 398.0, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"razorpay\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000009\", \"customer_id\": \"customer-009\", \"region\": \"SG\", \"amount\": 435.25, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000010\", \"customer_id\": \"customer-010\", \"region\": \"IN\", \"amount\": 472.5, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000011\", \"customer_id\": \"customer-011\", \"region\": \"US\", \"amount\": 509.75, \"currency\": \"USD\", \"status\": \"failed\", \"provider\": \"adyen\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000012\", \"customer_id\": \"customer-012\", \"region\": \"GB\", \"amount\": 547.0, \"currency\": \"USD\", \"status\": \"pending\", \"provider\": \"checkout\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000013\", \"customer_id\": \"customer-013\", \"region\": \"DE\", \"amount\": 584.25, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"razorpay\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000014\", \"customer_id\": \"customer-014\", \"region\": \"SG\", \"amount\": 621.5, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000015\", \"customer_id\": \"customer-015\", \"region\": \"IN\", \"amount\": 658.75, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000016\", \"customer_id\": \"customer-016\", \"region\": \"US\", \"amount\": 696.0, \"currency\": \"USD\", \"status\": \"failed\", \"provider\": \"adyen\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000017\", \"customer_id\": \"customer-017\", \"region\": \"GB\", \"amount\": 733.25, \"currency\": \"USD\", \"status\": \"pending\", \"provider\": \"checkout\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000018\", \"customer_id\": \"customer-018\", \"region\": \"DE\", \"amount\": 770.5, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"razorpay\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000019\", \"customer_id\": \"customer-019\", \"region\": \"SG\", \"amount\": 807.75, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000020\", \"customer_id\": \"customer-020\", \"region\": \"IN\", \"amount\": 845.0, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000021\", \"customer_id\": \"customer-021\", \"region\": \"US\", \"amount\": 882.25, \"currency\": \"USD\", \"status\": \"failed\", \"provider\": \"adyen\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000022\", \"customer_id\": \"customer-022\", \"region\": \"GB\", \"amount\": 919.5, \"currency\": \"USD\", \"status\": \"pending\", \"provider\": \"checkout\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000023\", \"customer_id\": \"customer-023\", \"region\": \"DE\", \"amount\": 956.75, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"razorpay\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000024\", \"customer_id\": \"customer-024\", \"region\": \"SG\", \"amount\": 994.0, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}, {\"transaction_id\": \"txn-000025\", \"customer_id\": \"customer-025\", \"region\": \"IN\", \"amount\": 1031.25, \"currency\": \"USD\", \"status\": \"completed\", \"provider\": \"stripe\", \"created_at\": \"2026-07-01T10:10:00Z\", \"metadata\": {\"product\": \"enterprise-plan\", \"source\": \"web\", \"description\": \"Synthetic transaction generated for testing large tool-result payloads and context-window handling.\"}}]"
        },
        {
            "role": "user",
            "content": "Return a concise summary."
        }
    ],
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "get_transactions",
                "description": "Fetches transaction records for a date range.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "start_date": {
                            "type": "string"
                        },
                        "end_date": {
                            "type": "string"
                        },
                        "limit": {
                            "type": "integer"
                        }
                    },
                    "required": [
                        "start_date",
                        "end_date"
                    ]
                }
            }
        }
    ],
    "optimize": {
        "compress": {
            "engine": "headroom"
        }
    }
}
```

**What it does:** collapses the duplicated rule block and normalizes the repetitive structure. `compress_system_messages` and `compress_user_messages` are both on, and `protect_recent: 0` means no trailing turns are exempted. Because `headroom` is lossless, every RULE/OUTPUT line and the full tool schema survive.

#### `caveman` — rule-based prose

**Best for** verbose, human-written instructions. `caveman` strips filler ("I would like you to please take the time to…"), redundant phrasing, and padding, while guaranteeing that negations, numbers, codes, URLs, paths, and quoted strings are kept. Here it's scoped to just the user message.

```bash
{
  "model": "openai/gpt-5.4",
  "messages": [
    {
      "role": "user",
      "content": "I would like you to please take the time to carefully review and analyze all of the information contained in this message. It is very important that you provide a response that is clear, readable, concise, well structured, and easy to understand. Please avoid unnecessary filler words, redundant phrases, repeated explanations, overly long introductory statements, and verbose transitions. However, you must preserve every important fact and every constraint. The final answer must contain exactly 3 bullet points. It must not exceed 120 words. You must not mention internal reasoning. Preserve the code FR-2048, the number 17.5, the URL https://example.com/docs, the file path /srv/app/config.json, and the quoted phrase \"do not delete\". Explain how to safely deploy a software configuration update."
    }
  ],
  "stream": false,
  "optimize": {
    "compress": {
      "engine": "caveman",
      "caveman_level": "semantic",
      "caveman_roles": ["user"]
    }
  }
}
```

**What it does:** at `caveman_level: "semantic"` it removes conversational filler and redundancy but keeps the hard constraints intact — `FR-2048`, `17.5`, `https://example.com/docs`, `/srv/app/config.json`, `"do not delete"`, the "exactly 3 bullet points" and "120 words" limits, and the negations. `caveman_roles: ["user"]` leaves any system message untouched.

#### `llmlingua` — ML token pruning

**Best for** long, information-dense prose: RAG chunks, transcripts, and lengthy support context. A small model scores tokens and drops the lowest-information ones. Here both a long system prompt and a long user report are pruned.

```bash
{
  "model": "anthropic/claude-sonnet-5",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "system",
      "content": "You are a technical support assistant for Meridian Cache, a distributed in-memory caching layer. Establish the user's version first, because behavior differs across major versions. 3.x uses gossip membership (500ms heartbeat, 3 missed = failure); 4.x uses a Raft coordinator (150ms heartbeat, odd number of coordinator nodes required, 3 or 5 recommended). … (long system prose — version-specific rules, eviction policies, limits, and triage order)"
    },
    {
      "role": "user",
      "content": "Hoping you can help me untangle something. We run a six-node Meridian Cache cluster, trouble-free for ~14 months. Ten days ago we started seeing intermittent p99 latency spikes (p50 unchanged at ~1.2ms, p99 up from ~8ms to 180–400ms). Two weeks ago we shipped a feature that ~3x'd write volume and added a namespace with ~180-char keys; total keys went from ~4M to ~31M and climbing. … (long user prose describing what changed and what was already tried)"
    }
  ],
  "optimize": {
    "compress": {
      "engine": "llmlingua",
      "llmlingua_rate": 0.75
    }
  }
}
```

**What it does:** keeps roughly 75% of tokens (`llmlingua_rate: 0.75` = gentle), pruning low-signal words from both the system rules and the user's narrative while retaining the concrete facts (versions, thresholds, key counts, latency figures). Drop the rate toward `0.5` for more aggressive savings. Remember the first `llmlingua` call loads the model (\~70s); subsequent calls are fast.

> **Anthropic note:** if you send the system prompt as the top-level `system` field (Messages API) instead of a `system` message, it's compressed too — see [Anthropic Messages API notes](https://claude.ai/chat/d09d5af0-e351-4fc0-8e96-613fc1c7b41e#anthropic-messages-api-notes).

#### `all` — full pipeline

**Best for** mixed content where you want maximum savings and can tolerate lossy prose changes. Runs **headroom → llmlingua → caveman** in order.

```json
{
  "model": "openai/gpt-5.5",
  "messages": [
    { "role": "system", "content": "… (rules + schema)" },
    { "role": "user", "content": "… (long prose)" }
  ],
  "optimize": {
    "compress": {
      "engine": "all",
      "llmlingua_rate": 0.75
    }
  }
}
```

***

### Quick recipes

**Lossless (safe default)**

```bash
{ "optimize": { "compress": { "engine": "headroom" } } }
```

**Gentle ML compression, user messages only**

```bash
{
  "optimize": {
    "compress": {
      "engine": "llmlingua",
      "llmlingua_rate": 0.8,
      "llmlingua_roles": ["user"]
    }
  }
}
```

**Explicit engine list (order still headroom → llmlingua → caveman)**

```bash
{ "optimize": { "compress": { "engine": ["headroom", "llmlingua"] } } }
```

**Audit only — measure savings, change nothing**

```bash
{ "optimize": { "compress": { "engine": "all", "mode": "audit" } } }
```

**Full request (OpenAI format)**

```bash
curl -L https://api.fastrouter.ai/v1/chat/completions \
  -H 'Authorization: Bearer <api-key>' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "openai/gpt-5.5",
    "messages": [{"role": "user", "content": "<long prose>"}],
    "optimize": {"compress": {"engine": "llmlingua", "llmlingua_rate": 0.75}}
  }'
```

Check the `X-FastRouter-Compression-*` response headers to confirm it ran.

#### Reading the results

The response body is the normal provider response. Compression status is reported via headers:

| Header                         | Example | Meaning                        |
| ------------------------------ | ------- | ------------------------------ |
| `X-FastRouter-Compressed`      | `true`  | Compression was applied.       |
| `X-FastRouter-Tokens-Saved`    | `328`   | Tokens saved (before − after). |
| `X-FastRouter-Savings-Percent` | `26.23` | Percent saved.                 |

Headers are absent when compression did not apply (disabled, not opted-in, or failed open).

#### Anthropic Messages API notes

The same block works on `/v1/messages`, with a few specifics:

* The top-level `system` prompt is compressed too — often the largest prose block.
* Non-text blocks (`images`, `tool_use`, `tool_result`) pass through untouched.
* Content with a `cache_control` marker is always skipped, so prompt caching is never invalidated.
* `caveman` and `llmlingua` are structure-preserving and recommended here.

```bash
curl -L https://api.fastrouter.ai/v1/messages \
  -H 'Authorization: Bearer <api-key>' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "anthropic/claude-sonnet-5",
    "max_tokens": 1024,
    "system": "You are a helpful assistant. <long system prose>",
    "messages": [{"role": "user", "content": "<long prose>"}],
    "optimize": {"compress": {"engine": "llmlingua", "llmlingua_rate": 0.75}}
  }'
```

#### Fail-open behavior

Compression is silently skipped — originals kept, request proceeds normally — when:

* The request has no `optimize.compress` block.
* The compression service is unreachable, times out, or returns an error.
* The response can't be decoded or doesn't match the input message count.

A request can never be broken by compression.
