---
description: >-
  End-to-end guide to PDF ingestion in FastRouter chat completions, including
  plugin config, request examples, and billing details.
icon: file-pdf
---

# PDF Processing

***

### Introduction

FastRouter supports PDF understanding in chat completions by attaching PDFs to messages and enabling the **file-parser** plugin with the **mistral-ocr** engine.

This allows you to ask questions about PDFs (including scanned/image PDFs) using your chosen chat model—FastRouter will OCR/parse the document and provide the extracted text to the model.

**Pricing:** PDF OCR via `mistral-ocr` is billed at **$2 / 1,000 pages**, in addition to normal model token usage.

**Note:** If you’re using a model with native document understanding (e.g., `openai/gpt-4.1`), you can leverage the model’s built-in document processing — no plugin may be needed.

***

### Endpoint

**POST** `https://api.fastrouter.ai/api/v1/chat/completions`

***

### How PDF processing works

When you include a PDF in `messages[].content[]` and enable the plugin:

* FastRouter fetches/decodes the PDF (URL or base64)
* Runs **mistral-ocr** to extract text (works well on scanned pages and embedded images)
* Feeds the extracted content into the target model so it can answer your prompt

***

### Plugin configuration (required for OCR)

Enable PDF processing with the `plugins` parameter:

```json
{
  "plugins": [
    {
      "id": "file-parser",
      "pdf": {
        "engine": "mistral-ocr"
      }
    }
  ]
}
```

***

### Sending PDFs in messages

Attach PDFs inside a message’s `content` array using `type: "file"`:

```json
{
  "type": "file",
  "file": {
    "filename": "document.pdf",
    "file_data": "https://example.com/document.pdf"
  }
}
```

#### Supported `file_data` formats

* **Public URL** (recommended): `https://.../file.pdf`
* **Base64 data URL** (for local/private docs): `data:application/pdf;base64,JVBERi0xLjc...`

***

### Example: PDF via public URL

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.fastrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "z-ai/glm-4.7",
    "messages": [
      {
        "role": "user",
        "content": [
          { "type": "text", "text": "Summarize this PDF in 5 bullets and list any key dates." },
          {
            "type": "file",
            "file": {
              "filename": "document.pdf",
              "file_data": "https://example.com/document.pdf"
            }
          }
        ]
      }
    ],
    "plugins": [
      {
        "id": "file-parser",
        "pdf": {
          "engine": "mistral-ocr"
        }
      }
    ]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

url = "https://api.fastrouter.ai/api/v1/chat/completions"
headers = {
    "Authorization": f"Bearer {FASTROUTER_API_KEY}",
    "Content-Type": "application/json",
}

payload = {
    "model": "anthropic/claude-4.5-sonnet",
    "messages": [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What does this document say about termination clauses? Quote the relevant sections."},
                {
                    "type": "file",
                    "file": {
                        "filename": "contract.pdf",
                        "file_data": "https://example.com/contract.pdf",
                    },
                },
            ],
        }
    ],
    "plugins": [
        {
            "id": "file-parser",
            "pdf": {"engine": "mistral-ocr"},
        }
    ],
}

resp = requests.post(url, headers=headers, json=payload)
print(resp.json())
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
const response = await fetch("https://api.fastrouter.ai/api/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FASTROUTER_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "z-ai/glm-4.7",
    messages: [
      {
        role: "user",
        content: [
          { type: "text", text: "Extract the invoice total, invoice date, and vendor name." },
          {
            type: "file",
            file: {
              filename: "invoice.pdf",
              file_data: "https://example.com/invoice.pdf",
            },
          },
        ],
      },
    ],
    plugins: [
      {
        id: "file-parser",
        pdf: { engine: "mistral-ocr" },
      },
    ],
  }),
});

const data = await response.json();
console.log(data);
```
{% endtab %}
{% endtabs %}

***

### Example: PDF via base64 (data URL)

Use base64 when the PDF is local or not publicly reachable.

{% tabs %}
{% tab title="cURL" %}
```bash
curl https://api.fastrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $FASTROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "z-ai/glm-4.7",
    "messages": [
      {
        "role": "user",
        "content": [
          { "type": "text", "text": "Summarize this PDF in 5 bullets and list any key dates." },
          {
            "type": "file",
            "file": {
              "filename": "document.pdf",
              "file_data": "data:application/pdf;base64,..."
            }
          }
        ]
      }
    ],
    "plugins": [
      {
        "id": "file-parser",
        "pdf": {
          "engine": "mistral-ocr"
        }
      }
    ]
  }'
```
{% endtab %}

{% tab title="Python" %}
```python
import base64
import requests
from pathlib import Path

def pdf_to_data_url(path: str) -> str:
    b = Path(path).read_bytes()
    return "data:application/pdf;base64," + base64.b64encode(b).decode("utf-8")

data_url = pdf_to_data_url("path/to/document.pdf")

payload = {
    "model": "anthropic/claude-haiku-4.5",
    "messages": [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Give me a structured outline of this document with headings."},
                {
                    "type": "file",
                    "file": {
                        "filename": "document.pdf",
                        "file_data": data_url,
                    },
                },
            ],
        }
    ],
    "plugins": [
        {
            "id": "file-parser",
            "pdf": {"engine": "mistral-ocr"},
        }
    ],
}

resp = requests.post(
    "https://api.fastrouter.ai/api/v1/chat/completions",
    headers={"Authorization": f"Bearer {FASTROUTER_API_KEY}", "Content-Type": "application/json"},
    json=payload,
)
print(resp.json())
```
{% endtab %}

{% tab title="Node.js" %}
```javascript
import fs from "node:fs/promises";

const buf = await fs.readFile("path/to/document.pdf");
const dataUrl = `data:application/pdf;base64,${buf.toString("base64")}`;

const res = await fetch("https://api.fastrouter.ai/api/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.FASTROUTER_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "z-ai/glm-4.7",
    messages: [
      {
        role: "user",
        content: [
          { type: "text", text: "List the risks mentioned and the mitigation steps." },
          {
            type: "file",
            file: { filename: "risk-report.pdf", file_data: dataUrl },
          },
        ],
      },
    ],
    plugins: [
      {
        id: "file-parser",
        pdf: { engine: "mistral-ocr" },
      },
    ],
  }),
});

console.log(await res.json());
```
{% endtab %}
{% endtabs %}

***

### Pricing

* **mistral-ocr:** **$2 / 1,000 pages** processed
* **Model tokens:** billed normally for the model you select
* Prefer **URLs** for large PDFs to avoid request size limits and base64 overhead.

***
