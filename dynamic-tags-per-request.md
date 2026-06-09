---
description: >-
  Leverage dynamic tags for reporting, billing, or feature tracking with custom
  tags sent per request.
icon: tag
---

# Dynamic Tags Per Request

The `request_tags` parameter lets you add custom tags to each request, enabling dynamic tracking at the individual request level. It supports both flat and hierarchical formats, making it easy to organize usage data for reporting, cost allocation, team billing, feature usage, or environment segmentation. These tags are available in the dashboard as a filter and also in the meta data of the activity logs.

### Parameter Type:

```json
request_tags: [string]
```

### What You Can Do with Tags:

* Group and filter requests by project, team, environment, or feature
* Organize usage reports using nested tag paths (e.g., `team/feature/test`)
* Track granular usage without needing to manage multiple API keys

### Tag Formats Supported

<table><thead><tr><th width="124.91796875">Format Type</th><th width="301.203125">Example Tags</th><th>Description</th></tr></thead><tbody><tr><td><strong>Flat Tag</strong></td><td><code>["asia", "production"]</code></td><td>Simple, one-level tags</td></tr><tr><td><strong>Hierarchical</strong></td><td><code>["USA/NYC", "org/team/project"]</code></td><td>Path-style, nested classification</td></tr><tr><td><strong>Mixed</strong></td><td><code>[ "Asia", "UAE/Dubai" ]</code></td><td>Combine both types in the same request</td></tr></tbody></table>

### Example Request

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer API-KEY' \
--data '{
    "model": "openai/gpt-4o-mini",
    "stream": false,
    "messages": [
        {
            "role": "user",
            "content": "What are some famous tourist attractions in London?"
        }
    ],
    "temperature": 0.0,
    "request_tags": ["UK/London", "Europe"]
}'
```

### Why Use Tags?

* **Team-level visibility**: e.g., `marketing/email-gen`
* **Feature usage breakdown**: e.g., `feature/summary-widget`
* **Cost attribution**: e.g., `customer/acme`, `env/staging`
* **Geo-based tracking**: e.g., `US/California/SF` , `India/Mumbai`

