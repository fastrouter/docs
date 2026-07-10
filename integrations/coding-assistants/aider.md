---
description: Track usage, control costs, and add guardrails to your Aider coding sessions
icon: square-code
---

# Aider

What is Aider?

[Aider](https://aider.chat/) is an AI pair programming tool that runs in your terminal. It edits code in your local git repository, makes commits, and works across multiple files—driven entirely by natural-language instructions. It connects to any OpenAI-compatible endpoint.

By routing Aider through FastRouter, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint—including coding models like [Grok Code Fast 1](https://fastrouter.ai/models/x-ai/grok-code-fast-1)
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation

This guide covers configuring Aider to use FastRouter through its OpenAI-compatible settings.

**Prerequisites**

* A FastRouter.ai account ([sign up](https://fastrouter.ai))
* Python 3.10 or higher, and a git repository to work in

***

#### Quick Start

**Step 1: Install Aider**

```bash
python -m pip install aider-install
aider-install
```

**Step 2: Get Your FastRouter API Key**

1. Sign up or log in at [fastrouter.ai](https://fastrouter.ai)
2. Navigate to your project's **Keys** page
3. Click **Create User Key**
4. Copy the key immediately. FastRouter does not display the key again after creation.

**Step 3: Point Aider at FastRouter**

Aider uses LiteLLM under the hood. Set FastRouter as the OpenAI-compatible endpoint with two environment variables. **Run these in your terminal—not at Aider's `>` prompt—before launching Aider:**

```bash
export OPENAI_API_BASE=https://api.fastrouter.ai/api/v1
export OPENAI_API_KEY=sk-add-your-key-here
echo $OPENAI_API_KEY   # confirm it prints before continuing
```

> **Note:** Environment variables only last for the current terminal session. For a setup that persists, see Persisting your credentials below.

**Step 4: Launch Aider with a FastRouter Model**

Prefix the FastRouter model slug with `openai/` so LiteLLM uses the OpenAI-compatible protocol:

```bash
cd /path/to/your/git/repo
aider --model openai/x-ai/grok-code-fast-1
```

> **Note:** The `openai/` prefix is **required**. It selects the OpenAI-compatible protocol; the rest (`x-ai/grok-code-fast-1`) is the FastRouter model slug. Without it, LiteLLM cannot determine the provider and fails with `LLM Provider NOT provided`. For an Anthropic model, use `--model openai/anthropic/claude-4.5-sonnet`.

If the folder isn't a git repository, Aider offers to create one—answer **Yes** (recommended, so it can track and undo changes), or run with `--no-git` to skip git entirely.

On first launch you'll see two harmless warnings: _"Unknown context window size and costs"_ and _"Did you mean …"_. Both occur because Aider's local model database doesn't recognize FastRouter slugs. They don't affect anything—silence them with `--no-show-model-warnings`.

Aider starts in your repo and is ready for instructions. All requests route through FastRouter, and every request, token count, and cost appears in your [FastRouter Dashboard](https://dashboard.fastrouter.ai/).

**Persisting your credentials**

Because `export` only lasts for the current terminal session, the recommended setup is a `.env` file in your project root, which Aider loads automatically:

```bash
cat > .env <<'EOF'
OPENAI_API_BASE=https://api.fastrouter.ai/api/v1
OPENAI_API_KEY=sk-add-your-key-here
EOF
echo ".env" >> .gitignore   # never commit your key
```

With `.env` in place, just run `aider --model openai/x-ai/grok-code-fast-1`—no exports needed. Alternatively, add the two `export` lines to your shell profile (`~/.zshrc` or `~/.bashrc`) to apply them everywhere.

> **Security:** If you use a `.env` file, make sure it is git-ignored (the command above does this). Aider auto-ignores `.aider*` but **not** `.env`.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

***

#### Use Aider with 100+ Models

FastRouter uses the `provider/model-name` format (after LiteLLM's `openai/` protocol prefix). Switch models with the `--model` flag:

```bash
# Anthropic Claude
aider --model openai/anthropic/claude-4.5-sonnet

# OpenAI
aider --model openai/openai/gpt-5.2
```

**Recommended models for Aider:**

* `x-ai/grok-code-fast-1` — fast and inexpensive; a good default for routine edits
* `anthropic/claude-4.5-sonnet` — highest quality for complex, multi-file changes
* `openai/gpt-5.2` — strong all-round alternative

A common Aider pattern is a strong "architect" model for planning plus a cheaper "editor" model for the mechanical edits. Both use the same FastRouter key:

```bash
aider --model openai/anthropic/claude-4.5-sonnet \
      --editor-model openai/x-ai/grok-code-fast-1
```

For large files, add `--edit-format diff` so Aider sends only the changed lines instead of rewriting whole files.

[Explore the full model catalog](https://fastrouter.ai/models)

**Automatic Model Selection**

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

```bash
aider --model openai/fastrouter/auto
```

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

***

#### FAQs

**Configuration & Setup**

**I get `litellm.AuthenticationError: ... OPENAI_API_KEY ... must be set`. What's wrong?**

The environment variables aren't set in the terminal you launched Aider from—usually because you opened a new terminal (variables don't persist across sessions) or typed the `export` lines at Aider's `>` prompt instead of in the shell. Run the two `export` commands in your terminal _before_ launching Aider, confirm with `echo $OPENAI_API_KEY`, or use the `.env` file so the key is always available.

**I get `litellm.BadRequestError: LLM Provider NOT provided`. What's wrong?**

You omitted the `openai/` protocol prefix. The model must be `openai/<fastrouter-slug>`—for example, `openai/x-ai/grok-code-fast-1`, not `x-ai/grok-code-fast-1`.

**Aider reports an unknown model or context-window warning. Is that a problem?**

No. Aider's local model database doesn't have metadata for FastRouter slugs, so it shows _"Unknown context window size and costs"_ and _"Did you mean …"_ and falls back to sane defaults. Everything works—suppress the warnings with `--no-show-model-warnings`.

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. Switch models with `--model` at any time—no key changes needed.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

**Costs & Budgeting**

**Aider can send large repo context. How do I control cost?**

Set a budget and rate limit on the key, and use Aider's `/tokens` command to monitor context size. The Dashboard breaks down costs by project, key, model, and tag.

**Privacy & Security**

**Is my code sent to FastRouter's servers?**

FastRouter acts as a pass-through gateway. Requests are routed to the model provider and responses are returned to your client. Content logging can be disabled per key for sensitive workloads. See the **Disable Content Logging** option in key settings.

***

#### Next Steps

* [Explore the full model catalog](https://fastrouter.ai/models)
* [Set up Fallback Models](https://docs.fastrouter.ai/fallback-models) for high availability
* [Configure Alerts](https://docs.fastrouter.ai/alerts) for spend and performance monitoring
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the Discord community](https://discord.gg/QfTgEtMyyU)

