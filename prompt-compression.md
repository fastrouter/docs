---
description: >-
  Prompt Compression intelligently shrinks prompts before they're sent to an AI
  model, reducing token usage while maintaining response quality. It helps lower
  costs and maximize available context.
icon: compress
---

# Prompt Compression

### Overview

Opt-in compression for chat requests. Add one block to your request body and FastRouter compresses your messages before they reach the provider — cutting input tokens without changing the response.

* **Off by default** — nothing happens unless you opt in
* **Opt-in per request** — one block in the body
* **Fail-open** — if compression can't run, your original messages are sent unchanged

### How it works

```
Client ──► FastRouter gateway ──► compression ──► Provider (OpenAI / Anthropic / …)
```

You send a normal chat request plus a small `optimize` block. The gateway compresses eligible messages, forwards the compressed payload upstream, and returns the normal provider response with `X-FastRouter-Compression-*` headers reporting savings.

### Enabling it

Compression runs only when your request opts in — the body includes an `optimize.compress` block. Without it, the request flows through completely unchanged.

### Choosing an engine

Pick based on your content, not the implementation:

| Your content                               | Engine value           | What it does                                                                                | Type     |
| ------------------------------------------ | ---------------------- | ------------------------------------------------------------------------------------------- | -------- |
| Strict system prompts, rules, tool schemas | `"headroom"` (default) | Structural compression — restructures JSON/repetitive payloads without dropping information | Lossless |
| Chat prose, verbose instructions           | `"caveman"`            | Rule-based compaction — strips filler and redundancy, preserves all facts and constraints   | Lossy    |
| RAG chunks, docs, transcripts              | `"llmlingua"`          | ML token pruning — a small model scores and drops low-information tokens                    | Lossy    |
| Maximum savings on mixed content           | `"all"`                | Full pipeline: headroom → llmlingua → caveman                                               | Mixed    |

`engine` accepts a single value, a preset (`"both"`/`"hybrid"` = headroom → caveman, `"all"` = the full pipeline), or an explicit list like `["headroom", "llmlingua"]`. Application order is always **headroom → llmlingua → caveman**.

> **Try before you buy:** set `"mode": "audit"` to measure what you _would_ save without changing a single byte of your request. Stats are still returned in the response headers.

### Request format

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

### Supported routes

| Surface                 | Endpoints                                                                              | Coverage                             |
| ----------------------- | -------------------------------------------------------------------------------------- | ------------------------------------ |
| OpenAI chat completions | `POST /v1/chat/completions`, `POST /api/v1/chat/completions`, `POST /chat/completions` | Streaming, non-streaming & SDK paths |
| Anthropic Messages      | `POST /v1/messages`, `POST /api/v1/messages`                                           | Streaming & non-streaming            |

Native Responses / Gemini handlers are not covered yet.

### Parameters

#### General

| Param         | Type                | Default      | Description                                                                     |
| ------------- | ------------------- | ------------ | ------------------------------------------------------------------------------- |
| `engine`      | string \| string\[] | `"headroom"` | Which compressor(s) to run.                                                     |
| `mode`        | string              | `"optimize"` | `"optimize"` applies transforms; `"audit"` only observes (still returns stats). |
| `provider`    | string              | inferred     | Provider hint for token counting.                                               |
| `cache_align` | bool                | `false`      | Improve provider prompt-cache hits (does not reduce tokens).                    |

#### Lossless structural (`engine: "headroom"`)

| Param                           | Type  | Description                                |
| ------------------------------- | ----- | ------------------------------------------ |
| `target_ratio`                  | float | Desired compression ratio target.          |
| `min_tokens_to_compress`        | int   | Skip blocks smaller than this.             |
| `compress_user_messages`        | bool  | Compress user-role messages.               |
| `compress_system_messages`      | bool  | Compress system-role messages.             |
| `protect_recent` / `keep_turns` | int   | Leave the N most recent turns untouched.   |
| `protect_analysis_context`      | bool  | Protect analysis/reasoning context blocks. |

#### Rule-based prose (`engine: "caveman"`)

| Param           | Type      | Default   | Description                                 |
| --------------- | --------- | --------- | ------------------------------------------- |
| `caveman_level` | string    | `"light"` | `"light"`, `"semantic"`, or `"aggressive"`. |
| `caveman_roles` | string\[] | all roles | Restrict to given roles, e.g. `["user"]`.   |

Never drops negations, modals, quantifiers, code, URLs, paths, numbers, quoted strings, or ALL-CAPS codes.

#### ML prose (`engine: "llmlingua"`)

| Param                    | Type        | Default   | Description                                                       |
| ------------------------ | ----------- | --------- | ----------------------------------------------------------------- |
| `llmlingua_rate`         | float (0–1) | `0.75`    | Fraction of tokens to keep. Higher = gentler. `0.5` = aggressive. |
| `llmlingua_target_token` | int         | —         | Absolute token budget (overrides rate).                           |
| `llmlingua_roles`        | string\[]   | all roles | Restrict to given roles, e.g. `["user"]`.                         |

> The first `llmlingua` request loads the ML model (\~70s). Later requests are fast. `headroom` and `caveman` have no load cost.

### Reading the results

The response body is the normal provider response. Compression status is reported via headers:

| Header                         | Example | Meaning                        |
| ------------------------------ | ------- | ------------------------------ |
| `X-FastRouter-Compressed`      | `true`  | Compression was applied.       |
| `X-FastRouter-Tokens-Saved`    | `328`   | Tokens saved (before − after). |
| `X-FastRouter-Savings-Percent` | `26.23` | Percent saved.                 |

Headers are absent when compression did not apply (disabled, not opted-in, or failed open).

### Anthropic Messages API notes

The same block works on `/v1/messages`, with a few specifics:

* The top-level `system` prompt is compressed too — often the largest prose block.
* Non-text blocks (`images`, `tool_use`, `tool_result`) pass through untouched.
* Content with a `cache_control` marker is always skipped, so prompt caching is never invalidated.
* `caveman` and `llmlingua` are structure-preserving and recommended here.

```bash
curl -sS -i -X POST https://api.fastrouter.ai/v1/messages \
  -H 'x-api-key: <api-key>' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "anthropic/claude-sonnet-5",
    "max_tokens": 1024,
    "system": "You are a helpful assistant. <long system prose>",
    "messages": [{"role": "user", "content": "<long prose>"}],
    "optimize": {"compress": {"engine": "llmlingua", "llmlingua_rate": 0.75}}
  }'
```

### Examples

**Lossless (safe default)**

```json
{ "optimize": { "compress": { "engine": "headroom" } } }
```

**Gentle ML compression, user messages only**

```json
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

**Stack all engines**

```json
{ "optimize": { "compress": { "engine": "all", "llmlingua_rate": 0.75 } } }
```

**Audit only — measure savings, change nothing**

```json
{ "optimize": { "compress": { "engine": "all", "mode": "audit" } } }
```

**Full request (OpenAI format)**

```bash
curl -sS -i -X POST https://api.fastrouter.ai/v1/chat/completions \
  -H 'Authorization: Bearer <api-key>' \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "openai/gpt-5.5",
    "messages": [{"role": "user", "content": "<long prose>"}],
    "optimize": {"compress": {"engine": "llmlingua", "llmlingua_rate": 0.75}}
  }'
```

Check the `X-FastRouter-Compression-*` response headers to confirm it ran.

### Fail-open behavior

Compression is silently skipped — originals kept, request proceeds normally — when:

* The request has no `optimize.compress` block.
* The compression service is unreachable, times out, or returns an error.
* The response can't be decoded or doesn't match the input message count.

A request can never be broken by compression.
