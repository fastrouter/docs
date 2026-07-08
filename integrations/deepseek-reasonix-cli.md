---
description: Integrating DeepSeek Reasonix CLI with FastRouter.
icon: square-code
---

# DeepSeek Reasonix CLI

### What is DeepSeek Reasonix CLI

DeepSeek Reasonix CLI is an open-source DeepSeek-native AI coding agent that runs in your terminal. It supports any DeepSeek-compatible model provider, including FastRouter, allowing you to leverage FastRouter's unified API gateway, provider failover, unified observability, BYOK (Bring Your Own Key) support, and organizational controls while using Reasonix's agentic coding workflows.

### Install DeepSeek Reasonix CLI

Start by installing Reasonix on your local machine. Follow the [installation instructions](https://github.com/esengine/DeepSeek-Reasonix) for your operating system.

**Install command**

```bash
npm install -g reasonix
```

### Create a FastRouter API key

Sign up or log in to your FastRouter account to access models through FastRouter's unified AI gateway.

1. Open the [FastRouter dashboard](https://dashboard.fastrouter.ai/).
2. Navigate to **API Keys** (keys page).
3. Create a new API key.
4. Copy and securely store your API key.

### Configure DeepSeek Reasonix CLI to use FastRouter

Edit the `~/.reasonix/config.json` file to connect to FastRouter:

```json
{
  "apiKey": "sk-v1-***",
  "baseUrl": "https://api.fastrouter.ai/api/v1",
  "model": "deepseek/deepseek-v3.2"
}
```

**Start DeepSeek Reasonix CLI**

```bash
reasonix code
```

### Alternative: install Reasonix CLI with FastRouter

#### One-line install script

In the commands below, replace `$API_KEY` with your actual FastRouter API key.

```bash
curl -fsSL https://fastrouter.ai/reasonix/install-fastrouter.sh | sh -s -- $API_KEY
```

Or download, inspect, then run:

```bash
curl -fsSL https://fastrouter.ai/reasonix/install-fastrouter.sh -o install-fastrouter.sh
less install-fastrouter.sh
chmod +x install-fastrouter.sh
./install-fastrouter.sh $API_KEY
```

#### Install Reasonix with the FastRouter skill file

Install using `skill.md`:

```bash
reasonix skills install https://fastrouter.ai/reasonix/skill.md
```

Or ask Reasonix to install FastRouter:

```
install fastrouter in reasonix with key <KEY>
```

### Why use DeepSeek Reasonix CLI with FastRouter

Configure DeepSeek Reasonix CLI to use your FastRouter API key and endpoint. Once configured, you can access models through FastRouter and take advantage of:

**Unified API** — Access models from multiple AI providers through a single, consistent API interface.

**Provider failover & routing** — Automatically route requests to available providers and fail over during outages or rate limits.

**Unified observability** — Monitor requests, latency, costs, and usage across all providers from a centralized dashboard.

**Troubleshooting** — Debug API requests, authentication, routing, and provider responses faster with FastRouter's audit logs.

**BYOK (Bring Your Own Key)** — Use your own provider API keys while maintaining centralized routing and management.

**Organization controls** — Manage organizations, projects, keys, and API usage with enterprise-grade governance controls.

### References

You can find the `skill.md` file attached below, or at [https://fastrouter.ai/reasonix/skill.md](https://fastrouter.ai/reasonix/skill.md).

***

````markdown
---
name: reasonix-cli-fastrouter
description: "Install and configure FastRouter (https://fastrouter.ai) as the model provider in DeepSeek Reasonix CLI. Use this skill whenever the user asks to add, set up, install, configure, or register fastrouter as a provider in reasonix, edit ~/.reasonix/config.json, point reasonix or deepseek-reasonix at FastRouter, or wants to use FastRouter models with DeepSeek Reasonix CLI. The skill takes exactly one input: the FastRouter API key."
version: 1.0.0
author: FastRouter
license: MIT
metadata:
  reasonix:
    tags:
      [
        fastrouter,
        reasonix,
        deepseek,
        provider,
        setup,
        configuration,
        custom-provider,
        openai-compatible,
      ]
    homepage: https://fastrouter.ai
    docs: https://docs.fastrouter.ai
---

# FastRouter Setup for DeepSeek Reasonix CLI

Configures **fastrouter** as the model provider in DeepSeek Reasonix CLI by writing `apiKey`, `baseUrl`, and `model` into `~/.reasonix/config.json`. After setup, FastRouter models are invoked when the user runs `reasonix code` from any project.

## Inputs

This skill accepts **exactly one input**: the FastRouter API key. Everything else is hardcoded and must not be changed:

| Field         | Value                              |
| ------------- | ---------------------------------- |
| Provider      | `fastrouter`                       |
| Base URL      | `https://api.fastrouter.ai/api/v1` |
| Default model | `deepseek/deepseek-v3.2`           |
| Config file   | `~/.reasonix/config.json`          |
| File mode     | `0600` (key is stored on disk)     |

## When to Use This Skill

Trigger this skill when the user asks any of:

- "install fastrouter for reasonix"
- "install fastrouter in reasonix with key <KEY>"
- "set up / configure / register fastrouter in reasonix"
- "add fastrouter as a provider in reasonix"
- "edit `~/.reasonix/config.json` for fastrouter"
- "use fastrouter with deepseek reasonix cli"
- "point reasonix at fastrouter"
- "configure deepseek reasonix with fastrouter"

Anything similar referencing **reasonix** (or "deepseek reasonix") plus install / setup / configure / add / register / provider plus **fastrouter** should trigger it.

## Extracting the API Key (single-argument rule)

Treat every invocation as a single-argument call: `setup(api_key)`. Find the key in the user's message using these rules, in order:

1. If the message contains `key=<value>`, `--key <value>`, `--api-key <value>`, or `apiKey=<value>`, the value is the key.
2. Else, the **last whitespace-separated token in the message that looks like an API key** is the key. A token "looks like an API key" if it matches `^sk-v1-[A-Za-z0-9_\-]+$` (FastRouter keys always have the prefix `sk-v1-` followed by a random string of letters/digits/dashes/underscores).
3. Else, ask the user **once**: _"Please paste your FastRouter API key (just the key, nothing else)."_ Then treat their next message as the key verbatim, after stripping surrounding whitespace and quotes.

Examples of correct extraction:

| User message                                            | Extracted key        |
| ------------------------------------------------------- | -------------------- |
| `install fastrouter for reasonix with key sk-v1-abc123` | `sk-v1-abc123`       |
| `setup reasonix with fastrouter sk-v1-abc123`           | `sk-v1-abc123`       |
| `configure reasonix --api-key sk-v1-XYZ789`             | `sk-v1-XYZ789`       |
| `add fastrouter to reasonix key=sk-v1-9aBc_def-456`     | `sk-v1-9aBc_def-456` |
| `set up reasonix with fastrouter`                       | (prompt the user)    |

Once extracted, **never modify, paraphrase, or transform the key**. Do not strip characters you think are formatting. Do not lowercase. Do not echo the full key back to the user — only ever show it masked (`****` + last 4 chars) in confirmations.

## Redaction Check (do this before writing)

Some agent harnesses (Cursor, IDE extensions, CI loggers) may redact secrets in messages before the model sees them. After extracting the key, validate it. The key is **invalid for write** if any of:

- Equals `[REDACTED]`, `<redacted>`, `***`, `***REDACTED***`, or any string containing the substring `REDACTED` (case-insensitive)
- Does not start with the literal prefix `sk-v1-` (all real FastRouter keys begin with `sk-v1-`; anything else is either redacted, mistyped, or from a different provider)
- Has nothing after the `sk-v1-` prefix (i.e. the random portion is empty)
- Contains only `*` or `•` characters
- Is empty / whitespace-only

If the key fails validation due to redaction, **stop and tell the user**:

> "It looks like your environment is masking the API key before I can read it. To install fastrouter for Reasonix, please run these in your terminal directly so the key never passes through the model:
>
> ```bash
> mkdir -p ~/.reasonix
> cat > ~/.reasonix/config.json <<'EOF'
> {
>   "apiKey": "<YOUR_KEY>",
>   "baseUrl": "https://api.fastrouter.ai/api/v1",
>   "model": "deepseek/deepseek-v3.2"
> }
> EOF
> chmod 600 ~/.reasonix/config.json
> ```
>
> If `~/.reasonix/config.json` already exists with other settings, edit it manually and merge the three keys above instead of overwriting."

Do **not** proceed with a redacted-looking value — writing `[REDACTED]` as `apiKey` silently breaks the provider with 401 errors that look unrelated to redaction.

## Procedure

Once you have a valid key, follow these steps in order.

### Step 1 — Ensure Reasonix CLI is installed

Run via the `terminal` tool:

```bash
reasonix --version
```

If `reasonix` is not found, install it first:

```bash
npm install -g reasonix
reasonix --version
```

Reference: https://github.com/esengine/DeepSeek-Reasonix

### Step 2 — Ensure `~/.reasonix/config.json` exists

```bash
mkdir -p ~/.reasonix
[ -f ~/.reasonix/config.json ] || echo '{}' > ~/.reasonix/config.json
```

If the user prefers an interactive setup, they can instead run `reasonix code` once and paste the API key when prompted — Reasonix's built-in wizard generates the file automatically. Either path produces the same end state, but only proceed past this step once `~/.reasonix/config.json` exists.

### Step 3 — Write the FastRouter config

Reasonix config is a flat JSON object. Merge these three keys, **preserving any other existing keys** in the file. Use the extracted key exactly as-is — never rename, lowercase, or transform it.

Required keys:

```json
{
  "apiKey": "<USER_KEY>",
  "baseUrl": "https://api.fastrouter.ai/api/v1",
  "model": "deepseek/deepseek-v3.2"
}
```

Safe merge with `jq` (preferred — preserves other keys):

```bash
KEY="<USER_KEY>"
jq --arg key "$KEY" \
   '. + {apiKey: $key, baseUrl: "https://api.fastrouter.ai/api/v1", model: "deepseek/deepseek-v3.2"}' \
   ~/.reasonix/config.json > ~/.reasonix/config.json.tmp \
   && mv ~/.reasonix/config.json.tmp ~/.reasonix/config.json
```

Fallback merge with Python (when `jq` is unavailable):

```bash
python3 - "<USER_KEY>" <<'PY'
import json, os, sys
key = sys.argv[1]
path = os.path.expanduser("~/.reasonix/config.json")
try:
    with open(path) as f: cfg = json.load(f)
except Exception:
    cfg = {}
cfg["apiKey"]  = key
cfg["baseUrl"] = "https://api.fastrouter.ai/api/v1"
cfg["model"]   = "deepseek/deepseek-v3.2"
with open(path, "w") as f: json.dump(cfg, f, indent=2)
PY
```

### Step 4 — Restrict file permissions

The API key is stored on disk in cleartext. Make the file owner-only:

```bash
chmod 600 ~/.reasonix/config.json
```

### Step 5 — Verify

```bash
ls -l ~/.reasonix/config.json
jq 'del(.apiKey)' ~/.reasonix/config.json
jq -r '.apiKey | "apiKey ends in: ****" + .[-4:]' ~/.reasonix/config.json
```

Expected:

- File mode is `-rw-------` (octal `0600`)
- `baseUrl` is `"https://api.fastrouter.ai/api/v1"` exactly
- `model` is `"deepseek/deepseek-v3.2"` (or the user-specified slug)
- Masked key suffix matches the last 4 chars the user provided

**Never `cat` or otherwise dump the full file** — the `apiKey` is in cleartext. Always use the `jq 'del(.apiKey)'` form when showing config.

### Step 6 — Report success

Reply to the user with usage examples. Show the API key **only masked**.

> ✓ FastRouter is configured for DeepSeek Reasonix CLI (`****<last 4 chars>`).
>
> Start Reasonix from any project:
>
> ```bash
> cd /path/to/your/project
> reasonix code
> ```
>
> Switch models by editing `model` in `~/.reasonix/config.json`. Browse available DeepSeek slugs at https://fastrouter.ai/models.

### Step 7 — Optional model wiring

If (and only if) the user named a specific model in their request, set it as `model` instead of the default `deepseek/deepseek-v3.2`:

```bash
jq --arg m "<that_model>" '.model = $m' \
   ~/.reasonix/config.json > ~/.reasonix/config.json.tmp \
   && mv ~/.reasonix/config.json.tmp ~/.reasonix/config.json
```

Otherwise leave `"model": "deepseek/deepseek-v3.2"` untouched.

## Pitfalls

- **The API key IS stored in the config file.** Reasonix reads `apiKey` directly from `~/.reasonix/config.json` in cleartext (no env-var indirection). Never commit this file to git or share it. Always `chmod 600` after writing. Add `.reasonix/` and `**/.reasonix/config.json` to global `~/.gitignore` if the user version-controls dotfiles.
- **Don't `cat ~/.reasonix/config.json`** when output may be streamed to logs, chat, or terminal recordings — the key is in cleartext. Use `jq 'del(.apiKey)'` for inspection.
- **Use merge, not overwrite.** A naive `echo '{...}' > ~/.reasonix/config.json` will wipe any other Reasonix settings the user has. Always merge via `jq` or the Python fallback.
- **Restart `reasonix` after changing the config.** An active session has the old config in memory; new values only apply to the next run.
- **Don't run `reasonix code` mid-config-write.** The interactive wizard may overwrite fields. Finish the procedure first, then launch.
- **Model slug must exist on FastRouter.** Use slugs from https://fastrouter.ai/models (DeepSeek-compatible models include `deepseek/deepseek-v3.2`, `deepseek/deepseek-r1`, etc.). A bad slug returns 404 from FastRouter, surfacing in Reasonix as a "model not found" error.
- **Redacted keys silently corrupt config.** Always run the redaction check before writing. Writing `[REDACTED]` (or pasting a masked key) as `apiKey` produces 401 errors later that look unrelated to redaction.
- **Never echo the full key back.** Only show it masked (last 4 chars). Some platforms log assistant output verbatim.

## Verification Checklist

Before reporting success, confirm:

- [ ] Exactly one key was extracted (Inputs rule)
- [ ] The key passed the redaction check (starts with `sk-v1-`, no `REDACTED` substring)
- [ ] `~/.reasonix/config.json` exists and contains all three required keys (`apiKey`, `baseUrl`, `model`)
- [ ] `baseUrl` equals `https://api.fastrouter.ai/api/v1` exactly
- [ ] File mode is `0600` (owner read/write only)
- [ ] Other pre-existing keys in the file were preserved (merge, not overwrite)
- [ ] You did NOT echo the raw API key — only masked form

## References

- DeepSeek Reasonix: https://github.com/esengine/DeepSeek-Reasonix
- FastRouter dashboard: https://dashboard.fastrouter.ai/
- FastRouter API base: `https://api.fastrouter.ai/api/v1`
- FastRouter models: https://fastrouter.ai/models
- FastRouter docs: https://docs.fastrouter.ai
````
