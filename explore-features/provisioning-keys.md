---
description: >-
  Provisioning Keys are special-purpose administrative tokens used to securely
  create, update, list, and delete Service Account Keys within your
  organization.
icon: key-skeleton-left-right
---

# Provisioning Keys

### Provisioning Keys **Overview**

* **Not for LLM Requests**: Provisioning Keys cannot be used to call model endpoints or route completions.
* **Org-Level Admin Only**: Only **Organization Owners** can generate and manage Provisioning Keys.
* **Service Key Management**: These keys provide full lifecycle control over Service Account Keys — including scoped permissions, rate limits, expiration, and metadata.

### Use Cases

Provisioning Keys are ideal for:

* Automating API key management
* Managing non-user-bound access (e.g., for CI/CD pipelines)
* Setting granular controls on model usage, budgets, and project tagging

### API Endpoints

#### Create a Service Account Key

```bash
curl 'https://api.fastrouter.ai/prod/createServiceKey' \
  -H 'accept: application/json' \
  -H 'accept-language: en-US,en;q=0.9' \
  -H 'authorization: Bearer PROVISIONING-KEY' \
  --data-raw '{
    "api_key_name": "SERVICE-KEY-NAME",
    "credit_limit": null,
    "reset_budget_interval": null,
    "expire_key": null,
    "models": null,
    "tpm_limit": null,
    "rpm_limit": null,
    "meta_data": null,
    "tags": null
  }'

```

The response includes the secret Service Account Key and its one-way hash, which you'll need for future updates or deletion.

**Note:** The following fields in the `createServiceKey` payload are **optional**:

* `credit_limit`
* `reset_budget_interval`
* `expire_key`
* `models`
* `tpm_limit`
* `rpm_limit`
* `meta_data`
* `tags`

These can be omitted entirely or set to `null` depending on your provisioning use case. Only `org_id` , `project_id` , `project_name` and `api_key_name` are required.

#### Update a Service Account Key

```bash
curl 'https://api.fastrouter.ai/prod/updateServiceKey' \
  -H 'accept: application/json' \
  -H 'accept-language: en-US,en;q=0.9' \
  -H 'authorization: Bearer PROVISIONING-KEY' \
  --data-raw '{
    "api_key_id_hash": "SERVICE-KEY-HASH",
    "api_key_name": "SERVICE-KEY-NAME",
    "credit_limit": 150,
    "reset_budget_interval": "daily",
    "expire_key": "2025-03-28 18:30:00.000",
    "models": [
      "GROQ/deepseek-r1-distill-llama-70b",
      "GROQ/llama-3.1-8b-instant"
    ],
    "tpm_limit": 10000,
    "rpm_limit": 10,
    "metadata": {
      "version": "v1",
      "source": "system",
      "data": {
        "user_id": "user_id",
        "name": "raw_key",
        "email": "hashed_key",
        "api_key_alias": "alias1"
      }
    },
    "tags": ["tag1", "tag2"]
}'
```

#### List Service Account Keys

```bash
curl 'https://api.fastrouter.ai/prod/getServiceKeys' \
  -H 'accept: application/json' \
  -H 'accept-language: en-US,en;q=0.9' \
  -H 'authorization: Bearer PROVISIONING-KEY'
```

#### Delete a Service Account Key

```bash
curl 'https://api.fastrouter.ai/prod/deleteServiceKey' \
  -H 'accept: application/json' \
  -H 'accept-language: en-US,en;q=0.9' \
  -H 'authorization: Bearer PROVISIONING-KEY' \
  --data-raw '{
    "api_key_id_hash": "SERVICE-KEY-HASH"
}'
```

### Notes

* Ensure your `PROVISIONING-KEY` is stored securely. It has high privileges.
* For `metadata`, you can store custom info like `user_id`, `team_name`, or `deployment`.
* `tags` help in grouping and filtering keys by project or environment.





