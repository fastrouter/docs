---
description: >-
  FastRouter supports structured JSON outputs, allowing you to enforce a
  specific schema in LLM responses. This feature ensures that responses are
  machine-readable and conform to a predefined structure.
icon: square-code
---

# Structured Outputs

### **Overview**

Using the `response_format` parameter, you can define a custom JSON schema that the model must follow. FastRouter will guide compatible models to format their outputs accordingly.

For a live, browsable list of models that support Structured Outputs, see the [**Models directory**](https://fastrouter.ai/models?supported_parameters=structured_outputs\&order=newest).

### **Supported Format**

* **Type**: `json_schema`
* **Schema**: Defined using standard [JSON Schema](https://json-schema.org/) format
* **Strict Mode**: If `strict: true`, compatible models will respond with a JSON object that strictly follows your schema.

### **Example: Enforcing Weather Data Schema**

This example instructs the model to return weather data in a strict JSON format:

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer API-KEY' \
--data '{
    "model": "openai/gpt-4o",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant that responds in JSON format."
      },
      {
        "role": "user",
        "content": "What is the weather like in London?"
      }
    ],
    "response_format": {
      "type": "json_schema",
      "json_schema": {
        "name": "weather",
        "strict": true,
        "schema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "City or location name"
            },
            "temperature": {
              "type": "number",
              "description": "Temperature in Celsius"
            },
            "conditions": {
              "type": "string",
              "description": "Weather conditions description"
            }
          },
          "required": ["location", "temperature", "conditions"],
          "additionalProperties": false
        }
      }
    }
}' 
```

### **Response Example**

```json
{
  "location": "London",
  "temperature": 17.5,
  "conditions": "Partly cloudy"
}
```

### **Schema Validation Options**

| Parameter              | Description                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| `type`                 | Must be set to `"json_schema"`                                                  |
| `name`                 | Optional name for the schema block (useful for logging/debugging)               |
| `strict`               | Informs the provider to validate the output against the schema (`true`/`false`) |
| `schema`               | A valid [JSON Schema](https://json-schema.org/) definition                      |
| `additionalProperties` | Set to `false` to block unexpected fields in the response                       |

### **Best Practices**

* Always define a `system` prompt that clearly instructs the model to respond in JSON format.
* Use `strict: true` for use cases that demand guaranteed structure.
* Combine with `stream: false` for easier schema validation (as streaming responses may break structure).
* Validate responses client-side to catch schema mismatches early.

