---
icon: brain-circuit
---

# Running Hermes Agent with FastRouter

***

**Hermes Agent** is an open-source, terminal-based AI agent built by [Nous Research](https://nousresearch.com). It brings a powerful, agentic assistant directly into your command line — capable of browsing the web, writing and running code, managing files, calling APIs, and orchestrating complex multi-step tasks, all without leaving your terminal.

**FastRouter** routes each call to the best backend model behind one OpenAI-compatible API. One key, every major model, unified billing.

***

## Prerequisites

* Hermes Agent installed and on your `PATH`. Verify with `hermes --version`. If you don't have it: [install Hermes](https://hermes-agent.nousresearch.com/docs).
* A FastRouter API key. Get one at [fastrouter.ai](https://fastrouter.ai).

## Three ways to install

Pick the one that matches where you're starting from. Please make sure you have hermes installed as listed in the prerequisites. Then chose any one of the below paths.

| Path                              | Best for                                                                   |
| --------------------------------- | -------------------------------------------------------------------------- |
| **A. One-line script**            | Fresh install . Fastest. FastRouter API key is the argument to the script. |
| **B. Manual `hermes config set`** | Explicit, CI-friendly, scriptable.                                         |
| **C. Skill inside Hermes chat**   | You already use Hermes with another provider.                              |

### A. One-line install script

In the below commands, replace <mark style="color:green;">$API\_KEY</mark> with your actual fastrouter api key.

```zsh
curl -fsSL https://fastrouter.ai/hermes/install-fastrouter.sh | sh -s -- $API_KEY
```

Or download, inspect, then run

```bash
curl -fsSL https://fastrouter.ai/hermes/install-fastrouter.sh -o install-fastrouter.sh
less install-fastrouter.sh
chmod +x install-fastrouter.sh
./install-fastrouter.sh $API_KEY
```

The script registers the provider, sets `fastrouter/auto` as your default model, enables cost display, and verifies the result.

## B. Manual hermes config set

```shellscript
# Required
hermes config set providers.fastrouter.base_url https://api.fastrouter.ai/api/v1
hermes config set providers.fastrouter.api_key $API_KEY

# Optional: make fastrouter/auto your default
hermes config set model.provider custom:fastrouter
hermes config set model.default fastrouter/auto

# Optional: show per-message cost
hermes config set display.show_cost true
```

Verify:

```shellscript
hermes config | grep -A 3 fastrouter
```

You should see both `base_url` and `api_key` under `providers.fastrouter`.

### C. Install FastRouter via Hermes Skill inside Hermes chat

Drive the FastRouter setup from a Hermes chat session. Install the skill once, then ask the agent to wire it up — and have it fetch and summarize available models on demand.

***

### 1. Install the Skill ( one time)

Run this once in the terminal to register the FastRouter skill with Hermes:

```bash
hermes skills install https://fastrouter.ai/hermes/skill.md
```

***

While installing, when prompted to pick a category of the skill , enter `gateway` or `router` or any other custom category that you want to put this skill into.

### 2. Ask Hermes to Install FastRouter

In a terminal, start a Hermes chat session by typing

`hermes`

Next, replace <mark style="color:green;">$API\_KEY</mark> with your fastrouter api key and type the following in the hermes chat

```
install fastrouter with key $API_KEY
```

The agent will load the skill, register FastRouter under `providers.fastrouter`, and confirm the setup. You can also ask it to list available models, group them by provider, or recommend one for your use case.

> #### <mark style="color:orange;">**Heads up — secret redaction. Hermes has an optional setting (**</mark><mark style="color:orange;">**`security.redact_secrets`**</mark><mark style="color:orange;">**) that masks API keys before the model sees them. It's off by default, so most users can ignore this. If you have it enabled, your key might be stripped before reaching the model and the skill won't be able to write it to config.**</mark>

***

### 3. Switch the Active Session to FastRouter

Custom providers are picked up at session start, so reset the session first:

```bash
/reset
```

Once the session restarts, open the model picker:

```bash
/model
```

Select `fastrouter` or `custom:fastrouter` from the provider list, then choose the FastRouter model you want to use. The session is now routing through FastRouter.

***

### Troubleshooting

* **Skill not triggering?** Type `/reload-skills` in the Hermes session, or restart the CLI.
* **Config changes not taking effect?** Custom providers load on session start. Run `/reset` in chat or relaunch `hermes`.
* **Script reports "hermes not found"?** Install Hermes first: [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/docs).
* **Getting invalid key error ?** check if the api key was redacted by hermes.

***

#### Reference

You can find the `skill.md` file:

* Attached below, or
* At: [https://fastrouter.ai/hermes/skill.md](https://fastrouter.ai/hermes/skill.md)

````xml
---
name: fastrouter-setup
description: "Install and configure FastRouter (https://fastrouter.ai) as a custom OpenAI-compatible provider in Hermes Agent. Use this skill whenever the user asks to add, set up, install, configure, or register fastrouter as a provider, or wants to use fastrouter models with Hermes. The skill takes exactly one input: the FastRouter API key."
version: 1.2.0
author: FastRouter
license: MIT
metadata:
  hermes:
    tags: [fastrouter, provider, setup, configuration, custom-provider, openai-compatible]
    homepage: https://fastrouter.ai
    docs: https://docs.fastrouter.ai
---

# FastRouter Setup for Hermes

Configures **fastrouter** as a custom OpenAI-compatible provider in Hermes
Agent. After setup, models can be invoked via `custom:fastrouter`.

## Inputs

This skill accepts **exactly one input**: the FastRouter API key.

Everything else is hardcoded and must not be changed:

| Field      | Value                                  |
|------------|----------------------------------------|
| Provider   | `fastrouter`                          |
| Base URL   | `https://api.fastrouter.ai/api/v1`     |
| API mode   | `chat_completions` (OpenAI-compatible) |
| Reference  | `custom:fastrouter`                   |

## When to Use This Skill

Trigger this skill when the user asks any of:

- "install fastrouter"
- "install fastrouter with key <KEY>"
- "install fastrouter sk-..."
- "add fastrouter as a provider"
- "set up / configure / register fastrouter in hermes"
- "use fastrouter with hermes"

Anything similar referencing **fastrouter** plus install / setup /
configure / add / register / provider should trigger it.

## Extracting the API Key (single-argument rule)

Treat every invocation as a single-argument call: `setup(api_key)`.

Find the key in the user's message using these rules, in order:

1. If the message contains `key=<value>`, `--key <value>`,
   `--api-key <value>`, or `apiKey=<value>`, the value is the key.
2. Else, the **last whitespace-separated token in the message that looks
   like an API key** is the key. A token "looks like an API key" if it
   matches `^[A-Za-z0-9_\-]{8,}$` and begins with one of: `sk-`, `fr-`,
   `fastrouter-`, or is otherwise clearly a credential (long random
   string of letters/digits/dashes/underscores).
3. Else, ask the user **once**: *"Please paste your FastRouter API key
   (just the key, nothing else)."* Then treat their next message as the
   key verbatim, after stripping surrounding whitespace and quotes.

Examples of correct extraction:

| User message                                  | Extracted key      |
|-----------------------------------------------|--------------------|
| `install fastrouter with key sk-abc123`      | `sk-abc123`        |
| `install fastrouter sk-abc123`               | `sk-abc123`        |
| `setup fastrouter --api-key fr-XYZ789`       | `fr-XYZ789`        |
| `add fastrouter key=fr-live-9aBc...`         | `fr-live-9aBc...`  |
| `install fastrouter`                         | (prompt the user)  |

Once extracted, **never modify, paraphrase, or transform the key**. Do
not strip characters you think are formatting. Do not lowercase. Do not
echo the full key back to the user — only ever show it masked
(`****` + last 4 chars) in confirmations.

## Redaction Check (do this before writing)

Hermes has an optional secret-redaction feature
(`security.redact_secrets`). It is **off by default**, but if the user
turned it on, the API key may arrive masked.

After extracting the key, validate it. The key is **invalid for write**
if any of:

- Equals `[REDACTED]`, `<redacted>`, `***`, `***REDACTED***`, or any
  string containing the substring `REDACTED` (case-insensitive)
- Is shorter than 8 characters AND does not match a known short test
  pattern the user clearly typed on purpose (e.g. the user literally
  wrote `sk-abc123` — accept that as a deliberate test value)
- Contains only `*` or `•` characters
- Is empty / whitespace-only

If the key fails validation due to redaction, **stop and tell the user**:

> "It looks like Hermes' secret redaction is masking your API key before
> I can read it. To install fastrouter, please run these in your
> terminal directly so the key never passes through the model:
>
> ```
> hermes config set providers.fastrouter.base_url https://api.fastrouter.ai/api/v1
> hermes config set providers.fastrouter.api_key <YOUR_KEY>
> ```
>
> Or temporarily disable redaction with
> `hermes config set security.redact_secrets false`, restart Hermes,
> ask me again, then re-enable redaction afterward."

Do **not** proceed with a redacted-looking value — writing `[REDACTED]`
as the API key silently breaks the provider with auth errors that look
unrelated to redaction.

## Procedure

Once you have a valid key, follow these steps in order.

### Step 1 — Write the provider config

Run these two commands via the `terminal` tool. Use the extracted key
exactly as-is.

```
hermes config set providers.fastrouter.base_url https://api.fastrouter.ai/api/v1
hermes config set providers.fastrouter.api_key <USER_KEY>
```

Both must exit 0. If either fails, surface the error and stop.

### Step 2 — Verify

```
hermes config | grep -A 3 fastrouter
```

Expected output should include both `base_url` and `api_key` under
`providers.fastrouter`.

### Step 3 — Report success

Reply to the user with usage examples. Show the API key **only masked**.

> ✓ FastRouter is configured (`****<last 4 chars>`).
>
> Use it for one-off calls:
> ```
> hermes chat --provider custom:fastrouter -m <model>
> ```
>
> Or set it as your default:
> ```
> hermes config set model.provider custom:fastrouter
> hermes config set model.default <model>
> ```
>
> Browse models at https://fastrouter.ai/models.

### Step 4 — Optional default-model wiring

If (and only if) the user named a specific model in their request, also
run:

```
hermes config set model.provider custom:fastrouter
hermes config set model.default <that_model>
```

Otherwise leave `model.provider` and `model.default` untouched.

## Pitfalls

- **Don't store the key in `.env`.** FastRouter lives under the
  top-level `providers:` dict in `config.yaml`, not as an aliased env
  var. If the user prefers env vars, use `key_env: FASTROUTER_API_KEY`
  instead of `api_key`, and have them export the var.
- **Two `providers:` keys exist** in some configs — one at top level and
  one nested under `model_catalog`. The `hermes config set
  providers.fastrouter.*` form always targets the top-level one. Don't
  hand-edit the wrong section.
- **Provider name is `custom:fastrouter`**, not just `fastrouter`.
  All custom providers are namespaced under `custom:` when referenced
  from the CLI, model picker, or `--provider` flag.
- **Config changes require a fresh session.** If the user is in an
  active Hermes chat when you run this skill, the new provider only
  becomes available after `/reset` or restart.
- **Redacted keys silently corrupt config.** Always run the redaction
  check before writing. Writing `[REDACTED]` as the API key produces
  auth errors later that look unrelated to redaction.
- **Never echo the full key back.** Only show it masked (last 4 chars).
  Some platforms log assistant output verbatim.

## Verification Checklist

Before reporting success, confirm:

- [ ] Exactly one key was extracted (Inputs rule)
- [ ] The key passed the redaction check
- [ ] Both `hermes config set` commands exited 0
- [ ] `hermes config` shows the `fastrouter` block with both fields
- [ ] You did NOT echo the raw API key — only masked form
````

_Need help?_ [_FastRouter docs_](https://docs.fastrouter.ai) _·_ [_Hermes docs_](https://hermes-agent.nousresearch.com/docs)
