---
description: >-
  FastRouter supports Function Calling for models capable of planning and
  invoking tools or functions. This allows LLMs to return structured function
  calls instead of natural language responses.
icon: function
---

# Function Calling

### Overview

Function calling empowers LLMs to:

* **Identify** when a tool or function is needed based on user input.
* **Select** the appropriate function from a provided set of tools.
* **Generate** structured JSON arguments to invoke that function.

When you provide a list of tools in your API request, compatible models can choose to respond with one or more function calls. You then execute those functions in your application code and feed the results back to the model in a subsequent request. This creates a multi-turn conversation loop for tasks like data retrieval, API integrations, or complex workflows.

FastRouter.ai routes your requests to the best available providers (e.g., Google AI Studio, OpenAI) while supporting OpenAI-compatible formats for tools. This includes parallel function calling for models that support it.

**Key Benefits:**

* Build agents that interact with real-world APIs (e.g., weather services, calendars).
* Handle complex queries by breaking them into tool-based steps.
* Improve reliability with structured outputs over free-form text.

### Supported Models

FastRouter.ai supports function calling on models that natively offer this capability. Here's a partial list (check the [Models page ](https://fastrouter.ai/models)for the latest):

* **Google Models**: Gemini 2.5 Pro, Gemini 2.5 Flash and others
* **OpenAI Models**: GPT-5, GPT-4.1, GPT-4o, o4-mini, o3-mini and others
* **Anthropic Models**: Claude Opus 4.1, Claude Sonnet 4.5, Claude Haiku 4.5 and others
* **xAI Models**: Grok 4, Grok 3 and others

Use the `provider` field in your request to route to a specific backend if needed.

### Usage

To use function calling:

1. Define your tools in the `tools` array of your `/chat/completions` request. Each tool follows the OpenAI-compatible schema (a JSON object with `name`, `description`, and `parameters`).
2. Send the request to FastRouter.ai's endpoint: `https://api.fastrouter.ai/api/v1/chat/completions`.
3. If the model responds with a `tool_calls` array in the response, execute the functions in your code.
4. Append the tool results as a new message (with `role: "tool"`) and send a follow-up request to let the model generate a final response.

**Request Parameters:**

* `tools`: Array of tool definitions.
* `tool_choice`: Optional; controls how the model uses tools (e.g., `"auto"` for automatic selection, `"none"` to disable, or specify a tool name).

**Response Format:**

* If a tool is called, the response will include `choices[0].message.tool_calls`—an array of objects with `function.name` and `function.arguments` (JSON string).
* Execute the function and respond with a message like: `{"role": "tool", "content": "JSON result", "tool_call_id": "call_id_from_response"}`.

For authentication, use your API key in the `Authorization: Bearer YOUR_API_KEY` header.

### Executing Tools: Multi-Turn Example

After receiving a `tool_calls` response, execute the function in your code and send the result back. Here's how to handle a full loop.

#### Python (Using `requests`)

```python
import requests
import json

API_KEY = "YOUR_API_KEY"
ENDPOINT = "https://api.fastrouter.ai/api/v1/chat/completions"

# Step 1: Initial request with tools
payload = {
    "model": "openai/gpt-4o",
    "messages": [{"role": "user", "content": "What is the temperature in Paris?"}],
    "tools": [
        # Same tools as above...
    ]
}

response = requests.post(ENDPOINT, json=payload, headers={
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}).json()

# Step 2: Check for tool calls and execute
tool_calls = response["choices"][0]["message"].get("tool_calls")
if tool_calls:
    for tool_call in tool_calls:
        func_name = tool_call["function"]["name"]
        args = json.loads(tool_call["function"]["arguments"])
        
        # Simulate tool execution (replace with real API call)
        if func_name == "get_current_weather":
            result = {"temperature": 72, "condition": "Sunny"}  # Mock weather data
        # Add handling for other tools...
        
        # Step 3: Append tool result to messages
        payload["messages"].append(response["choices"][0]["message"])  # Add model's message
        payload["messages"].append({
            "role": "tool",
            "content": json.dumps(result),
            "tool_call_id": tool_call["id"]
        })
    
    # Step 4: Send follow-up request for final response
    final_response = requests.post(ENDPOINT, json=payload, headers={
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }).json()
    
    print(final_response["choices"][0]["message"]["content"])  # e.g., "The temperature in Paris is 72°F and sunny."
```

#### Node.js (Using `fetch`)

<pre class="language-javascript"><code class="lang-javascript">const API_KEY = 'YOUR_API_KEY';
const ENDPOINT = 'https://api.fastrouter.ai/api/v1/chat/completions';

async function main() {
  // Step 1: Initial request
  const payload = {
    model: 'openai/gpt-4o',
    messages: [{ role: 'user', content: 'What is the temperature in Paris?' }],
<strong>    tools: [ /* Same tools as above... */ ]
</strong>  };

  let response = await fetch(ENDPOINT, {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${API_KEY}`, 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  }).then(res => res.json());

  // Step 2: Handle tool calls
  const toolCalls = response.choices[0].message.tool_calls;
  if (toolCalls) {
    for (const toolCall of toolCalls) {
      const funcName = toolCall.function.name;
      const args = JSON.parse(toolCall.function.arguments);

      // Simulate execution
      let result;
      if (funcName === 'get_current_weather') {
        result = { temperature: 72, condition: 'Sunny' };  // Mock
      }
      // Add other tool handlers...

      // Step 3: Append to messages
      payload.messages.push(response.choices[0].message);
      payload.messages.push({
        role: 'tool',
        content: JSON.stringify(result),
        tool_call_id: toolCall.id
      });
    }

    // Step 4: Follow-up request
    response = await fetch(ENDPOINT, {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${API_KEY}`, 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    }).then(res => res.json());

    console.log(response.choices[0].message.content);  // Final response
  }
}

main();
</code></pre>

### Important: Preserving Reasoning Details in Multi-Turn Tool Calls

When using models that support reasoning (extended thinking) like Google's Gemini-3-Pro , the response may include `reasoning_details` in the assistant message. **You must pass these reasoning details back in subsequent requests** to maintain context continuity and allow the model to continue reasoning from where it left off.

#### Why This Matters

* Models with reasoning capabilities generate internal thinking blocks that inform their decisions
* Omitting `reasoning_details` in follow-up requests can lead to errors
* Preserving this context ensures the model maintains its chain of thought across tool execution

#### Example: Handling Reasoning Details with Tool Calls

```python
from openai import OpenAI

client = OpenAI(
 base_url="https://api.fastrouter.ai/api/v1",
 api_key="API-KEY",
)

# Define flight search tool
tools = [{
 "type": "function",
 "function": {
 "name": "search_flights",
 "description": "Search for available flights between two cities",
 "parameters": {
 "type": "object",
 "properties": {
 "origin": {"type": "string", "description": "Departure city"},
 "destination": {"type": "string", "description": "Arrival city"},
 "date": {"type": "string", "description": "Travel date (YYYY-MM-DD)"}
 },
 "required": ["origin", "destination", "date"]
 }
 }
}]

# First API call with reasoning enabled
response = client.chat.completions.create(
 model="google/gemini-3-pro-preview",
 messages=[
 {"role": "user", "content": "Find flights from New York to London on December 20th and recommend the best one."}
 ],
 tools=tools,
 extra_body={"reasoning": {"max_tokens": 2000}}
)

message = response.choices[0].message

print("First response:", message.content)
print("Tool calls:", message.tool_calls)

if message.tool_calls:
 # Simulate flight search results
 flight_results = {
 "flights": [
 {"airline": "British Airways", "departure": "08:00", "arrival": "20:00", "price": 450, "stops": 0},
 {"airline": "Delta", "departure": "14:30", "arrival": "02:30+1", "price": 380, "stops": 0},
 {"airline": "United", "departure": "19:00", "arrival": "07:00+1", "price": 320, "stops": 1}
 ]
 }
 
 # Build follow-up messages, preserving reasoning_details
 messages = [
 {"role": "user", "content": "Find flights from New York to London on December 20th and recommend the best one."},
 {
 "role": "assistant",
 "content": message.content,
 "tool_calls": message.tool_calls,
 "reasoning_details": getattr(message, 'reasoning_details', None)  # Preserve if exists
 },
 {
 "role": "tool",
 "tool_call_id": message.tool_calls[0].id,
 "content": str(flight_results)
 }
 ]
 
 # Second API call - model continues with preserved reasoning
 response2 = client.chat.completions.create(
 model="google/gemini-3-pro-preview",
 messages=messages,
 tools=tools
 )
 
 print("\
Recommendation:", response2.choices[0].message.content)
else:
 print("No tool calls were made.")

```

### Best Practices

* **Error Handling:** If tool execution fails, return an error message in the `content` field.
* **Security:** Validate arguments before executing tools to prevent injection attacks.
* **Streaming:** Function calling works with `stream: true`, but tool calls appear in the final chunk.
* **Costs:** Tool calls count toward token usage—monitor via the response's `usage` field.
* **Testing:** Start with simple tools and iterate based on model behavior.
* **Reasoning Details:** When using models with reasoning capabilities, preserve `reasoning_details` from the assistant message and include it in subsequent requests to maintain context continuity.
