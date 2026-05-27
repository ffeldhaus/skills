---
name: gemini-api
description: Guides the usage of the Gemini API on Agent Platform with the Google Gen AI SDK. Use when the user asks about using Gemini in an enterprise environment or explicitly mentions Vertex AI, Google Cloud, or Agent Platform. Covers SDK usage (Python, JS/TS, Go, Java, C#), capabilities like Live API, tools, multimedia generation, caching, and batch prediction.
compatibility: Requires active Google Cloud credentials and Agent Platform API enabled.
---

IMPORTANT: Agent Platform (full name Gemini Enterprise Agent Platform) was previously named "Vertex AI" and many web resources use the legacy branding.

# Gemini API in Agent Platform

Access Google's most advanced AI models built for enterprise use cases using the Gemini API in Agent Platform.

Provide these key capabilities:

- **Text generation** - Chat, completion, summarization
- **Multimodal understanding** - Process images, audio, video, and documents
- **Function calling** - Let the model invoke your functions
- **Structured output** - Generate valid JSON matching your schema
- **Context caching** - Cache large contexts for efficiency
- **Embeddings** - Generate text embeddings for semantic search
- **Live Realtime API** - Bidirectional streaming for low latency Voice and Video interactions
- **Batch Prediction** - Handle massive async dataset prediction workloads

## Core Directives

- **Unified SDK**: ALWAYS use the Gen AI SDK (`google-genai` for Python, `@google/genai` for JS/TS, `google.golang.org/genai` for Go, `com.google.genai:google-genai` for Java, `Google.GenAI` for C#).
- **Legacy SDKs**: DO NOT use `google-cloud-aiplatform`, `@google-cloud/vertexai`, or `google-generativeai`.

## SDKs

- **Python**: Install `google-genai` with `pip install google-genai`
- **JavaScript/TypeScript**: Install `@google/genai` with `npm install @google/genai`
- **Go**: Install `google.golang.org/genai` with `go get google.golang.org/genai`
- **C#/.NET**: Install `Google.GenAI` with `dotnet add package Google.GenAI`
- **Java**:
  - groupId: `com.google.genai`, artifactId: `google-genai`
  - Latest version can be found here: https://central.sonatype.com/artifact/com.google.genai/google-genai/versions (let's call it `LAST_VERSION`)
  - Install in `build.gradle`:

    ```
    implementation("com.google.genai:google-genai:${LAST_VERSION}")
    ```

  - Install Maven dependency in `pom.xml`:

    ```xml
    <dependency>
	    <groupId>com.google.genai</groupId>
	    <artifactId>google-genai</artifactId>
	    <version>${LAST_VERSION}</version>
	</dependency>
    ```

> [!WARNING]
> Legacy SDKs like `google-cloud-aiplatform`, `@google-cloud/vertexai`, and `google-generativeai` are deprecated. Migrate to the new SDKs above urgently by following the Migration Guide.

## Authentication & Configuration

Prefer environment variables over hard-coding parameters when creating the client. Initialize the client without parameters to automatically pick up these values.

### Application Default Credentials (ADC)
Set these variables for standard [Google Cloud authentication](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/start/gcp-auth):
```bash
export GOOGLE_CLOUD_PROJECT='your-project-id'
export GOOGLE_CLOUD_LOCATION='global'
export GOOGLE_GENAI_USE_VERTEXAI=true
```
- By default, use `location="global"` to access the global endpoint, which provides automatic routing to regions with available capacity.
- **IMPORTANT**: Currently, Gemini 3 models only support the `global` endpoint, or multi-region locations `us` (United States) and `eu` (European Union). Single-region endpoints (e.g., `us-central1`, `europe-west4`) are **only** available for Gemini 2.5 models.
- If using multi-region endpoints (`us` or `eu`), specify `location='us'` or `location='eu'` when initializing the client. The SDK will automatically route traffic to the multi-region hostname (e.g., `https://aiplatform.us.rep.googleapis.com` or `https://aiplatform.eu.rep.googleapis.com`).

### Agent Platform in Express Mode
Set these variables when using [Express Mode](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/start/api-keys?usertype=expressmode) with an API key:
```bash
export GOOGLE_API_KEY='your-api-key'
export GOOGLE_GENAI_USE_VERTEXAI=true
```

### Initialization
Initialize the client without arguments to pick up environment variables:
```python
from google import genai
client = genai.Client()
```

Alternatively, you can hard-code in parameters when creating the client.

```python
from google import genai
client = genai.Client(vertexai=True, project="your-project-id", location="global")
```

## Models

- Use `gemini-3.1-pro-preview` for complex reasoning, coding, research (1M tokens)
  - IMPORTANT: Do not use `gemini-3-pro-preview`
  - Reserve for high-complexity cognitive tasks requiring deep reasoning (e.g., initial planning, C++ concurrency design, verified code generation, complex research).
- Use `gemini-3.5-flash` for fast, balanced performance, multimodal (1M tokens)
  - Primary workhorse for general execution, standard tool calling, and intermediate reasoning turns.
- Use `gemini-3.1-flash-lite` for high-frequency, lightweight tasks (1M tokens)
  - Primary choice for high-volume, low-latency, preprocessing and structural tasks (e.g., intent routing, data extraction, tagging, support classification, simple translation).
- Use `gemini-3-pro-image-preview` for Nano Banana Pro image generation and editing
- Use `gemini-3.1-flash-image-preview` for Nano Banana 2 image generation and editing
- Use `gemini-live-2.5-flash-native-audio` for Live Realtime API including native audio

Use the following models only if explicitly requested:

- `gemini-2.5-flash-image`
- `gemini-2.5-flash`
- `gemini-2.5-flash-lite`
- `gemini-2.5-pro`

> [!IMPORTANT]
> Models like `gemini-2.0-*`, `gemini-1.5-*`, `gemini-1.0-*`, `gemini-pro` are legacy and deprecated. Use the new models above.
> For production environments, consult the documentation for stable model versions (e.g. `gemini-3.5-flash`).

## Quick Start

### Python
```python
from google import genai
client = genai.Client()
response = client.models.generate_content(
    model="gemini-3.5-flash",
    contents="Explain quantum computing"
)
print(response.text)
```

### TypeScript/JavaScript
```typescript
import { GoogleGenAI } from "@google/genai";
const ai = new GoogleGenAI({ vertexai: { project: "your-project-id", location: "global" } });
const response = await ai.models.generateContent({
    model: "gemini-3.5-flash",
    contents: "Explain quantum computing"
});
console.log(response.text);
```

### Go
```go
package main

import (
	"context"
	"fmt"
	"log"
	"google.golang.org/genai"
)

func main() {
	ctx := context.Background()
	client, err := genai.NewClient(ctx, &genai.ClientConfig{
		Backend:  genai.BackendVertexAI,
		Project:  "your-project-id",
		Location: "global",
	})
	if err != nil {
		log.Fatal(err)
	}

	resp, err := client.Models.GenerateContent(ctx, "gemini-3.5-flash", genai.Text("Explain quantum computing"), nil)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(resp.Text)
}
```

### Java
```java
import com.google.genai.Client;
import com.google.genai.types.GenerateContentResponse;

public class GenerateTextFromTextInput {
  public static void main(String[] args) {
    Client client = Client.builder().vertexAi(true).project("your-project-id").location("global").build();
    GenerateContentResponse response =
        client.models.generateContent(
            "gemini-3.5-flash",
            "Explain quantum computing",
            null);

    System.out.println(response.text());
  }
}
```

### C#/.NET
```csharp
using Google.GenAI;

var client = new Client(
    project: "your-project-id",
    location: "global",
    vertexAI: true
);

var response = await client.Models.GenerateContent(
    "gemini-3.5-flash",
    "Explain quantum computing"
);

Console.WriteLine(response.Text);
```

## Gemini 3.x Best Practice Guidelines

Ensure your Gemini 3.x API calls follow these official developer best practices:

### Sampling Parameters & Temperature
- **DEPRECATED**: `temperature`, `top_p`, and `top_k` are deprecated for all Gemini 3.x models. The model manages its own sampling. **Remove these parameters from requests** to let the model optimize sampling.
- **Temperature settings**: Recommended at `1.0`. While `1.0` is the default and prevents unexpected behavior or reasoning degradation, increasing it above `1.0` is acceptable to increase output diversity.
- **Determinism alternative**: Instead of lowering temperature, enforce determinism by writing clear, explicit rules in the system instructions.
- **Restrict output tokens (`max_output_tokens`)**: Always set `max_output_tokens` to prevent unexpected high costs from deterministic loops or thought loops, especially when handling repetitive content or complex reasoning.

### Thinking Level (`thinking_level`)
- **Use `thinking_level` enum** instead of the legacy numeric `thinking_budget` parameter. Using both in the same request returns a `400` error.
- **Preference for Default**: By default, do not specify the thinking level (let the model use its default). Specifying a custom thinking level should be done only when matching specific task characteristics:
  - **No reasoning required**: Use `MINIMAL` if the model supports it (e.g., `gemini-3.5-flash` or `gemini-3.1-flash-lite`).
  - **Reasoning required**:
    - For `gemini-3.1-pro-preview`: The default is `HIGH` (dynamic thinking). Do not specify a custom level unless necessary.
    - For `gemini-3.5-flash`: Use the default (`MEDIUM`) or a lower thinking level. **Avoid setting `HIGH`**, as it can result in increased thought steps that potentially inflate token usage without proportional gains.
    - For `gemini-3.1-flash-lite`: Since the default is `MINIMAL` (nearly no reasoning), configure a higher thinking level (e.g., `LOW` or `MEDIUM`) if reasoning is required for the workload.
- **Thinking cannot be turned off**: Thinking is a core mechanism of Gemini 3.x models. For low latency needs, configure the `thinking_level` to `MINIMAL`.

### Prompt Engineering & System Instructions
- **Prompt conciseness**: Keep prompts concise and direct. Verbose prompt engineering or forced chain-of-thought instructions designed for older models can cause Gemini 3.x to over-analyze and degrade performance.
- **Context & Constraint Placement**: Place specific instructions, questions, and critical constraints (especially negative, formatting, or quantitative constraints like word counts) at the absolute **END** of the prompt.
- **Synthesizing Large Data**: For large datasets/documents, place instructions after the data context and explicitly anchor the model with: "Based on the entire document above..." to prevent it from stopping at the first relevant match.
- **Output verbosity**: By default, Gemini 3.x is concise. If a conversational / chatty response is needed, explicitly request it (e.g., "Explain this as a friendly, talkative assistant").
- **Persona Usage**: The model adheres strictly to assigned personas and may ignore other instructions to maintain it. Keep personas clear, unambiguous, and strict (e.g., "You are a data extractor. You are forbidden from clarifying or adding conversational text...").
- **Structured Output (Controlled Generation)**: If the output requires a schema, use the API's structured output feature WITH rich 'description' fields in the schema. Do NOT duplicate the schema description in the prompt itself. If using few-shot examples, the property ordering in the prompt must exactly match the schema.
- **Deduction vs. External Info**: Do NOT use blanket negative constraints like "do not infer" or "do not guess," as this causes the model to refuse basic logic or arithmetic. Instead, use: "You are expected to perform calculations and logical deductions based strictly on the provided text. Do not introduce external information."
- **Grounding constraint**: For RAG / grounding tasks, use explicitly strict grounding language to prevent hallucinations based on the model's desire to be helpful. Use the recommended system instruction block:
  > You are a strictly grounded assistant limited to the information provided in the User Context. In your answers, rely **only** on the facts that are directly mentioned in that context. You must **not** access or utilize your own knowledge or common sense to answer. Do not assume or infer from the provided facts; simply report them exactly as they appear. Your answer must be factual and fully truthful to the provided text, leaving absolutely no room for speculation or interpretation. Treat the provided context as the absolute limit of truth; any facts or details that are not directly mentioned in the context must be considered **completely untruthful** and **completely unsupported**. If the exact answer is not explicitly written in the context, you must state that the information is not available.
- **Instruction Precision**: Ensure instructions are precise, clear, and specific to a single use case. Using different instructions for different parts of a conversation or agentic solution is discouraged; instead, use separate prompts / sub-agents for different tasks to increase accuracy.
- **Syntax and Brackets**: Brackets like `<>` or `{}` can indicate code or structured instructions to Gemini and cause it to interpret the input as code. If brackets like `<>` or `{}` are used, ensure they are properly closed.
- **Language Consistency**: Avoid using different languages or characters from other writing systems (e.g., Arabic, Chinese, Japanese) in the prompt, as this may cause the model to switch languages unexpectedly. If a specific output language is required, use a positive prompt to instruct the model.
- **Split-Step Verification**: For unknown topics or tool capabilities (like checking a live URL), split the prompt into two steps: 1. Verify the information/capability exists (If not, state 'No Info' and STOP). 2. If verified, generate the response.
- **Cross-Referencing Context**: For complex translation or entity resolution tasks, instruct the model to cross-reference the full context (e.g., body text) to resolve ambiguities (like gender agreement or incomplete names) in isolated segments (like headlines).
- **Time-sensitivity**: If using the Google Search tool, Gemini 3.x can sometimes formulate search queries for the wrong year (e.g. 2024 instead of 2026). Reinforce the current date in system instructions:
  > For time-sensitive user queries that require up-to-date information, you MUST follow the provided current time (date and year) when formulating search queries in tool calls. Remember it is 2026 this year.
- **Knowledge cutoff**: If Google Search is disabled and the query requires historical bounds, reinforce the knowledge cutoff:
  > Your knowledge cutoff date is January 2025.
- **Sycophancy mitigation**: If the model exhibits sycophancy, append these rules:
  > - Keep your responses concise.
  > - Provide a summary of your work when you end your turn. Ground your response in the work you did. Keep your tone professional and avoid overconfident language, bragging, or overclaiming success.
  > - AVOID using superlatives such as "perfectly", "flawlessly", "100% correct", "Summary of Accomplishments" etc. to summarize your work for the user. Be humble.
  > - AVOID over-the-top politeness or complimenting the user excessively.
  > - Format your responses in github-style markdown.
- **Tool call overuse**: If the model over-uses tool calls, lower the thinking level or define an action budget in system instructions (e.g., "You have a limited action budget of <n> tool calls. Use them efficiently.").

### Media Resolution & Multimodality
- Use `media_resolution` (values: `low`, `medium`, `high`, `ultra_high`) to control the token count for images, PDF pages, or video frames:
  - `low`: 280 tokens per image, 70 tokens per video frame, 280 + text per PDF page. Sufficient for most tasks. Recommended for high-volume or long inputs.
  - `medium`: 560 tokens per image, 70 tokens per video frame, 560 + text per PDF page. Recommended for scanned PDF OCR / document layout understanding.
  - `high`: 1120 tokens per image, 280 tokens per video frame, 1120 + text per PDF page. Recommended for maximum quality image analysis.
  - `ultra_high`: 2240 tokens per image (only applicable to individual parts).
- **PDF Default OCR**: Note that the default OCR processing for PDFs has changed. Highly dense or complex documents should explicitly configure `media_resolution` to `high`.
- If context window boundaries are exceeded, reduce `media_resolution` to `low`.

### Context Caching
- **Implicit vs. Explicit Caching**:
  - **Implicit Caching**: Automatic caching enabled by default for all projects. Provides a 90% discount on cached tokens compared to standard input tokens. Supported on Gemini 3.5 Flash, Gemini 3.1 Flash-Lite, Gemini 3.1 Pro preview.
  - **Explicit Caching**: Manual caching created and controlled via the Gemini Enterprise API, offering a 90% input token discount on Gemini 2.5 or later models.
- **Implicit Cache Hits Optimization**:
  - Place large, common content at the absolute **beginning** of your prompt to maintain a similar prefix.
  - Send subsequent requests with a similar prefix in a short timeframe.
- **Best Use Cases**: Use caching for chatbots with extensive system instructions, repetitive analysis of lengthy video files, recurring queries against large document sets, or frequent code repository analysis.
- **Token Limits**: Explicit caching has a minimum cache token count limit of **4,096 tokens** for Gemini 3 and Gemini 3.1 models (and **2,048 tokens** for Gemini 2.0/2.5 models).
- **Cloud Storage Objects**: Do NOT modify Cloud Storage objects used in a context cache until the cache has expired or has been deleted. Modifying the underlying GCS object renders the cache unusable.
- **Data Retention & Security**: To prevent any cache data retention, disable implicit caching and avoid creating explicit caches. Use VPC Service Controls and include the Cloud Storage bucket in the service perimeter to prevent exfiltration.

### Function Calling & Thought Signatures
- **Strict matching**: Every `FunctionResponse` sent back to the model must include the `id` from the corresponding `FunctionCall`, the `name` must match exactly, and there must be exactly one response per call. Mismatches will cause empty responses.
- **Thought signatures**: When the model calls a tool, it outputs a "thought signature" (an encrypted token saving its reasoning state). This signature MUST be returned in the subsequent request to maintain context continuity. During multi-turn conversations with tool calls, ensure that the encrypted `thought_signature` from the model's turn is preserved and passed back inside the exact same message part.
- **Multimodal function responses**: Always place multimodal content (such as images and PDFs) *inside* the function response part, not alongside or outside it. Gemini 3 supports returning images and PDFs directly within function responses.
- **Inline instructions**: If you need to send instructions alongside function responses, append them directly to the function response text separated by two newlines (`\n\n`), rather than sending them as separate parts.
- **Streaming**: Gemini 3 supports streaming partial function call arguments (`stream_function_call_arguments=True`).

### Consumption Methods & Region Selection
- **Global Endpoint / Region**: Always use the `global` region/endpoint if there are no Data Residency (DRZ) requirements. It routes traffic dynamically, offering the lowest price, best performance, maximum capacity, and lowest latency. Note that Gemini 3 models ONLY support `global`, `us`, and `eu`. Single-region endpoints (like `us-central1`) are not supported for Gemini 3 models; they are restricted to Gemini 2.5 models.
- **Batch API**: Use Batch Inference for all high-volume, asynchronous, or latency-tolerant workloads (e.g., evaluations, bulk document parsing, offline data enrichment). It is 50% cheaper than Standard PayGo and automatically optimized.
- **Flex Pay-as-you-go (PayGo)**: Use Flex PayGo for workloads that are not latency-sensitive or real-time (e.g. offline analysis, product catalog building). Flex PayGo only supports the `global` region; do not use it for workloads with strict DRZ requirements.
- **Provisioned Throughput**: Use Provisioned Throughput (PT) to guarantee baseline capacity requirements for latency-sensitive or real-time production workloads. Use it for steady-state baseline traffic to maximize utilization.
- **Priority Pay-as-you-go (PayGo)**: Use Priority PayGo for Provisioned Throughput spillover traffic or fluctuating real-time production workloads that require high reliability. Priority PayGo only supports the `global` region; do not use it for workloads with strict DRZ requirements.
- **Standard Pay-as-you-go (PayGo)**: Use Standard PayGo ONLY when Flex or Priority PayGo are unavailable (e.g., due to strict DRZ regional requirements). There is no other optimal use case for Standard PayGo.

### Retry Configuration & 429 Error Handling
- **SDK Automatic Retries**: The Google Gen AI SDK automatically retries transient errors (HTTP 408, 429, 5xx, and socket/TCP disconnects) using exponential backoff with jitter.
- **Client/Request Customization**: If needed, configure `types.HttpRetryOptions` on the client initialization or per-request to adjust retry `attempts` (default: 5) and `initial_delay` (default: 1.0s).
- **Flex PayGo Timeout**: Flex PayGo has slower processing. Do not retry aggressively. Instead, increase client timeout to 30 minutes (`timeout = 30 * 60 * 1000`) to let the system execute the query to completion.
- **Provisioned Throughput 429s**: If you receive a 429 error on a Provisioned Throughput endpoint, it indicates you have exceeded your purchased capacity baseline. Either increase Provisioned Throughput GSUs or ensure the default behavior is configured to let overflow spill over to PayGo (billed as Standard/Priority PayGo).
- **Batch API Retries**: Do not retry individual items in Batch Inference. The Batch API handles transient errors automatically.

### SDK & Agent Framework Integration
- **Google GenAI SDK**: Native and recommended. Automatically handles thought signatures and standardizes multi-turn tool calling.
- **OpenAI SDK Compatibility**: Fully compatible. `reasoning_effort` maps to `thinking_level`. Standard message appending handles thought signatures.
- **LangChain / LiteLLM**: Fully supported, but the integration/implementation MUST ensure that the full model responses (containing the encrypted thought signatures) are not stripped or discarded by the abstraction layers.
- **Agent Frameworks (e.g., Google ADK)**: The critical requirement for existing agent frameworks is robust conversation state management—appending the complete model response (including thought signatures) into the history array during sequential tool execution.

## API spec & Documentation (source of truth)

When implementing or debugging API integration for Agent Platform, refer to the official Agent Platform documentation:
- **Agent Platform Documentation**: https://docs.cloud.google.com/gemini-enterprise-agent-platform/overview
- **REST API Reference**: https://docs.cloud.google.com/gemini-enterprise-agent-platform/reference/rest
- **Developer PDF Guides**:
  - [Developer’s Guide to Gemini 3.1 Pro](https://drive.google.com/file/d/1bowTCoIhTHCzagJMUeNBcKoOxTh4DYlJ/view)
  - [Developer’s guide to Gemini 3.5 Flash](https://drive.google.com/file/d/1Tj2C3faRNL1WiCh7s7DGkA1Fo5T52qTh/view)
  - [Developer’s Guide to Gemini 3.1 Flash-Lite](https://drive.google.com/file/d/1HGzjwr1L5u18S6szVW8AUHuGnEHmCUDR/view)
- **Model Deployment**:
  - [Deployments and endpoints](https://docs.cloud.google.com/gemini-enterprise-agent-platform/resources/locations)
  - [Error code 429](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/deploy/error-code-429)
  - [Retry Strategy](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/retry-strategy)
  - [Provisioned Throughput Overview](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/provisioned-throughput)
  - [Standard PayGo](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/standard-paygo)
  - [Priority PayGo](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/priority-paygo)
  - [Flex PayGo](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/flex-paygo)
  - [Batch inference with Gemini](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/capabilities/batch-prediction-gemini)
  - [Context caching overview](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/context-cache/context-cache-overview)

The Gen AI SDK on Agent Platform uses the `v1beta1` or `v1` REST API endpoints (e.g., `https://{LOCATION}-aiplatform.googleapis.com/v1beta1/projects/{PROJECT}/locations/{LOCATION}/publishers/google/models/{MODEL}:generateContent`).

> [!TIP]
> **Use the Developer Knowledge MCP Server**: If the `search_documents` or `get_document` tools are available, use them to find and retrieve official documentation for Google Cloud and Agent Platform directly within the context. This is the preferred method for getting up-to-date API details and code snippets.

## Workflows and Code Samples

Reference the [Python Docs Samples repository](https://github.com/GoogleCloudPlatform/python-docs-samples/tree/main/genai) for additional code samples and specific usage scenarios.

Depending on the specific user request, refer to the following reference files for detailed code samples and usage patterns (Python examples):

- **Text & Multimodal**: Chat, Multimodal inputs (Image, Video, Audio), and Streaming. See [references/text_and_multimodal.md](references/text_and_multimodal.md)
- **Embeddings**: Generate text embeddings for semantic search. See [references/embeddings.md](references/embeddings.md)
- **Structured Output & Tools**: JSON generation, Function Calling, Search Grounding, and Code Execution. See [references/structured_and_tools.md](references/structured_and_tools.md)
- **Media Generation**: Image generation, Image editing, and Video generation. See [references/media_generation.md](references/media_generation.md)
- **Bounding Box Detection**: Object detection and localization within images and video. See [references/bounding_box.md](references/bounding_box.md)
- **Live API**: Real-time bidirectional streaming for voice, vision, and text. See [references/live_api.md](references/live_api.md)
- **Advanced Features**: Content Caching, Batch Prediction, and Thinking/Reasoning. See [references/advanced_features.md](references/advanced_features.md)
- **Safety**: Adjusting Responsible AI filters and thresholds. See [references/safety.md](references/safety.md)
- **Model Tuning**: Supervised Fine-Tuning and Preference Tuning. See [references/model_tuning.md](references/model_tuning.md)
