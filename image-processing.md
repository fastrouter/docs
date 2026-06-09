---
description: A guide to sending images via Base64 or URLs in API requests.
icon: file-image
---

# Image Processing

### Overview

FastRouter lets you include images in chat completions using an OpenAI-compatible message format. This enables vision use cases like image captioning, visual Q\&A, object recognition, and multimodal chat across supported providers and models.

***

### Supported Image Types

* **Images:** URLs or base64-encoded data URLs
  * Common formats: **jpg, jpeg, png, webp** (and others depending on the model/provider)

***

### Supported Models

Use models that support **Image** as an input modality in the FastRouter model catalog:\
[https://fastrouter.ai/models?input\_modalities=image](https://fastrouter.ai/models?input_modalities=image)

Examples include multimodal/vision-capable models such as `x-ai/grok-4` (availability varies).

***

### Sending Image Inputs

Images are sent inside `messages[].content[]` as entries of type `"image_url"`, either pointing to a public URL or a base64-encoded data URL.

#### Example: Sending an Image via URL (Python)

```python
import requests

url = "https://api.fastrouter.ai/api/v1/chat/completions"
headers = {
  "Authorization": "Bearer API-KEY",
  "Content-Type": "application/json"
}

messages = [
  {
    "role": "user",
    "content": [
      {"type": "text", "text": "Describe this image and identify any landmarks."},
      {
        "type": "image_url",
        "image_url": {
          "url": "https://upload.wikimedia.org/wikipedia/commons/d/da/Taj-Mahal.jpg"
        }
      }
    ]
  }
]

payload = {
  "model": "openai/gpt-4.1-nano",
  "messages": messages
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

#### Example: Sending a Base64-Encoded Image (Python)

Use base64 data URLs for local/private images.

```python
import requests
import base64

def encode_image_to_base64(image_path: str) -> str:
  with open(image_path, "rb") as f:
    return base64.b64encode(f.read()).decode("utf-8")

url = "https://api.fastrouter.ai/api/v1/chat/completions"
headers = {
  "Authorization": "Bearer API-KEY",
  "Content-Type": "application/json"
}

image_path = "/path/to/your/image.jpg"
base64_image = encode_image_to_base64(image_path)

# Match the MIME type to your file (e.g., image/png for .png)
data_url = f"data:image/jpeg;base64,{base64_image}"

messages = [
  {
    "role": "user",
    "content": [
      {"type": "text", "text": "What's in this image?"},
      {"type": "image_url", "image_url": {"url": data_url}}
    ]
  }
]

payload = {
  "model": "openai/gpt-4.1-nano",
  "messages": messages
}

response = requests.post(url, headers=headers, json=payload)
print(response.json())
```

***

### Tips & Best Practices

* **Put text first:** Send your prompt before the image(s) for best results.
* **Multiple images:** Add multiple `"image_url"` items within the same `content` array if your model supports it.
* **Prefer URLs for large images:** URLs avoid base64 overhead and request-size limits.
* **Use the correct MIME type:** For base64 data URLs, ensure `data:image/...` matches the actual file type.
* **Model limits vary:** Image count/size support differs by model/provider — check the model’s docs in the FastRouter catalog.
