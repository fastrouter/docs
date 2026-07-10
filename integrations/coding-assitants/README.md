---
description: >-
  Use FastRouter as an OpenAI-compatible provider inside your coding
  assistant.   One key, 100+ models, full observability and cost control.
icon: code
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
---

# Coding Assitants

Every coding assistant below speaks the OpenAI-compatible API, so pointing it at FastRouter takes two fields: a base URL and a key. Once traffic routes through FastRouter you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, and Mistral behind one endpoint, including coding models like [Grok Code Fast 1](https://fastrouter.ai/models/x-ai/grok-code-fast-1). Swap models by changing one slug, no re-auth.
* **Observability** on every request: cost, tokens, latency, and which model ran, visible in real time in your [dashboard](https://dashboard.fastrouter.ai/).
* **Reliability** through automatic failover across providers, response caching, and intelligent routing.
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation.

The setup is the similar everywhere: paste your FastRouter key, set the base URL, enter a model slug in `provider/model-name` format. Pick your tool below for the exact steps.

<table data-view="cards"><thead><tr><th data-type="content-ref"></th></tr></thead><tbody><tr><td><a href="codex-cli.md">codex-cli.md</a></td></tr><tr><td><a href="claude-code.md">claude-code.md</a></td></tr><tr><td><a href="deepseek-reasonix-cli-1.md">deepseek-reasonix-cli-1.md</a></td></tr><tr><td><a href="cursor.md">cursor.md</a></td></tr><tr><td><a href="cline.md">cline.md</a></td></tr><tr><td><a href="roo-code.md">roo-code.md</a></td></tr><tr><td><a href="opencode.md">opencode.md</a></td></tr><tr><td><a href="kilo-code.md">kilo-code.md</a></td></tr><tr><td><a href="xcode.md">xcode.md</a></td></tr><tr><td><a href="aider.md">aider.md</a></td></tr></tbody></table>
