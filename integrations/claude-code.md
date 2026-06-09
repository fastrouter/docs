---
description: >-
  This guide walks you through setting up Claude Code to work with FastRouter.ai
  as your API provider.
icon: claude
---

# Claude Code

#### Introduction

**Claude Code** is Anthropic's official CLI tool that brings Claude's AI capabilities directly to your terminal and development workflow. It enables you to interact with Claude for coding tasks, file operations, git workflows, and more—all from the command line.

**FastRouter.ai** is an intelligent AI routing platform that offers access to multiple model providers with a single API, offering cost reduction, improved reliability, and enhanced performance through smart routing algorithms.

By integrating **Claude Code** with **FastRouter.ai**, you gain a centralized and enterprise-ready way to operate Claude across your organization. FastRouter acts as the control plane for all usage—allowing you to securely manage and rotate user API keys, standardize access across teams, and eliminate the risks of scattered credentials.

In addition, FastRouter provides built-in observability and governance, giving you clear visibility into usage patterns, spend, and performance. You can enforce rate limits to prevent abuse, define role-based access controls to ensure the right people have the right level of access, and set budgets to keep costs predictable. Together, this integration lets you scale Claude Code safely, efficiently, and with full operational control—without slowing down developers.

#### Prerequisites

* Linux operating system
* Terminal access
* A FastRouter.ai API key ([Get one here](https://fastrouter.ai/))

#### Installation

**Step 1: Install Claude Code**

Run the following command in your terminal to install Claude Code on Linux:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

This will download and install the latest version of Claude Code on your system.

#### Configuration

**Step 2: Navigate to Your Project Directory**

Change to the directory where you want to use Claude Code:

```bash
cd /path/to/your/project
```

**Step 3: Set Environment Variables**

Run the following commands in your terminal to configure Claude Code to use FastRouter.ai:

```bash
export ANTHROPIC_BASE_URL="https://api.fastrouter.ai"
export ANTHROPIC_API_KEY=""
export ANTHROPIC_AUTH_TOKEN=[YOUR-FASTROUTER-API-KEY]
```

**Important Notes:**

* Replace `[YOUR-FASTROUTER-API-KEY]` with your actual FastRouter.ai API key
* Setting `ANTHROPIC_API_KEY=""` (empty string) is **required** for proper functionality
* These environment variables will only persist for your current terminal session

**Step 4: Persist Environment Variables (Optional)**

To make these settings permanent across terminal sessions, add them to your shell profile file.

**For Bash users** (`~/.bashrc` or `~/.bash_profile`):

```bash
echo 'export ANTHROPIC_BASE_URL="https://api.fastrouter.ai"' >> ~/.bashrc
echo 'export ANTHROPIC_API_KEY=""' >> ~/.bashrc
echo 'export ANTHROPIC_AUTH_TOKEN=[YOUR-FASTROUTER-API-KEY]' >> ~/.bashrc
source ~/.bashrc
```

**For Zsh users** (`~/.zshrc`):

```bash
echo 'export ANTHROPIC_BASE_URL="https://api.fastrouter.ai"' >> ~/.zshrc
echo 'export ANTHROPIC_API_KEY=""' >> ~/.zshrc
echo 'export ANTHROPIC_AUTH_TOKEN=[YOUR-FASTROUTER-API-KEY]' >> ~/.zshrc
source ~/.zshrc
```

**For Fish users** (`~/.config/fish/config.fish`):

```bash
echo 'set -x ANTHROPIC_BASE_URL "https://api.fastrouter.ai"' >> ~/.config/fish/config.fish
echo 'set -x ANTHROPIC_API_KEY ""' >> ~/.config/fish/config.fish
echo 'set -x ANTHROPIC_AUTH_TOKEN [YOUR-FASTROUTER-API-KEY]' >> ~/.config/fish/config.fish
source ~/.config/fish/config.fish
```

#### Verification

**Step 5: Launch Claude Code**

Start Claude Code by running:

```bash
claude
```

Claude Code should now be configured to route all API requests through FastRouter.ai.

***

#### Using Other Models via FastRouter

One of the key advantages of routing Claude Code through FastRouter is access to the **full range of models** supported by FastRouter—not just Anthropic's Claude. You can point Claude Code at any model available on FastRouter by passing the `--model` flag at startup.

**Syntax**

```bash
claude --model <provider-name>/<model-name>
```

**Examples**

**OpenAI GPT-5.4 Mini**

```bash
claude --model openai/gpt-5.4-mini
```

**DeepSeek V4 Pro**

```bash
claude --model deepseek/deepseek-v4-pro
```

**How It Works**

When you specify a model using the `--model` flag, FastRouter intercepts the request and routes it to the appropriate provider on your behalf. Your FastRouter API key handles authentication across all supported providers—no additional credentials needed.

> **Note:** Model availability depends on your FastRouter plan. Visit the [FastRouter model catalog](https://fastrouter.ai/) to see the full list of supported models and providers.

***

#### Troubleshooting

**Common Issues**

**Issue: Authentication errors**

* Verify your FastRouter.ai API key is correct
* Ensure `ANTHROPIC_API_KEY` is set to an empty string (`""`)
* Check that `ANTHROPIC_BASE_URL` is exactly `https://api.fastrouter.ai`

**Issue: Environment variables not persisting**

* Make sure you've added the exports to the correct shell profile file
* Run `source ~/.bashrc` (or appropriate file) to reload the configuration
* Verify the variables are set with: `echo $ANTHROPIC_BASE_URL`

**Issue: Claude Code not found after installation**

* Restart your terminal or open a new terminal window
* Check if the installation path is in your `PATH` variable
* Try running the installation command again

**Issue: Model not found or unsupported**

* Confirm the model name matches FastRouter's catalog exactly (e.g., `openai/gpt-5.4-mini`)
* Ensure your FastRouter plan includes access to the requested provider
* Check [FastRouter documentation](https://docs.fastrouter.ai/) for the current list of supported models

***
