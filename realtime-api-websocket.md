---
description: >-
  Enable real-time speech-to-speech and text conversations with AI models via
  WebSocket.
icon: head-side-speak
---

# Realtime API (WebSocket)

## Overview

The FastRouter Realtime API enables low-latency, multimodal conversations over a persistent WebSocket connection. Unlike the standard chat completions API, where each request is a discrete HTTP round trip, the Realtime API maintains a stateful session — letting you stream audio and text to the model as it happens and receive responses incrementally, token by token and audio chunk by audio chunk.

This makes it the right choice for building voice agents, live transcription-and-response experiences, interactive customer support bots, and any application where conversational latency matters. The API supports speech-to-speech interactions natively, so you can send raw audio and get spoken responses back without stitching together separate STT, LLM, and TTS pipelines.

Because FastRouter exposes the Realtime API through a single gateway endpoint, you get unified billing, observability, and key management across realtime models — using the same API key as the rest of your FastRouter workloads. Sessions are event-driven: you send client events (like `session.update` or `input_audio_buffer.append`) and listen for server events (like `response.audio.delta`) over one connection.

**Key capabilities:**

* **Speech-to-speech** — stream microphone audio in, receive natural spoken audio out
* **Text and audio modalities** — mix and match input and output formats per response
* **Function calling** — let the model invoke your tools mid-conversation
* **Voice activity detection** — automatic turn detection, or disable it for push-to-talk
* **Streaming everything** — text deltas, audio chunks, and transcripts arrive in real time

### Endpoint

```
wss://go.fastrouter.ai/v1/realtime
```

### Connection

Connect to the WebSocket endpoint with your API key and model as query parameters:

```javascript
const url = new URL("wss://go.fastrouter.ai/v1/realtime");
url.searchParams.set("model", "openai/gpt-realtime-2.1");
url.searchParams.set("api_key", "sk-v1-...");

const ws = new WebSocket(url.toString());
```

#### Available Models

| Model                      | Description                              |
| -------------------------- | ---------------------------------------- |
| `openai/gpt-realtime-2.1`  | Latest GPT Realtime model (recommended)  |
| `openai/gpt-realtime-1.5`  | Previous-generation GPT Realtime model   |
| `openai/gpt-realtime-mini` | Smaller, lower-cost realtime model       |
| `openai/gpt-realtime`      | Alias for the current GPT Realtime model |

### Session Configuration

After the connection opens, configure the session by sending a `session.update` event:

```javascript
ws.onopen = () => {
  const sessionConfig = {
    type: "session.update",
    session: {
      modalities: ["text", "audio"],
      voice: "alloy",
      input_audio_format: "pcm16",
      output_audio_format: "pcm16",
      instructions: "You are a helpful assistant.",
      temperature: 0.8
    }
  };
  ws.send(JSON.stringify(sessionConfig));
};
```

#### Session Options

| Property              | Type       | Description                                                          |
| --------------------- | ---------- | -------------------------------------------------------------------- |
| `modalities`          | `string[]` | Output modalities: `["text"]`, `["audio"]`, or `["text", "audio"]`   |
| `voice`               | `string`   | Voice for audio: `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer` |
| `input_audio_format`  | `string`   | Input audio format: `pcm16`                                          |
| `output_audio_format` | `string`   | Output audio format: `pcm16`                                         |
| `instructions`        | `string`   | System instructions for the model                                    |
| `temperature`         | `number`   | Sampling temperature (0.6–1.2 recommended)                           |

### Sending Text Messages

Send text messages using `conversation.item.create` followed by `response.create`:

```javascript
// Create the conversation item
ws.send(JSON.stringify({
  type: "conversation.item.create",
  item: {
    type: "message",
    role: "user",
    content: [{
      type: "input_text",
      text: "Hello, how are you?"
    }]
  }
}));

// Request a response
ws.send(JSON.stringify({
  type: "response.create",
  response: { modalities: ["text", "audio"] }
}));
```

### Sending Audio (Streaming)

Stream audio input using `input_audio_buffer.append`, then commit and request a response:

```javascript
// Stream audio chunks (Base64-encoded PCM16)
ws.send(JSON.stringify({
  type: "input_audio_buffer.append",
  audio: base64AudioChunk
}));

// When finished recording, commit and request response
ws.send(JSON.stringify({ type: "input_audio_buffer.commit" }));
ws.send(JSON.stringify({
  type: "response.create",
  response: { modalities: ["text", "audio"] }
}));
```

#### Audio Format Requirements

* **Format:** 16-bit PCM (little-endian)
* **Sample Rate:** 24,000 Hz recommended
* **Channels:** Mono
* **Encoding:** Base64

#### Converting Audio to Base64 PCM16

```javascript
// Convert Float32 audio samples to PCM16 bytes
function float32ToPcm16Bytes(float32Array) {
  const bytes = new Uint8Array(float32Array.length * 2);
  const view = new DataView(bytes.buffer);
  for (let i = 0; i < float32Array.length; i++) {
    let s = Math.max(-1, Math.min(1, float32Array[i]));
    const val = s < 0 ? s * 0x8000 : s * 0x7fff;
    view.setInt16(i * 2, val, true);
  }
  return bytes;
}

// Convert bytes to Base64
function uint8ArrayToBase64(bytes) {
  let binary = "";
  const chunkSize = 0x8000;
  for (let i = 0; i < bytes.length; i += chunkSize) {
    binary += String.fromCharCode.apply(null, bytes.subarray(i, i + chunkSize));
  }
  return btoa(binary);
}
```

### Receiving Responses

Handle incoming events with an `onmessage` handler:

```javascript
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);

  switch (data.type) {
    case "session.created":
      console.log("Session created");
      break;

    case "response.text.delta":
      // Streaming text chunk
      console.log("Text:", data.delta);
      break;

    case "response.audio.delta":
      // Base64-encoded PCM16 audio chunk
      playAudioChunk(data.delta);
      break;

    case "response.audio_transcript.delta":
      // Transcript of audio response
      console.log("Transcript:", data.delta);
      break;

    case "response.done":
      console.log("Response complete", data.response?.usage);
      break;

    case "error":
      console.error("Error:", data.error?.message);
      break;
  }
};
```

#### Playing Audio Output

```javascript
let audioContext = null;
let nextPlayTime = 0;

function playAudioChunk(base64Audio) {
  if (!audioContext) {
    audioContext = new AudioContext({ sampleRate: 24000 });
    nextPlayTime = audioContext.currentTime;
  }

  // Decode Base64 to PCM16 bytes
  const bytes = base64ToUint8Array(base64Audio);
  const samples = pcm16BytesToFloat32(bytes);

  // Create and play audio buffer
  const buffer = audioContext.createBuffer(1, samples.length, 24000);
  buffer.getChannelData(0).set(samples);

  const source = audioContext.createBufferSource();
  source.buffer = buffer;
  source.connect(audioContext.destination);

  const startTime = Math.max(nextPlayTime, audioContext.currentTime + 0.02);
  source.start(startTime);
  nextPlayTime = startTime + buffer.duration;
}

function base64ToUint8Array(base64) {
  const binary = atob(base64);
  const bytes = new Uint8Array(binary.length);
  for (let i = 0; i < binary.length; i++) {
    bytes[i] = binary.charCodeAt(i);
  }
  return bytes;
}

function pcm16BytesToFloat32(bytes) {
  const view = new DataView(bytes.buffer, bytes.byteOffset, bytes.byteLength);
  const samples = new Float32Array(bytes.byteLength / 2);
  for (let i = 0; i < samples.length; i++) {
    samples[i] = view.getInt16(i * 2, true) / 0x8000;
  }
  return samples;
}
```

### Event Reference

#### Client Events (Send)

| Event                       | Description                         |
| --------------------------- | ----------------------------------- |
| `session.update`            | Configure session settings          |
| `conversation.item.create`  | Add a message to the conversation   |
| `input_audio_buffer.append` | Stream audio input chunks           |
| `input_audio_buffer.commit` | Commit buffered audio as user input |
| `input_audio_buffer.clear`  | Clear the audio input buffer        |
| `response.create`           | Request a model response            |
| `response.cancel`           | Cancel an in-progress response      |

#### Server Events (Receive)

| Event                             | Description                             |
| --------------------------------- | --------------------------------------- |
| `session.created`                 | Session initialized                     |
| `session.updated`                 | Session configuration applied           |
| `response.created`                | Response generation started             |
| `response.text.delta`             | Streaming text chunk                    |
| `response.text.done`              | Text response complete                  |
| `response.audio.delta`            | Streaming audio chunk (Base64 PCM16)    |
| `response.audio.done`             | Audio response complete                 |
| `response.audio_transcript.delta` | Streaming transcript of audio           |
| `response.done`                   | Full response complete (includes usage) |
| `error`                           | Error occurred                          |

### Voice Activity Detection (VAD)

By default, VAD is enabled and the API automatically detects when users start and stop speaking. For push-to-talk interfaces, disable VAD:

```javascript
ws.send(JSON.stringify({
  type: "session.update",
  session: {
    turn_detection: null  // Disable VAD
  }
}));
```

> **Note:** With VAD disabled, you must manually call `input_audio_buffer.commit` to finalize audio input, `response.create` to trigger a response, and `input_audio_buffer.clear` before starting new input.

### Function Calling

Define functions the model can call:

```javascript
ws.send(JSON.stringify({
  type: "session.update",
  session: {
    tools: [{
      type: "function",
      name: "get_weather",
      description: "Get current weather for a location",
      parameters: {
        type: "object",
        properties: {
          location: {
            type: "string",
            description: "City name"
          }
        },
        required: ["location"]
      }
    }],
    tool_choice: "auto"
  }
}));
```

When the model calls a function, handle it and provide results:

```javascript
// In your message handler, check for function calls
if (data.type === "response.done") {
  const output = data.response?.output?.[0];
  if (output?.type === "function_call") {
    const args = JSON.parse(output.arguments);
    const result = await yourFunction(args);

    // Send function result back
    ws.send(JSON.stringify({
      type: "conversation.item.create",
      item: {
        type: "function_call_output",
        call_id: output.call_id,
        output: JSON.stringify(result)
      }
    }));

    // Request continuation
    ws.send(JSON.stringify({ type: "response.create" }));
  }
}
```

### Complete Example

```javascript
const API_KEY = "sk-v1-...";
const MODEL = "openai/gpt-realtime-2.1";

// Connect
const url = new URL("wss://go.fastrouter.ai/v1/realtime");
url.searchParams.set("model", MODEL);
url.searchParams.set("api_key", API_KEY);

const ws = new WebSocket(url.toString());

ws.onopen = () => {
  // Configure session
  ws.send(JSON.stringify({
    type: "session.update",
    session: {
      modalities: ["text", "audio"],
      voice: "alloy",
      input_audio_format: "pcm16",
      output_audio_format: "pcm16",
      instructions: "You are a helpful assistant. Be concise.",
      temperature: 0.8
    }
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);

  switch (data.type) {
    case "session.created":
    case "session.updated":
      console.log("Ready");
      break;
    case "response.text.delta":
      process.stdout.write(data.delta);
      break;
    case "response.audio.delta":
      // Handle audio playback
      break;
    case "response.done":
      console.log("\n[Done]");
      break;
    case "error":
      console.error("Error:", data.error?.message);
      break;
  }
};

ws.onerror = (error) => {
  console.error("WebSocket error:", error);
};

ws.onclose = () => {
  console.log("Disconnected");
};

// Send a text message
function sendMessage(text) {
  ws.send(JSON.stringify({
    type: "conversation.item.create",
    item: {
      type: "message",
      role: "user",
      content: [{ type: "input_text", text }]
    }
  }));
  ws.send(JSON.stringify({
    type: "response.create",
    response: { modalities: ["text"] }
  }));
}
```
