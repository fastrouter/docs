---
description: >-
  API keys in FastRouter allow granular control over access, usage, and budget.
  You can generate multiple keys with custom configurations for different users,
  projects, or integrations.
icon: key-skeleton
---

# Keys & Settings

API Keys in **FastRouter** are tied to individual users and inherit their project-level permissions.\
They provide secure, scoped access to FastRouter’s APIs — ideal for development, experimentation, and fine-grained control within specific projects.

#### Key Characteristics

* **User-linked:** Each key belongs to a specific user.
* **Permission-aware:** Inherits the user’s project-level permissions.
* **Self-service:** Can be created by any project member.
* **Flexible use:** Best suited for testing, scoped integrations, or per-user API access.

{% embed url="https://youtu.be/cnVH9E27ppc" %}

***

### Creating an API Key

When you create an API key, **FastRouter generates a one-time visible token.** Be sure to copy and store it securely — it **cannot be retrieved later** for security reasons.

***

### Key Settings

#### **Key Name**

Provide a custom label to identify the key within your project.

***

#### **Budget Controls**

* Set a **maximum spend** (in USD) for this key.
* Leave blank for unlimited usage.

**Reset Budget**

Choose when the key’s budget resets:

* **Never** (default)
* **Daily**
* **Weekly**
* **Monthly**

***

#### **Advanced Settings**

**Select Models**

* Choose which **LLM models** this key can access.
* **Default:** All models selected under the project.

**Rate Limits: Tokens per Minute (TPM)**

* Limit how many tokens this key can consume per minute.
* Cannot exceed the project-level TPM cap.

**Rate Limits: Requests per Minute (RPM)**

* Set the maximum number of requests per minute.
* Cannot exceed the project-level RPM cap.

**Expire Key**

* Schedule an automatic expiration date and time.
* Useful for **temporary access**, **contractor accounts**, or **testing environments**.

***

#### **Disable Content Logging**

* Prevents logging of requests and responses associated with this key.
* Recommended for keys handling **sensitive or private data**.

***

### Best Practices

* **Rotate keys periodically** and revoke unused ones.
* **Set budget and rate limits** to prevent accidental overspending.
* **Tag keys by project or purpose** for better organization.
* **Restrict model access** for tighter control and security.

