---
icon: xmark
---

# Error Codes

### Error Code Overview

FastRouter uses standard HTTP status codes to indicate errors during API requests. Below is a list of common errors you might encounter, along with tips to resolve them.

| Code | Meaning                           | Description                                                                                                |
| ---- | --------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| 400  | **Bad Request**                   | The request is malformed. This could be due to missing parameters, invalid formats, or CORS issues.        |
| 401  | **Invalid Credentials**           | Your API key is invalid, disabled, or your OAuth session has expired. Check your credentials.              |
| 402  | **Insufficient Credits**          | Your account or API key has run out of credits. Add more credits and retry the request.                    |
| 403  | **Moderation Blocked**            | The input was flagged by the model’s content moderation system. Modify the prompt and try again.           |
| 429  | **Rate Limited**                  | You have exceeded your request limits (TPM/RPM). Slow down or increase your limits.                        |
| 500  | **Internal Error**                | Something went wrong on our side. Retry the request, and contact support if the issue persists.            |
| 502  | **Model Down / Invalid Response** | The selected model is currently unavailable or returned an invalid response. Retry or use fallback models. |
