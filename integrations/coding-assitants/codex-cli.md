---
description: Integrating Codex CLI with FastRouter.
icon: openai
---

# Codex CLI

### What is Codex CLI

Codex CLI is OpenAI's open-source local coding agent that runs in your terminal. It supports multiple model providers, including FastRouter, allowing you to leverage FastRouter's unified API gateway, provider failover, unified observability, BYOK (Bring Your Own Key) support, and organizational controls while using Codex's agentic coding workflows.

### 1. Install Codex CLI

Start by installing Codex CLI on your local machine. Follow the [installation instructions](https://github.com/openai/codex) for your operating system.

```bash
npm install -g @openai/codex
# provide fastrouter api key by following steps 2 and 3
codex --version
```

### 2. Create a FastRouter account

Sign up or log in to your FastRouter account to access models through FastRouter's unified AI gateway.

### 3. Generate a FastRouter API key

1. Open the [FastRouter dashboard](https://dashboard.fastrouter.ai/).
2. Navigate to **API Keys** (keys page).
3. Create a new API key.
4. Copy and securely store your API key.

### 4. Configure Codex CLI to use FastRouter

#### 4.1. Edit the config file

Edit the `~/.codex/config.toml` file to connect to FastRouter:

```toml
model_provider = "fastrouter"
model_reasoning_effort = "high"
model = "openai/gpt-5.5"

[model_providers.fastrouter]
name = "fastrouter"
base_url = "https://api.fastrouter.ai/api/v1"
env_key = "FASTROUTER_API_KEY"
wire_api = "responses"
```

#### 4.2. Export your FastRouter API key

Export the API key in your terminal:

```bash
export FASTROUTER_API_KEY="sk-v1-****"
```

#### 4.3. Start Codex

```bash
cd /path/to/your/project
codex
```

### 5. Codex core settings reference

Codex supports project-level trust settings. Define project paths to control which directories and resources the agent can access.

```toml
[projects."/path/to/trusted/project"]
trust_level = "trusted"

[projects."/path/to/untrusted/project"]
trust_level = "untrusted"
```

* **trusted** — Agent can freely access and modify the project, including running commands and editing files.
* **untrusted** — Agent operates with limited permissions and requires additional approval for sensitive actions.

### 6. Alternative: install Codex CLI with FastRouter

#### One-line install script

In the commands below, replace `$API_KEY` with your actual FastRouter API key.

```bash
curl -fsSL https://fastrouter.ai/codex/install-fastrouter.sh | sh -s -- $API_KEY
```

Or download, inspect, then run:

```bash
curl -fsSL https://fastrouter.ai/codex/install-fastrouter.sh -o install-fastrouter.sh
less install-fastrouter.sh
chmod +x install-fastrouter.sh
./install-fastrouter.sh $API_KEY
```

#### Install Codex with the FastRouter skill file

Install using `skill.md`:

```bash
codex skills install https://fastrouter.ai/codex/skill.md
```

Or ask Codex to install FastRouter:

```
install fastrouter in codex with key <API_KEY>
```

### 7. Why use Codex CLI with FastRouter

Configure Codex CLI to use your FastRouter API key and endpoint. Once configured, you can access models through FastRouter and take advantage of:

**Unified API** — Access models from multiple AI providers through a single, consistent API interface.

**Provider failover & routing** — Automatically route requests to available providers and fail over during outages or rate limits.

**Unified observability** — Monitor requests, latency, costs, and usage across all providers from a centralized dashboard.

**Troubleshooting** — Debug API requests, authentication, routing, and provider responses faster with FastRouter's audit logs.

**BYOK (Bring Your Own Key)** — Use your own provider API keys while maintaining centralized routing and management.

**Organization controls** — Manage organizations, projects, keys, and API usage with enterprise-grade governance controls.

### 8. References

You can find the `skill.md` file attached below, or at [https://fastrouter.ai/codex/skill.md](https://fastrouter.ai/codex/skill.md).

***

````markdown
---
name: codex-cli-fastrouter
description: "Install and configure FastRouter (https://fastrouter.ai) as a custom OpenAI-compatible model provider in OpenAI Codex CLI. Use this skill whenever the user asks to add, set up, install, configure, or register fastrouter as a provider in codex, edit ~/.codex/config.toml, export FASTROUTER_API_KEY, or wants to use FastRouter models with Codex CLI. The skill takes exactly one input: the FastRouter API key."
version: 1.0.0
author: FastRouter
license: MIT
metadata:
  codex:
    tags:
      [
        fastrouter,
        codex,
        provider,
        setup,
        configuration,
        custom-provider,
        openai-compatible,
      ]
    homepage: https://fastrouter.ai
    docs: https://docs.fastrouter.ai
---

# FastRouter Setup for Codex CLI

Configures **fastrouter** as a custom OpenAI-compatible model provider in OpenAI Codex CLI. After setup, FastRouter models are invoked by setting `model_provider = "fastrouter"` and `model = "<slug>"` in `~/.codex/config.toml`, with auth flowing through the `FASTROUTER_API_KEY` env var.

## Inputs

This skill accepts **exactly one input**: the FastRouter API key. Everything else is hardcoded and must not be changed:

| Field        | Value                              |
| ------------ | ---------------------------------- |
| Provider key | `fastrouter`                       |
| Base URL     | `https://api.fastrouter.ai/api/v1` |
| Wire API     | `responses` (OpenAI Responses API) |
| Env var      | `FASTROUTER_API_KEY`               |
| Config file  | `~/.codex/config.toml`             |

## When to Use This Skill

Trigger this skill when the user asks any of:

- "install fastrouter for codex"
- "install fastrouter in codex with key <KEY>"
- "set up / configure / register fastrouter in codex"
- "add fastrouter as a provider in codex"
- "edit `~/.codex/config.toml` for fastrouter"
- "use fastrouter with codex cli"
- "point codex at fastrouter"

Anything similar referencing **codex** plus install / setup / configure / add / register / provider plus **fastrouter** should trigger it.

## Extracting the API Key (single-argument rule)

Treat every invocation as a single-argument call: `setup(api_key)`. Find the key in the user's message using these rules, in order:

1. If the message contains `key=<value>`, `--key <value>`, `--api-key <value>`, or `apiKey=<value>`, the value is the key.
2. Else, the **last whitespace-separated token in the message that looks like an API key** is the key. A token "looks like an API key" if it matches `^sk-v1-[A-Za-z0-9_\-]+$` (FastRouter keys always have the prefix `sk-v1-` followed by a random string of letters/digits/dashes/underscores).
3. Else, ask the user **once**: _"Please paste your FastRouter API key (just the key, nothing else)."_ Then treat their next message as the key verbatim, after stripping surrounding whitespace and quotes.

Examples of correct extraction:

| User message                                         | Extracted key     |
| ---------------------------------------------------- | ----------------- |
| `install fastrouter for codex with key sk-v1-abc123` | `sk-v1-abc123`    |
| `setup codex with fastrouter sk-v1-abc123`           | `sk-v1-abc123`    |
| `configure codex --api-key sk-v1-XYZ789`             | `sk-v1-XYZ789`    |
| `add fastrouter to codex key=sk-v1-9aBc...`          | `sk-v1-9aBc...`   |
| `set up codex with fastrouter`                       | (prompt the user) |

Once extracted, **never modify, paraphrase, or transform the key**. Do not strip characters you think are formatting. Do not lowercase. Do not echo the full key back to the user — only ever show it masked (`****` + last 4 chars) in confirmations.

## Redaction Check (do this before writing)

Some agent harnesses (Cursor, IDE extensions, CI loggers) may redact secrets in messages before the model sees them. After extracting the key, validate it. The key is **invalid for write** if any of:

- Equals `[REDACTED]`, `<redacted>`, `***`, `***REDACTED***`, or any string containing the substring `REDACTED` (case-insensitive)
- Does not start with the literal prefix `sk-v1-` (all real FastRouter keys begin with `sk-v1-`; anything else is either redacted, mistyped, or from a different provider)
- Has nothing after the `sk-v1-` prefix (i.e. the random portion is empty)
- Contains only `*` or `•` characters
- Is empty / whitespace-only

If the key fails validation due to redaction, **stop and tell the user**:

> "It looks like your environment is masking the API key before I can read it. To install fastrouter for Codex CLI, please run these in your terminal directly so the key never passes through the model:
>
> ```bash
> mkdir -p ~/.codex
> cat >> ~/.codex/config.toml <<'EOF'
> model_provider = "fastrouter"
> model = "openai/gpt-5.5"
>
> [model_providers.fastrouter]
> name = "fastrouter"
> base_url = "https://api.fastrouter.ai/api/v1"
> env_key = "FASTROUTER_API_KEY"
> wire_api = "responses"
> EOF
>
> export FASTROUTER_API_KEY="<YOUR_KEY>"
> echo 'export FASTROUTER_API_KEY="<YOUR_KEY>"' >> ~/.zshrc   # or ~/.bashrc
> ```

Do **not** proceed with a redacted-looking value — writing `[REDACTED]` as the API key silently breaks the provider with 401 errors that look unrelated to redaction.

## Procedure

Once you have a valid key, follow these steps in order.

### Step 1 — Ensure Codex CLI is installed

Run via the `terminal` tool:

```bash
codex --version
```

If `codex` is not found, install it first:

```bash
npm install -g @openai/codex
codex --version
```

### Step 2 — Ensure `~/.codex/config.toml` exists

```bash
mkdir -p ~/.codex
touch ~/.codex/config.toml
```

### Step 3 — Write the provider config

Codex stores configuration in TOML. Merge the following into `~/.codex/config.toml`. **Preserve existing keys** outside this block; do not clobber the file. If a top-level key already exists, update its value rather than duplicating.

Top-level keys (overwrite if present):

```toml
model_provider = "fastrouter"
model_reasoning_effort = "high"
model = "openai/gpt-5.5"
```

Provider table (must exist exactly as shown):

```toml
[model_providers.fastrouter]
name = "fastrouter"
base_url = "https://api.fastrouter.ai/api/v1"
env_key = "FASTROUTER_API_KEY"
wire_api = "responses"
```

The API key is **never** written to the TOML file. It always flows through the `FASTROUTER_API_KEY` env var, controlled by the `env_key = "FASTROUTER_API_KEY"` line above.

### Step 4 — Export the API key

Run via the `terminal` tool, using the extracted key **exactly as-is**:

```bash
export FASTROUTER_API_KEY="<USER_KEY>"
```

For persistence across shells, append the same line to the user's shell rc file:

```bash
echo 'export FASTROUTER_API_KEY="<USER_KEY>"' >> ~/.zshrc   # or ~/.bashrc
```

If unsure which shell rc to use, run `echo $SHELL` and pick the matching file.

### Step 5 — Verify

```bash
grep -A 4 '\[model_providers.fastrouter\]' ~/.codex/config.toml
[ -n "$FASTROUTER_API_KEY" ] && echo "FASTROUTER_API_KEY is set" || echo "FASTROUTER_API_KEY is NOT set"
```

Expected: all four fields (`name`, `base_url`, `env_key`, `wire_api`) appear under `[model_providers.fastrouter]`, and the env var prints `set`. **Never echo the actual value** of `FASTROUTER_API_KEY`.

### Step 6 — Report success

Reply to the user with usage examples. Show the API key **only masked**.

> ✓ FastRouter is configured for Codex CLI (`****<last 4 chars>`).
>
> Start Codex from any project:
>
> ```bash
> cd /path/to/your/project
> codex
> ```
>
> Switch models by editing `model = "..."` in `~/.codex/config.toml`. Browse available slugs at https://fastrouter.ai/models.

### Step 7 — Optional model wiring

If (and only if) the user named a specific model in their request, set it as `model` instead of the default `openai/gpt-5.5`:

```toml
model = "<that_model>"
```

Otherwise leave `model = "openai/gpt-5.5"` and `model_reasoning_effort = "high"` untouched.

## Pitfalls

- **Don't write the API key into `~/.codex/config.toml`.** Codex reads keys only from env vars, controlled by `env_key = "FASTROUTER_API_KEY"`. Storing the key in the TOML does nothing useful and risks committing it to git if the user version-controls dotfiles.
- **`model_provider` must match the table key exactly.** `model_provider = "fastrouter"` requires `[model_providers.fastrouter]` (same string). A mismatch produces `provider 'fastrouter' not found`.
- **`wire_api` must be `"responses"`, not `"chat"`.** FastRouter speaks the OpenAI Responses API. Setting `wire_api = "chat"` causes wire-format errors mid-stream.
- **Env var must be exported in the same shell that runs `codex`.** If the user starts `codex` from a fresh terminal, the var is gone unless persisted in `~/.zshrc` / `~/.bashrc`.
- **Config changes require restarting `codex`.** If the user is in an active Codex session, the new provider only becomes available after quitting and restarting.
- **Redacted keys silently corrupt config.** Always run the redaction check before writing. Writing `[REDACTED]` (or pasting a masked key into `export`) produces 401 errors later that look unrelated to redaction.
- **Never echo the full key back.** Only show it masked (last 4 chars). Some platforms log assistant output verbatim.

## Verification Checklist

Before reporting success, confirm:

- [ ] Exactly one key was extracted (Inputs rule)
- [ ] The key passed the redaction check (starts with `sk-v1-`, no `REDACTED` substring)
- [ ] `~/.codex/config.toml` exists and contains all four required fields under `[model_providers.fastrouter]`
- [ ] Top-level `model_provider = "fastrouter"` is set
- [ ] `FASTROUTER_API_KEY` is exported in the current shell (and persisted in shell rc if the user requested)
- [ ] You did NOT echo the raw API key — only masked form

## References

- Codex CLI: https://github.com/openai/codex
- FastRouter dashboard: https://dashboard.fastrouter.ai/
- FastRouter API base: `https://api.fastrouter.ai/api/v1`
- FastRouter models: https://fastrouter.ai/models
- FastRouter docs: https://docs.fastrouter.ai
````
