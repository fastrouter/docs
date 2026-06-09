---
icon: layer-group
---

# Batch Processing

## FastRouter Batch API

Batch processing allows you to send large volumes of requests efficiently at reduced costs. Upload a JSONL file containing your requests, trigger the batch, monitor its progress, and download the results — all through a simple four-step workflow.

> **Note:** Batch requests are processed asynchronously within a specified completion window, making them ideal for non-time-sensitive workloads such as bulk evaluations, data processing, and large-scale content generation.

***

### 1. File Upload (POST Request)

Upload your JSONL file to FastRouter to initiate the batch process. This returns a `file_id` for use in subsequent steps.

{% openapi-operation spec="fastrouter-api" path="/v1/files" method="post" %}
[OpenAPI fastrouter-api](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/115e26023071c6163655edeabd2d24ef1a064ceb6c0e681b6d51361ec47cf990.json?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20260609%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20260609T185233Z&X-Amz-Expires=172800&X-Amz-Signature=538c27563a6be6525cc869ecb87162c49c6759eee1d4f24e1cc27d556071ccfe&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### 2. Trigger Batch (POST Request)

Once the file is uploaded, trigger the batch processing by providing the `input_file_id`, `endpoint`, and `completion_window` (currently accepts only `"24h"` for a 24-hour processing window).

{% openapi-operation spec="fastrouter-api" path="/v1/batches" method="post" %}
[OpenAPI fastrouter-api](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/115e26023071c6163655edeabd2d24ef1a064ceb6c0e681b6d51361ec47cf990.json?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20260609%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20260609T185233Z&X-Amz-Expires=172800&X-Amz-Signature=538c27563a6be6525cc869ecb87162c49c6759eee1d4f24e1cc27d556071ccfe&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### 3. Get Status (GET Request)

Check the status of your batch job using the `batch_id` returned from the trigger step. Statuses include `"in_progress"`, `"completed"`, `"failed"`, etc.

{% openapi-operation spec="fastrouter-api" path="/v1/batches/{batch_id}" method="get" %}
[OpenAPI fastrouter-api](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/115e26023071c6163655edeabd2d24ef1a064ceb6c0e681b6d51361ec47cf990.json?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20260609%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20260609T185233Z&X-Amz-Expires=172800&X-Amz-Signature=538c27563a6be6525cc869ecb87162c49c6759eee1d4f24e1cc27d556071ccfe&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

### 4. Download File (GET Request)

Once the batch is completed (verify via the status endpoint), download the output JSONL file containing responses for each request.

{% openapi-operation spec="fastrouter-api" path="/v1/files/{file_id}/content" method="get" %}
[OpenAPI fastrouter-api](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/115e26023071c6163655edeabd2d24ef1a064ceb6c0e681b6d51361ec47cf990.json?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20260609%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20260609T185233Z&X-Amz-Expires=172800&X-Amz-Signature=538c27563a6be6525cc869ecb87162c49c6759eee1d4f24e1cc27d556071ccfe&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

