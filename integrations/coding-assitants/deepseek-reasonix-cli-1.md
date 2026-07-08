---
description: >-
  Route DeepSeek Reasonix CLI through FastRouter for unified model access,
  observability, failover, and organizational controls.
icon: deepseek
---

# DeepSeek Reasonix CLI

### What is DeepSeek Reasonix CLI?

DeepSeek Reasonix CLI is an open-source, DeepSeek-native AI coding agent that runs in your terminal. It supports DeepSeek-compatible model providers, including FastRouter.

By routing DeepSeek Reasonix CLI through FastRouter, you get:

* **Unified API access** through a single, consistent endpoint
* **Provider failover and routing** for outages, rate limits, and provider availability
* **Unified observability** for requests, latency, costs, and usage
* **Bring Your Own Key (BYOK)** support for centralized provider-key management
* **Organization controls** for projects, keys, budgets, and usage governance

This guide covers installing DeepSeek Reasonix CLI, creating a FastRouter API key, configuring `~/.reasonix/config.json`, using the alternative installer, and installing the FastRouter Reasonix skill file. It does not cover general Reasonix usage outside FastRouter provider configuration.

#### Prerequisites

* Node.js and npm installed on your local machine
* A FastRouter account ([dashboard](https://dashboard.fastrouter.ai/))
* A FastRouter API key
* Terminal access to edit `~/.reasonix/config.json`

***

### Quick Start

#### Step 1: Install DeepSeek Reasonix CLI

Install Reasonix on your local machine. For platform-specific instructions, see the [DeepSeek Reasonix GitHub repository](https://github.com/esengine/DeepSeek-Reasonix).

```bash
npm install -g reasonix
```

#### Step 2: Create a FastRouter API Key

1. Sign up or log in to your FastRouter account.
2. Open the [FastRouter dashboard](https://dashboard.fastrouter.ai/).
3. Navigate to **API Keys**.
4. Create a new API key.
5. Copy and securely store your API key.

FastRouter API keys are only shown once. Store the key securely and do not commit it to source control.

#### Step 3: Configure DeepSeek Reasonix CLI to Use FastRouter

Edit `~/.reasonix/config.json` and add the FastRouter API key, base URL, and model.

```json
{
  "apiKey": "sk-v1-***",
  "baseUrl": "https://api.fastrouter.ai/api/v1",
  "model": "deepseek/deepseek-v3.2"
}
```

#### Step 4: Start DeepSeek Reasonix CLI

Start Reasonix from any project directory:

```bash
reasonix code
```

After configuration, Reasonix sends model requests through FastRouter.

***

### Alternative Install: Reasonix CLI with FastRouter

Use the FastRouter installer if you want a scripted setup. In the commands below, replace `$API_KEY` with your actual FastRouter API key.

#### One-Line Install Script

```bash
curl -fsSL https://fastrouter.ai/reasonix/install-fastrouter.sh | sh -s -- $API_KEY
```

#### Download, Inspect, Then Run

Use this option if you want to inspect the installer before running it.

```bash
curl -fsSL https://fastrouter.ai/reasonix/install-fastrouter.sh -o install-fastrouter.sh
```

```bash
less install-fastrouter.sh
```

```bash
chmod +x install-fastrouter.sh
```

```bash
./install-fastrouter.sh $API_KEY
```

***

### Install Reasonix with the FastRouter Skill File

Reasonix can install the FastRouter setup skill directly from the hosted skill file.

```bash
reasonix skills install https://fastrouter.ai/reasonix/skill.md
```

Then ask Reasonix to install FastRouter with your key:

```
install fastrouter in reasonix with key <KEY>
```

The skill file is available at https://fastrouter.ai/reasonix/skill.md.

***

### Use DeepSeek Reasonix CLI with FastRouter Models

Reasonix uses the `model` value in `~/.reasonix/config.json`. The default model in this guide is:

```
deepseek/deepseek-v3.2
```

To switch models, edit the `model` field in `~/.reasonix/config.json` and restart Reasonix. Use a valid FastRouter model slug from the [FastRouter models page](https://fastrouter.ai/models).

Example DeepSeek-compatible slugs include:

```
deepseek/deepseek-v3.2
deepseek/deepseek-r1
```

***

### Verify the Configuration

Check that Reasonix is installed:

```bash
reasonix --version
```

Check that `~/.reasonix/config.json` contains the FastRouter settings. Do not print the full file if it contains a real API key.

```bash
jq 'del(.apiKey)' ~/.reasonix/config.json
```

Expected values:

| Field     | Expected value                                   |
| --------- | ------------------------------------------------ |
| `baseUrl` | `https://api.fastrouter.ai/api/v1`               |
| `model`   | `deepseek/deepseek-v3.2`                         |
| `apiKey`  | A valid FastRouter key that starts with `sk-v1-` |

For security, restrict the config file to the file owner:

```bash
chmod 600 ~/.reasonix/config.json
```

***

### Why Use DeepSeek Reasonix CLI with FastRouter?

FastRouter centralizes access, routing, and governance for DeepSeek Reasonix CLI traffic.

#### Unified API

Access models from multiple AI providers through a single, consistent API interface.

#### Provider Failover & Routing

Automatically route requests to available providers and fail over during outages or rate limits.

#### Unified Observability

Monitor requests, latency, costs, and usage across all providers from a centralized dashboard.

#### Troubleshooting

Debug API requests, authentication, routing, and provider responses faster with FastRouter audit logs.

#### BYOK (Bring Your Own Key)

Use your own provider API keys while maintaining centralized routing and management.

#### Organization Controls

Manage organizations, projects, keys, and API usage with enterprise-grade governance controls.

***

### Troubleshooting

<table><thead><tr><th width="227.78125">Issue</th><th width="211.67578125">Cause</th><th width="300.625">Fix</th></tr></thead><tbody><tr><td><code>401</code> or authentication error</td><td>The API key is missing, invalid, or redacted.</td><td>Confirm <code>apiKey</code> starts with <code>sk-v1-</code> and replace masked values such as <code>***</code> or <code>[REDACTED]</code> with the real key.</td></tr><tr><td>Model not found</td><td>The <code>model</code> value is not a valid FastRouter slug.</td><td>Use a model slug from https://fastrouter.ai/models.</td></tr><tr><td>Reasonix still uses an old provider</td><td>An active Reasonix session loaded the previous config.</td><td>Restart <code>reasonix code</code> after editing <code>~/.reasonix/config.json</code>.</td></tr><tr><td>Config values disappeared</td><td>The config file was overwritten instead of merged.</td><td>Re-add the <code>apiKey</code>, <code>baseUrl</code>, and <code>model</code> keys while preserving other settings.</td></tr><tr><td>API key appears in logs</td><td>The full config file was printed or shared.</td><td>Rotate the API key in FastRouter, update <code>~/.reasonix/config.json</code>, and avoid using <code>cat ~/.reasonix/config.json</code>.</td></tr></tbody></table>

***

### Security Notes

* Reasonix stores `apiKey` in `~/.reasonix/config.json` in cleartext.
* Do not commit `~/.reasonix/config.json` to git.
* Use `chmod 600 ~/.reasonix/config.json` to restrict file access.
* Show masked API keys only. Do not echo the full key in logs, screenshots, or support messages.
* If you accidentally expose a key, rotate it in the FastRouter dashboard.

***

### FastRouter Reasonix Skill File Reference

You can find the `skill.md` file attached below, or at https://fastrouter.ai/reasonix/skill.md.

The skill configures **fastrouter** as the model provider in DeepSeek Reasonix CLI by writing `apiKey`, `baseUrl`, and `model` into `~/.reasonix/config.json`. After setup, FastRouter models are invoked when the user runs `reasonix code` from any project.

```yaml
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
```

#### Inputs

This skill accepts **exactly one input**: the FastRouter API key. Everything else is hardcoded and must not be changed:

| Field         | Value                              |
| ------------- | ---------------------------------- |
| Provider      | `fastrouter`                       |
| Base URL      | `https://api.fastrouter.ai/api/v1` |
| Default model | `deepseek/deepseek-v3.2`           |
| Config file   | `~/.reasonix/config.json`          |
| File mode     | `0600` (key is stored on disk)     |

#### When to Use This Skill

Trigger this skill when the user asks any of:

* "install fastrouter for reasonix"
* "install fastrouter in reasonix with key "
* "set up / configure / register fastrouter in reasonix"
* "add fastrouter as a provider in reasonix"
* "edit `~/.reasonix/config.json` for fastrouter"
* "use fastrouter with deepseek reasonix cli"
* "point reasonix at fastrouter"
* "configure deepseek reasonix with fastrouter"

Anything similar referencing **reasonix** (or "deepseek reasonix") plus install / setup / configure / add / register / provider plus **fastrouter** should trigger it.

#### Extracting the API Key (single-argument rule)

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

Once extracted, **never modify, paraphrase, or transform the key**. Do not strip characters you think are formatting. Do not lowercase. Do not echo the full key back to the user - only ever show it masked (`****` + last 4 chars) in confirmations.

#### Redaction Check (do this before writing)

Some agent harnesses (Cursor, IDE extensions, CI loggers) may redact secrets in messages before the model sees them. After extracting the key, validate it. The key is **invalid for write** if any of:

* Equals `[REDACTED]`, `<redacted>`, `***`, `***REDACTED***`, or any string containing the substring `REDACTED` (case-insensitive)
* Does not start with the literal prefix `sk-v1-` (all real FastRouter keys begin with `sk-v1-`; anything else is either redacted, mistyped, or from a different provider)
* Has nothing after the `sk-v1-` prefix (i.e. the random portion is empty)
* Contains only `*` or `•` characters
* Is empty / whitespace-only

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

Do **not** proceed with a redacted-looking value - writing `[REDACTED]` as `apiKey` silently breaks the provider with 401 errors that look unrelated to redaction.

#### Procedure

Once you have a valid key, follow these steps in order.

**Step 1: Ensure Reasonix CLI is installed**

Run via the `terminal` tool:

```bash
reasonix --version
```

If `reasonix` is not found, install it first:

```bash
npm install -g reasonix
reasonix --version
```

Reference: https://github.com/esengine/DeepSeek-Reasonix.

**Step 2: Ensure `~/.reasonix/config.json` exists**

```bash
mkdir -p ~/.reasonix
[ -f ~/.reasonix/config.json ] || echo '{}' > ~/.reasonix/config.json
```

If the user prefers an interactive setup, they can instead run `reasonix code` once and paste the API key when prompted — Reasonix's built-in wizard generates the file automatically. Either path produces the same end state, but only proceed past this step once `~/.reasonix/config.json` exists.

**Step 3: Write the FastRouter config**

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

**Step 4: Restrict file permissions**

The API key is stored on disk in cleartext. Make the file owner-only:

```bash
chmod 600 ~/.reasonix/config.json
```

**Step 5: Verify**

```bash
ls -l ~/.reasonix/config.json
jq 'del(.apiKey)' ~/.reasonix/config.json
jq -r '.apiKey | "apiKey ends in: ****" + .[-4:]' ~/.reasonix/config.json
```

Expected:

* File mode is `-rw-------` (octal `0600`)
* `baseUrl` is `"https://api.fastrouter.ai/api/v1"` exactly
* `model` is `"deepseek/deepseek-v3.2"` (or the user-specified slug)
* Masked key suffix matches the last 4 chars the user provided

**Never `cat` or otherwise dump the full file** — the `apiKey` is in cleartext. Always use the `jq 'del(.apiKey)'` form when showing config.

**Step 6: Report success**

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

**Step 7: Optional model wiring**

If (and only if) the user named a specific model in their request, set it as `model` instead of the default `deepseek/deepseek-v3.2`:

```bash
jq --arg m "<that_model>" '.model = $m' \
   ~/.reasonix/config.json > ~/.reasonix/config.json.tmp \
   && mv ~/.reasonix/config.json.tmp ~/.reasonix/config.json
```

Otherwise leave `"model": "deepseek/deepseek-v3.2"` untouched.

#### Pitfalls

* **The API key IS stored in the config file.** Reasonix reads `apiKey` directly from `~/.reasonix/config.json` in cleartext (no env-var indirection). Never commit this file to git or share it. Always `chmod 600` after writing. Add `.reasonix/` and `**/.reasonix/config.json` to global `~/.gitignore` if the user version-controls dotfiles.
* **Don't `cat ~/.reasonix/config.json`** when output may be streamed to logs, chat, or terminal recordings — the key is in cleartext. Use `jq 'del(.apiKey)'` for inspection.
* **Use merge, not overwrite.** A naive `echo '{...}' > ~/.reasonix/config.json` will wipe any other Reasonix settings the user has. Always merge via `jq` or the Python fallback.
* **Restart `reasonix` after changing the config.** An active session has the old config in memory; new values only apply to the next run.
* **Don't run `reasonix code` mid-config-write.** The interactive wizard may overwrite fields. Finish the procedure first, then launch.
* **Model slug must exist on FastRouter.** Use slugs from https://fastrouter.ai/models (DeepSeek-compatible models include `deepseek/deepseek-v3.2`, `deepseek/deepseek-r1`, etc.). A bad slug returns 404 from FastRouter, surfacing in Reasonix as a "model not found" error.
* **Redacted keys silently corrupt config.** Always run the redaction check before writing. Writing `[REDACTED]` (or pasting a masked key) as `apiKey` produces 401 errors later that look unrelated to redaction.
* **Never echo the full key back.** Only show it masked (last 4 chars). Some platforms log assistant output verbatim.

#### Verification Checklist

Before reporting success, confirm:

* [ ] Exactly one key was extracted (Inputs rule)
* [ ] The key passed the redaction check (starts with `sk-v1-`, no `REDACTED` substring)
* [ ] `~/.reasonix/config.json` exists and contains all three required keys (`apiKey`, `baseUrl`, `model`)
* [ ] `baseUrl` equals `https://api.fastrouter.ai/api/v1` exactly
* [ ] File mode is `0600` (owner read/write only)
* [ ] Other pre-existing keys in the file were preserved (merge, not overwrite)
* [ ] You did NOT echo the raw API key - only masked form

***

### References

* DeepSeek Reasonix: https://github.com/esengine/DeepSeek-Reasonix
* FastRouter dashboard: https://dashboard.fastrouter.ai/
* FastRouter API base: `https://api.fastrouter.ai/api/v1`
* FastRouter models: https://fastrouter.ai/models
* FastRouter docs: https://docs.fastrouter.ai
* FastRouter Reasonix skill file: https://fastrouter.ai/reasonix/skill.md
