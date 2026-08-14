---
description: >-
  Add deterministic and LLM-based guardrails to protect your AI applications
  from unwanted behaviors and ensure compliance.
icon: badge-check
---

# Guardrails

### **Introduction**

Guardrails in FastRouter allow you to validate and monitor requests and responses flowing through your LLM gateway. They help you:

* Prevent sensitive information (PII) from being exposed
* Ensure responses adhere to specific topics or formats
* Block toxic or inappropriate content
* Validate output structure (regex patterns)

**⚠️ Feature Note**

Guardrails do not support streaming (`stream=true`). Ensure that `stream=false` in your request payload when using guardrails.

### Guardrail Actions

Each guardrail can be configured with one of two actions that determine how the system behaves when a guardrail check fails:

| Action   | Default State | Behavior                                                                                                                                                                                                                                                                                                                                         | Use Case                                                                                                                                                             |
| -------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Validate | —             | <p><strong>On Request &#x26; Response:</strong></p><ul><li>If <strong>any</strong> guardrail check <strong>fails</strong>, the request is killed with status code <code>446</code></li><li>If <strong>all</strong> guardrail checks <strong>succeed</strong>, the request/response proceeds with status code <code>200</code></li></ul>          | Use when guardrails are critical and a failed check should block the request entirely. **Recommended:** Test on a subset of requests first to understand the impact. |
| Observe  | ✓ Default     | <p><strong>On Request &#x26; Response:</strong></p><ul><li>If <strong>any</strong> guardrail check <strong>fails</strong>, the request still proceeds but with status code <code>246</code></li><li>If <strong>all</strong> guardrail checks <strong>succeed</strong>, the request/response proceeds with status code <code>200</code></li></ul> | Use when you want to log guardrail results without affecting your application flow. Ideal for monitoring and gathering insights before enforcing strict validation.  |

⚠️ **Testing Recommendation**

We strongly recommend running guardrails in **Observe** mode first on a subset of your requests to understand their impact before switching to **Validate** mode.

### Status Codes

FastRouter uses specific status codes to communicate guardrail results:

| Status Code | Description                                                                                                                  |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `200`       | All guardrail checks passed. Request/response processed successfully.                                                        |
| `246`       | **Observe mode:** One or more guardrail checks failed, but the request/response was still processed. Check logs for details. |
| `446`       | **Validate mode:** One or more guardrail checks failed. Request was blocked and not processed.                               |

### Using Guardrails in Requests

After creating guardrails in the FastRouter dashboard, you'll receive a **Config ID** (e.g., `gr_a8f3k9m2`) for each guardrail. Use these IDs to apply guardrails to your requests:

#### Adding Guardrails to Requests

Include the `input_guardrails` and/or `output_guardrails` parameters in your request body with an array of guardrail config IDs:

```shellscript
{
  "model": "openai/gpt-3.5-turbo",
  "messages": [
    {
      "role": "user",
      "content": "What is the meaning of life?"
    }
  ],
  "input_guardrails": ["gr_e9k2v7h5"],
  "output_guardrails": ["gr_a8f3k9m2", "gr_b2n7p1q4"]
}
```

In this example:

* `input_guardrails` - Applied to the incoming user request before it's sent to the LLM
* `output_guardrails` - Applied to the LLM's response before returning it to your application

#### Multiple Guardrails

You can apply multiple guardrails at each stage. They will be evaluated in the order specified:

```shellscript
{
  "model": "openai/gpt-4",
  "messages": [
    {
      "role": "user",
      "content": "Tell me about product pricing"
    }
  ],
  "input_guardrails": [
    "gr_e9k2v7h5",  // Topic Adherence
    "gr_a8f3k9m2"   // PII Check
  ],
  "output_guardrails": [
    "gr_b2n7p1q4",  // Toxicity Detection
    "gr_c5x8r3w9"   // Competitor Mention
  ]
}
```

### Guardrail Types

#### Basic Guardrails

Deterministic guardrails that run quickly and don't require LLM evaluation:

* **RegEx Check** - Validate content against custom regex patterns

#### LLM Judge Guardrails

Intelligent guardrails that use LLM evaluation for complex checks:

* **PII Check** - Detect and optionally redact emails, phone numbers, SSN, credit cards
* **Topic Adherence** - Ensure conversations stay on allowed topics
* **Toxicity Detection** - Detect hate speech, harassment, violence, and inappropriate content

**💡 Performance Note**

Basic guardrails execute in milliseconds. LLM Judge guardrails require an additional LLM call and may add 500ms-2s latency depending on your Default Guardrails Key configuration.

### Creating Guardrails

To create a new guardrail:

1. Navigate to **Guardrails** in your FastRouter dashboard
2. Click the **Browse Templates** tab
3. Choose a template and click **Create**
4. Configure your guardrail settings:
   * **Name** - A descriptive name for your guardrail
   * **Action** - Choose `Observe` or `Validate`
   * **Stage** - Choose `Input` or `Output`
   * Template-specific settings (e.g., PII types, word limits, regex patterns)
5. Click **Create Guardrail**
6. Copy the generated **Config ID** to use in your requests

### Best Practices

* **Start with Observe mode** - Monitor guardrail behavior before enforcing validation
* **Test incrementally** - Apply guardrails to a small percentage of traffic first
* **Layer guardrails** - Use both Basic and LLM Judge guardrails for comprehensive protection
* **Monitor costs** - LLM Judge guardrails consume tokens from your Default Guardrails Key
* **Keep guardrails focused** - Create specific guardrails for specific use cases rather than one large guardrail
* **Review logs regularly** - Check status `246` responses to understand when guardrails are triggered

### Example: Complete Request Flow

```shellscript
// Request with input and output guardrails
POST https://api.fastrouter.ai/v1/chat/completions
Authorization: Bearer YOUR_API_KEY

{
  "model": "openai/gpt-4",
  "messages": [
    {
      "role": "user",
      "content": "What's your email for support?"
    }
  ],
  "input_guardrails": ["gr_e9k2v7h5"],
  "output_guardrails": ["gr_a8f3k9m2"]
}

// Response (200 - All checks passed)
{
  "id": "fr_...",
  "model": "gpt-4",
  "choices": [...],
  "guardrails": {
    "input": {
      "passed": true,
      "checks": [
        {
          "id": "gr_e9k2v7h5",
          "name": "Topic Adherence",
          "passed": true,
          "cost": 0.00011
        }
      ]
    },
    "output": {
      "passed": true,
      "checks": [
        {
          "id": "gr_a8f3k9m2",
          "name": "PII Check",
          "passed": true,
          "cost": 0.00012
        }
      ]
    }
  }
}

// Response (246 - Check failed in Observe mode)
// Request still processed, but guardrail logged failure
{
  "id": "fr_...",
  "model": "gpt-4",
  "choices": [...],
  "guardrails": {
    "output": {
      "passed": false,
      "checks": [
        {
          "id": "gr_a8f3k9m2",
          "name": "PII Check",
          "passed": false,
          "cost": 0.00013
        }
      ]
    }
  }
}

// Response (446 - Check failed in Validate mode)
// Request blocked
{
  "error": {
    "message": "Guardrail validation failed",
    "type": "guardrail_error",
    "code": 446,
    "guardrails": {
      "output": {
        "passed": false,
        "checks": [
          {
            "id": "gr_a8f3k9m2",
            "name": "PII Check",
            "passed": false,
            "cost": 0.00014
          }
        ]
      }
    }
  }
}
```

<br>
