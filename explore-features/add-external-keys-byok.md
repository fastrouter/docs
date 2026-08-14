---
description: >-
  Bring Your Own Key — Connect your own provider credentials to FastRouter and
  route traffic through your own accounts while retaining FastRouter's full
  routing, observability, and governance layer.
icon: key
---

# Add External Keys (BYOK)

### Overview

FastRouter's External Keys feature lets organization owners attach API credentials from any supported LLM provider directly to their organization. Traffic routes through your own provider account, preserving your negotiated pricing, rate limits, and compliance posture.

Each integration is a named, project-scoped record pairing a provider with credentials and a model selection. Multiple integrations can coexist for the same provider — for example, separate Anthropic integrations for production and research, each scoped to different projects.

***

### Prerequisites

* You must be an **Organization Owner** to create, edit, or delete integrations.
* Project Admins and Developers can use integrations within their project scope but cannot modify credentials.

***

### Supported providers

| Provider         | Credential type | Notes                                        |
| ---------------- | --------------- | -------------------------------------------- |
| Anthropic Claude | API Key         |                                              |
| Baseten          | API Key         |                                              |
| DeepInfra        | API Key         |                                              |
| FAL AI           | API Key         |                                              |
| Fireworks AI     | API Key         |                                              |
| Google AI Studio | API Key         |                                              |
| Google Vertex AI | Service Account | Service Account JSON required                |
| Groq             | API Key         |                                              |
| Microsoft Azure  | API Key         | Resource Name and deployment config required |
| Minimax          | API Key         |                                              |
| Moonshot         | API Key         |                                              |
| Nebius           | API Key         |                                              |
| OpenAI           | API Key         |                                              |
| Perplexity AI    | API Key         |                                              |
| Pollo AI         | API Key         |                                              |
| Together AI      | API Key         |                                              |
| X-AI             | API Key         |                                              |

***

### Creating an integration

Navigate to **Setup → External Keys** and click **New Integration**. The wizard has three steps.

#### Step 1 — Select a provider

Choose from the list above. Use the search box to filter by name.

<figure><img src="../.gitbook/assets/Select Provider.png" alt=""><figcaption><p>Select a provider</p></figcaption></figure>

#### Step 2 — Integration details

Enter configurations and credentials.

| Field             | Required | Notes                                                                        |
| ----------------- | -------- | ---------------------------------------------------------------------------- |
| Name              | Yes      | Display name, e.g. "Anthropic Production"                                    |
| Provider Slug     | Yes      | Unique identifier used in API headers, gateway configs, and the Activity Log |
| Short Description | No       |                                                                              |
| Project Scope     | Yes      | "All Projects" or one or more specific projects                              |

> The **Provider Slug** appears in your Activity Log on every request, making it easy to trace which credential was used.

<figure><img src="../.gitbook/assets/Configure Provider.png" alt=""><figcaption><p>Add provider key &#x26; configuration</p></figcaption></figure>

**Credential fields** vary by provider. For most, you supply an API key. Three providers support multiple authentication modes:

**Google Vertex AI**

| Auth mode            | Required fields      |
| -------------------- | -------------------- |
| Service Account File | JSON key file upload |

**Microsoft Azure**

| Auth mode         | Required fields                                 |
| ----------------- | ----------------------------------------------- |
| Default (API Key) | API Key, Azure Resource Name, deployment config |

**Advanced options** (collapsed by default)

* **Custom Host** — override the provider's default base URL. When set, select an API format: OpenAI-compatible, Anthropic-compatible, or Cohere-compatible.
* **Custom Auth Headers** — forwarded to your endpoint without logging.

#### Step 3 — Model provisioning

> **FastRouter catalog models only.** Only models in the FastRouter Model Catalog are routable, even with auto-enable on. Contact support to request a model addition.

* Toggle catalog models on or off individually, or use All / None.
* For now, models from the catalog or custom models have to mapped per provider.
* \[COMING SOON: **Auto-enable** — new catalog models from this provider are enabled on this integration automatically.]

<figure><img src="../.gitbook/assets/Enable Models.png" alt=""><figcaption><p>Enable/disable specific provider models</p></figcaption></figure>

**Adding custom models**

Use **Add Custom Model** to register fine-tuned or privately hosted models.

| Field                    | Required | Notes                                                                                                                                             |
| ------------------------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Model Slug               | Yes      | The `model` identifier used in API calls. Must be unique within this integration.                                                                 |
| Base Model               | No       | An existing catalog model your custom model is API-compatible with. Tells FastRouter which request/response schema to use — no effect on pricing. |
| Custom Host              | No       | Override the endpoint for this specific model only.                                                                                               |
| Input / Output pricing   | No       | USD per 1M tokens, for cost tracking in the dashboard.                                                                                            |
| Additional token pricing | No       | JSON for provider-specific token categories, e.g. `{"cache_read_input_tokens": 0.30}`                                                             |

<figure><img src="../.gitbook/assets/Add Custom Model.png" alt=""><figcaption><p>Configure custom models &#x26; endpoints</p></figcaption></figure>

**Note:** Custom models can be edited or deleted at any time from the model list.

***

### Editing an integration

Click **Edit** on any integration card. The edit wizard has two steps — Integration Details and Model Provisioning. The provider cannot be changed after creation.

***

### Security

* Credentials are encrypted at rest and never returned in API responses after saving.
* Rate limits and costs are governed by your provider account, not FastRouter's shared key pools.
* Only Organization Owners can create, modify, or delete credentials.

***

### Using BYOK integrations

Once active, reference an integration via its **Provider Slug** in:

* **Virtual Models** — associate an alias with a specific integration
* **Gateway Configs** — use in fallback or load-balancing configurations
* **Activity Log** — every request shows the Provider Slug of the integration used
