---
description: >-
  Add per-user OAuth tool calling to your FastRouter agent with Scalekit. Gmail,
  GitHub, Slack, and more, with zero custom OAuth code.
icon: circle-nodes
---

# Scalekit

### What is Scalekit?

Scalekit is an SDK that gives AI agents secure, per-user access to external services like Gmail, GitHub, and Slack. It handles OAuth flows, token storage, token refresh, tool discovery, and tool execution server-side. Your agent never touches raw access tokens.

By routing your agent through FastRouter with Scalekit, you get:

* **100+ models** from OpenAI, Anthropic, Google, xAI, Meta, Groq, Mistral, and more through one endpoint
* **Observability** for every request: cost, tokens, latency, and model selection tracked in real time
* **Reliability** through automatic failover across providers, response caching, and intelligent routing
* **Governance** with per-key budgets, rate limits, model restrictions, role-based access, and project isolation
* **Per-user OAuth** with zero custom OAuth code — Scalekit manages credentials for each connected account

This guide covers FastRouter + Scalekit integration, model selection, team governance, and production feature usage.

### Overview

FastRouter supports function calling out of the box. [Scalekit](https://docs.scalekit.com/agentkit) extends that with per-user OAuth tool access, so your agent can read Gmail, create GitHub issues, or post to Slack on behalf of individual users. You can choose from [100+ connectors](https://docs.scalekit.com/agentkit/connectors/).

The integration is one configuration change: point the OpenAI SDK's `baseURL` at FastRouter. Scalekit handles OAuth token storage, tool discovery, and tool execution. Your agent never touches raw access tokens.

**Sample repository:** [github.com/scalekit-developers/fastrouter-scalekit-demo](https://github.com/scalekit-developers/fastrouter-scalekit-demo)

***

### What you are building

**FastRouter as the LLM provider.** All chat completions go through FastRouter's OpenAI-compatible endpoint. Switch models by changing one environment variable.

**Scalekit for tool access.** `listScopedTools` returns per-user tool schemas ready to pass directly to FastRouter. `executeTool` runs each tool server-side and returns structured results.

**B2B OAuth without custom OAuth code.** Scalekit handles the OAuth flow, token storage, and refresh for each connected service. Your agent gets an auth link, waits for the user to authorize, and receives a verified, active connected account.

**Agentic loop.** The agent calls FastRouter, receives tool calls, executes them through Scalekit, and feeds results back, repeating until FastRouter returns a final answer.

***

### Prerequisites

* FastRouter account and API key ([sign up at fastrouter.ai](https://fastrouter.ai))
* Scalekit account with AgentKit enabled ([create one at app.scalekit.com](https://app.scalekit.com))
* At least one AgentKit connection configured (Gmail, GitHub, or Slack)
* Node.js 20 or later

***

### Clone and run the sample

#### 1. Clone the repository and install dependencies

```sh
git clone https://github.com/scalekit-developers/fastrouter-scalekit-demo
cd fastrouter-scalekit-demo
npm install
```

#### 2. Copy the example environment file and fill in your credentials

```sh
cp .env.example .env
```

Open `.env` and set these values:

```sh
# FastRouter — find your API key at fastrouter.ai/dashboard
FASTROUTER_API_KEY=sk-v1-...
FASTROUTER_BASE_URL=https://api.fastrouter.ai/api/v1
FASTROUTER_MODEL=openai/gpt-4o

# Scalekit — find these in your Scalekit dashboard under Developers → API credentials
SCALEKIT_ENVIRONMENT_URL=https://your-env.scalekit.dev
SCALEKIT_CLIENT_ID=your_client_id
SCALEKIT_CLIENT_SECRET=your_client_secret

# The AgentKit connection to use — must match a connection name in your dashboard
SCALEKIT_CONNECTION_NAME=gmail
SCALEKIT_IDENTIFIER=user_123
```

`SCALEKIT_CONNECTION_NAME` must match the exact connection name in your Scalekit dashboard under **AgentKit > Connections**.

`FASTROUTER_MODEL` accepts any model in the [FastRouter catalog](https://fastrouter.ai/models) that supports function calling.

#### 3. Run the agent

```sh
npm start
```

#### 4. Authorize the connection on first run

The agent prints an authorization link if the connected account is not yet active:

```
Authorization required.
Open this link and complete the flow:

https://your-env.scalekit.dev/magicLink/...

Waiting for callback on http://localhost:3000/callback ...
```

Open the link in your browser and complete the OAuth flow. The agent detects the callback automatically and continues.

After authorization, the agent loads tools, calls FastRouter, and prints a final answer:

```
Connected account is now active.
Loaded 17 scoped tools from Scalekit.
Model requested 1 tool call(s).

 Executing gmail_list_messages
  args: {"maxResults":5,"q":"is:unread"}

Final answer:

Here are your 5 most recent unread emails: ...
```

***

### How the agent works

Three pieces connect FastRouter to Scalekit tools.

#### 1. Initialize FastRouter using the OpenAI SDK

FastRouter's API is OpenAI-compatible. Point `baseURL` at FastRouter and pass your FastRouter API key:

```typescript
import OpenAI from 'openai';

const fastRouter = new OpenAI({
  apiKey: process.env.FASTROUTER_API_KEY,
  baseURL: process.env.FASTROUTER_BASE_URL ?? 'https://api.fastrouter.ai/api/v1',
});
```

No other FastRouter-specific setup is required. The standard `openai` package works as-is.

#### 2. B2B OAuth connects user accounts without custom token code

Scalekit handles the full OAuth flow. Your agent calls `getOrCreateConnectedAccount` to check whether the user's account is already connected, then calls `getAuthorizationLink` to get an auth URL if it isn't.

{% tabs %}
{% tab title="Node.js" %}
```typescript
import { ConnectorStatus } from '@scalekit-sdk/node/lib/pkg/grpc/scalekit/v1/connected_accounts/connected_accounts_pb.js';
import crypto from 'node:crypto';

const userVerifyUrl = 'http://localhost:3000/callback';

// Generate a random state value and store it (e.g. in a secure cookie or session)
// to validate on the OAuth callback and prevent CSRF / account mix-up attacks.
const state = crypto.randomUUID();

const { connectedAccount } = await scalekit.actions.getOrCreateConnectedAccount({
  connectionName: 'gmail',
  identifier: 'user_123',
  userVerifyUrl,
});

if (connectedAccount?.status !== ConnectorStatus.ACTIVE) {
  const { link } = await scalekit.actions.getAuthorizationLink({
    connectionName: 'gmail',
    identifier: 'user_123',
    userVerifyUrl,
    state,
  });
  // Show link to user, then wait for the browser redirect callback
}
```
{% endtab %}

{% tab title="Python" %}
```python
import secrets

user_verify_url = "http://localhost:3000/callback"

# Generate and store a state value (e.g. in a secure, HTTP-only cookie) for CSRF protection
state = secrets.token_urlsafe(32)

response = scalekit_client.actions.get_or_create_connected_account(
    connection_name="gmail",
    identifier="user_123",
    userVerifyUrl,
)

if response.connected_account.status != "ACTIVE":
    link_resp = scalekit_client.actions.get_authorization_link(
        connection_name="gmail",
        identifier="user_123",
        user_verify_url=user_verify_url,
        state=state,
    )
    # Show link_resp.link to the user
```
{% endtab %}
{% endtabs %}

`userVerifyUrl` is where Scalekit redirects the user's browser after the OAuth flow completes (a GET request with `auth_request_id` and `state` query parameters). The sample runs a minimal HTTP server on `localhost:3000` to catch that redirect, validate the `state` against the original value, extract the `auth_request_id`, and call `verifyConnectedAccountUser` to mark the account active:

{% tabs %}
{% tab title="Node.js" %}
```typescript
async function waitForCallback(port: number, expectedState: string): Promise<string> 
  return new Promise((resolve, reject) => {
    const server = http.createServer((req, res) => {
      const url = new URL(req.url ?? '/', `http://localhost:${port}`);
      const authRequestId = url.searchParams.get('auth_request_id');
      const returnedState = url.searchParams.get('state');

      res.writeHead(200, { 'Content-Type': 'text/html' });
      res.end('<html><body><h2>Authorization complete — return to your terminal.</h2></body></html>');
      server.close();

      if (authRequestId && returnedState === expectedState) {
        resolve(authRequestId);
      } else {
        reject(new Error('Invalid or missing auth_request_id or state in callback'));
      }
    });
    server.listen(port);
  });
}

const authRequestId = await waitForCallback(3000, state);
await scalekit.actions.verifyConnectedAccountUser({
  authRequestId,
  identifier: 'user_123',
});
```
{% endtab %}

{% tab title="Python" %}
```python
# In your web framework callback handler (e.g. FastAPI):
# 1. Validate that the "state" query param matches the value you stored earlier
# 2. Then exchange the auth_request_id (never trust identity from the URL alone)

result = scalekit_client.actions.verify_connected_account_user(
    auth_request_id=auth_request_id,
    identifier="user_123",
)
# redirect to result.post_user_verify_redirect_url
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**Production callback endpoint:** In a production web app, replace `localhost:3000/callback` with your server's callback endpoint. Scalekit redirects the browser to it with `auth_request_id` and `state` query params. Your handler must validate the state before calling `verifyConnectedAccountUser` to complete account activation.
{% endhint %}

#### 3. Tool discovery returns schemas in FastRouter's expected format

`listScopedTools` returns only the tools the connected account has permission to use. Map each tool's `input_schema` to the `parameters` field FastRouter expects:

{% tabs %}
{% tab title="Node.js" %}
```typescript
const { tools } = await scalekit.tools.listScopedTools('user_123', {
  filter: { connectionNames: ['gmail'] },
  pageSize: 100,
});

const fastRouterTools = tools
  .map((t) => t.tool?.definition)
  .filter((def): def is NonNullable<typeof def> => Boolean(def?.name))
  .map((def) => ({
    type: 'function' as const,
    function: {
      name: String(def.name),
      description: String(def.description ?? ''),
      parameters: def.input_schema ?? { type: 'object', properties: {} },
    },
  }));
```
{% endtab %}

{% tab title="Python" %}
```python
from google.protobuf.json_format import MessageToDict

scoped_response, _ = scalekit_client.actions.tools.list_scoped_tools(
    identifier="user_123",
    filter={"connection_names": ["gmail"]},
)

fast_router_tools = [
    {
        "type": "function",
        "function": {
            "name": MessageToDict(tool.tool).get("definition", {}).get("name"),
            "description": MessageToDict(tool.tool).get("definition", {}).get("description", ""),
            "parameters": MessageToDict(tool.tool).get("definition", {}).get("input_schema", {}),
        },
    }
    for tool in scoped_response.tools
]
```
{% endtab %}
{% endtabs %}

FastRouter uses the same function-calling format as OpenAI. No additional schema transformation is needed.

#### 4. The agentic loop runs until the model stops requesting tools

Pass the tool list to FastRouter and execute each tool call through Scalekit until the model returns a response with no tool calls:

{% tabs %}
{% tab title="Node.js" %}
```typescript
const messages: OpenAI.ChatCompletionMessageParam[] = [
  { role: 'system', content: 'You are a helpful assistant. Use tools when they help. Do not invent tool results.' },
  { role: 'user', content: 'Fetch my last 5 unread emails and summarize them.' },
];

for (let turn = 0; turn < 8; turn++) {
  const response = await fastRouter.chat.completions.create({
    model: process.env.FASTROUTER_MODEL ?? 'openai/gpt-4o',
    messages,
    tools: fastRouterTools,
    tool_choice: 'auto',
  });

  const message = response.choices[0].message;
  messages.push(message);

  // No tool calls means a final answer
  if (!message.tool_calls?.length) {
    console.log(message.content);
    return;
  }

  // Execute each tool call and append the result
  for (const call of message.tool_calls) {
    const result = await scalekit.actions.executeTool({
      toolName: call.function.name,
      identifier: 'user_123',
      connector: 'gmail',
      toolInput: JSON.parse(call.function.arguments),
    });

    messages.push({
      role: 'tool',
      tool_call_id: call.id,
      content: JSON.stringify(result.data ?? {}),
    });
  }
}
```
{% endtab %}

{% tab title="Python" %}
```python
from openai import OpenAI

fast_router = OpenAI(
    api_key=os.environ["FASTROUTER_API_KEY"],
    base_url=os.environ.get("FASTROUTER_BASE_URL", "https://api.fastrouter.ai/api/v1"),
)

messages = [
    {"role": "system", "content": "You are a helpful assistant. Use tools when they help. Do not invent tool results."},
    {"role": "user", "content": "Fetch my last 5 unread emails and summarize them."},
]

for turn in range(8):
    response = fast_router.chat.completions.create(
        model=os.environ.get("FASTROUTER_MODEL", "openai/gpt-4o"),
        messages=messages,
        tools=fast_router_tools,
        tool_choice="auto",
    )

    message = response.choices[0].message
    messages.append(message)

    # No tool calls means a final answer
    if not message.tool_calls:
        print(message.content)
        break

    # Execute each tool call and append the result
    for call in message.tool_calls:
        result = scalekit_client.actions.execute_tool(
            tool_input=json.loads(call.function.arguments),
            tool_name=call.function.name,
            identifier="user_123",
            connection_name="gmail",
        )

        messages.append({
            "role": "tool",
            "tool_call_id": call.id,
            "content": json.dumps(result.data or {}),
        })
```

`executeTool` runs the tool server-side using the connected account's stored OAuth tokens. Your agent never handles raw access tokens.
{% endtab %}
{% endtabs %}

***

### Customize the agent

**Change the model.** Set `FASTROUTER_MODEL` in `.env` to any model in the [FastRouter catalog](https://fastrouter.ai/models) that supports function calling. The agent code stays identical regardless of which model you pick.

#### Model Examples

Switch to a different provider or model at any time:

```sh
# Anthropic Claude
FASTROUTER_MODEL=anthropic/claude-sonnet-4.6

# Google Gemini
FASTROUTER_MODEL=google/gemini-3-pro

# Mistral
FASTROUTER_MODEL=mistralai/Mistral-Small-24B-Instruct-2501

# xAI Grok
FASTROUTER_MODEL=x-ai/grok-4
```

No code changes or SDK swaps required. Update the model string and your agent uses the new provider.

#### Automatic Model Selection

Let FastRouter pick the best model for each request based on query complexity, domain, and cost:

```sh
FASTROUTER_MODEL=fastrouter/auto
```

FastRouter analyzes the input and routes to the most appropriate model from the available pool. This is the fastest way to get started without maintaining model preferences.

[Explore automatic model selection](https://docs.fastrouter.ai/automatic-model-selection)

#### Cost-Optimized Routing with Sorting Suffixes

Append a suffix to any model to control provider selection:

```sh
# Route to the cheapest provider for this model
FASTROUTER_MODEL=openai/gpt-5.3-codex:price

# Route to the fastest provider
FASTROUTER_MODEL=openai/gpt-5.3-codex:throughput
```

[Configure cost and performance routing](https://docs.fastrouter.ai/provider-routing-strategies)

#### Flex Pricing

For batch-style agent tasks that tolerate higher latency, append `:flex` to access up to 50% lower token costs:

```sh
FASTROUTER_MODEL=openai/gpt-5.4-nano:flex
```

Flex routes your request to the provider's discounted inference tier. Same API key, same endpoint, same payload.

> **Note:** Flex is not recommended for interactive agent sessions where you need low-latency responses. Use it for large batch processing, bulk email triage, or scheduled background tasks.

[Compare flex pricing options](https://docs.fastrouter.ai/explore-features/flex-pricing)

**Change the connection.** Set `SCALEKIT_CONNECTION_NAME` to any connection configured in your Scalekit dashboard:

| Value    | What it connects                    |
| -------- | ----------------------------------- |
| `gmail`  | Gmail read/send                     |
| `github` | Repositories, issues, pull requests |
| `slack`  | Channels, messages, users           |

**Change the prompt.** Pass a prompt as a CLI argument to override the default:

```sh
npm start "List all GitHub pull requests assigned to me"
```

Or set `USER_PROMPT` in `.env` to change the default.

**Support multiple connections.** Call `listScopedTools` with multiple connection names to give the model tools from all of them at once:

```typescript
const { tools } = await scalekit.tools.listScopedTools('user_123', {
  filter: { connectionNames: ['gmail', 'github', 'slack'] },
});
```

**Use FastRouter-specific features.** Since FastRouter is the LLM provider, you can layer on any FastRouter capability alongside Scalekit tools. For example, add tags for cost tracking, use virtual model aliases to route across model pools, or configure fallback models for resilience.

***

### FAQs

#### Configuration & Setup

**Can I use multiple models with the same API key?**

Yes. The API key controls access and budget. The model is set via `FASTROUTER_MODEL`. You can switch models at any time without changing your key.

**Can I use my own provider keys (BYOK)?**

Yes. Set up an External Key integration in the FastRouter dashboard, then route agent traffic through it. You retain your provider's pricing and rate limits while gaining FastRouter's routing and observability layer.

**Can I restrict a key to only use specific models?**

Yes. When creating or editing a key, use the **Select Models** setting to limit which models the key can access. FastRouter rejects requests to unauthorized models.

#### Costs & Budgeting

**What happens when a key exceeds its budget?**

FastRouter blocks further requests until the budget resets (if a reset interval is configured) or an admin increases the limit. The agent receives an error response indicating the budget has been exceeded.

**How do I track agent spending by team or connection?**

Create separate projects for each team, issue project-scoped API keys, and use Dynamic Tags for finer-grained attribution. The dashboard breaks down costs by project, key, model, and tag.

#### Performance & Reliability

**Does FastRouter add latency to requests?**

FastRouter adds near-zero gateway overhead. For most workflows, this is negligible compared to model inference time.

#### Privacy & Security

**Does Scalekit see my users' OAuth tokens?**

Scalekit stores OAuth tokens server-side. Your agent never handles raw access tokens. Scalekit uses them to execute tools on behalf of connected accounts.

**Is my request content sent to FastRouter's servers?**

FastRouter acts as a pass-through gateway. Requests are routed to the model provider and responses are returned to your client. Content logging can be disabled per key for sensitive workloads — see the **Disable Content Logging** option in key settings.

***

### Next steps

* [Sample repository](https://github.com/scalekit-developers/fastrouter-scalekit-demo) for the full working code
* [Explore the full model catalog](https://fastrouter.ai/models) at FastRouter
* [Run a Free Audit](https://fastrouter.ai/audit) on your existing LLM traffic to identify savings
* [Join the FastRouter Discord community](https://discord.gg/QfTgEtMyyU)

***

