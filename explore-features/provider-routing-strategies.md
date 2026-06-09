---
icon: less-than
---

# Provider Routing Strategies

### Provider Routing Strategies **Overview**

FastRouter allows you to control how requests are routed to providers using flexible provider sorting and filtering parameters. Below are the options you can use within the `provider` object in your API request to fine-tune routing behavior. Using these parameters, you can define intelligent strategies to optimize performance, cost, or availability.

### Available Strategies

* **Default Strategy (Prioritizing Low Cost & High Uptime):** FastRouter assigns higher priority to models and providers that meet a performance and uptime threshold and weights them based on the inverse square of their price. This approach enables selecting more cost-effective options without compromising reliability.
* **Lowest Latency**: Routes requests to the fastest responding provider.
* **Lowest Price**: Routes to the least expensive provider for the selected model.
* **Highest Throughput**: Prefers providers with the highest tokens/minute output.

### Provider Object Parameters

Each provider object used in a routing strategy can include:

| Parameter            | Description                                                                |
| -------------------- | -------------------------------------------------------------------------- |
| `order`              | Define the exact sequence of providers to be used for the request.         |
| `allow_fallbacks`    | Control whether to allow backup providers when the primary is unavailable. |
| `only`               | Restrict the request to a specific set of providers.                       |
| `ignore`             | Exclude listed providers from being used in the request.                   |
| `sort`               | Automatically sort providers by the specified option.                      |
| `require_parameters` | Only use providers that support all parameters in the request.             |

***

### **1. Explicit Provider Priority: `order`**

Use the `order` field to define the exact sequence of providers to be used for the request. Providers will be attempted in the specified order.

```bash
curl --location 'https://api.fastrouter.ai/api/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer API-KEY' \
--data '{
  "model": "openai/gpt-4o",
  "messages": [{"role": "user", "content": "How many r’s in strawberry?"}],
  "stream": true,
  "provider": {
    "order": ["azure", "openai"]
  }
}'
```

***

### **2. Enable Fallbacks: `allow_fallbacks`**

This controls whether to allow backup providers when the primary is unavailable. If set to `true`, fallback providers from the ordered list are used in sequence.

```bash
"provider": {
  "order": ["openai", "azure"],
  "allow_fallbacks": true
}
```

***

### **3. Restrict to Specific Providers: `only`**

Restrict the request to a specific set of providers. The `order` or `sort` logic will be applied only to this subset.

```bash
"provider": {
  "only": ["openai"]
}
```

***

### **4. Exclude Specific Providers: `ignore`**

Exclude listed providers from being used in the request, regardless of ordering or sorting.

```bash
"provider": {
  "order": ["openai", "azure"],
  "ignore": ["openai"]
}
```

***

### **5. Automatically Sort Providers: `sort`**

Automatically sort providers by:

* `"price"` – Choose the cheapest option.
* `"throughput"` – Choose provider with highest token throughput.
* `"latency"` – Choose the fastest responding provider.

#### Example: Sort by price

```bash
"provider": {
  "sort": "price",
  "ignore": ["openai"]
}
```

#### Example: Sort by throughput

```bash
"provider": {
  "sort": "throughput"
}
```

#### Example: Sort by latency

```bash
"provider": {
  "sort": "latency"
}
```

### **Sorting Slugs: Sorting Via Model Suffix**

You can also use model suffixes to trigger automatic provider sorting without explicitly setting the `provider.sort` field.

* `:throughput` → Shortcut for `"sort": "throughput"`
* `:price` → Shortcut for `"sort": "price"`

#### Example: `:throughput` (Throughput prioritized)

```bash
"model": "openai/gpt-4o:throughput"
```

#### Example: `:price` (Lowest price prioritized)

```bash
"model": "openai/gpt-4o:price"
```

***

### **6. Strict Parameter Support: `require_parameters`**

Set this to `true` to ensure only providers that support _all_ parameters in your request are considered. Defaults to `false`.

```bash
"provider": {
  "require_parameters": true
}
```

***

This flexible provider routing configuration gives you granular control over cost, speed, privacy, and reliability for each request. Use combinations of `order`, `sort`, `ignore`, and `only` to build advanced routing strategies on-the-fly.

