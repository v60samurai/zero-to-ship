# LLM APIs: Zero to Ship

> You send text to a computer in the cloud, it sends text back, and you pay a fraction of a cent each time.

## Why Should You Care?

Every AI-powered feature you've ever used (ChatGPT, Claude, GitHub Copilot, the AI thing in Notion) is built on the same foundation: an API call. Your app sends text to a model hosted by someone else, the model generates a response, and your app displays it. That's it. The magic is in the model, but the plumbing is just HTTP requests.

If you're building anything with AI (a chatbot, a document summarizer, an email classifier, a code reviewer), you need to understand how these APIs work. Not "I pasted an API key into Cursor and it worked." Actually understand: what am I sending, what am I getting back, why is it slow sometimes, why did my bill spike, why does the model forget what I said three messages ago.

The difference between an AI prototype and a production AI feature is almost entirely in how you handle the API: streaming responses so users don't stare at a spinner, managing conversation history without blowing past context limits, choosing the right model for each task so you're not paying $50/day for something a $2/day model handles fine, and handling errors gracefully when the API inevitably goes down at 2 AM on a Saturday.

## The 30-Second Version

LLM APIs are HTTP endpoints. You send a JSON request containing messages (system instructions + conversation history + the user's latest message), and you get back a JSON response containing the model's generated text. You pay per token, which is roughly 3/4 of a word. The model is stateless: it has no memory between requests, so you must send the full conversation history every time. Streaming lets you show the response word-by-word instead of waiting for the whole thing. Function calling lets the model trigger your code to fetch data or perform actions.

## The Real Explanation (ELI5)

Imagine you're texting a really smart friend who has a peculiar condition: every time you send a message, they lose all memory of your previous conversation. They start completely fresh. The only way they can follow a conversation is if you copy-paste the entire conversation history before your new message, every single time.

So if you're three messages deep, your next text actually looks like:

"Hey, here's everything we've discussed so far:
ME: What's a good restaurant in Bangalore?
YOU: Try Vidyarthi Bhavan for dosas.
ME: What about something non-veg?
YOU: Meghana Foods is great for biryani.
ME: Cool, what's the price range at Meghana?"

Your friend reads that entire wall of text, then responds to the latest question. They respond really fast (a few seconds), and for every character they read and write, you owe them a tiny amount of money. Read a lot of characters? Costs more. Get a long response? Costs more.

Now replace "your friend" with an AI model running on someone else's servers. Replace "text messages" with HTTP requests. Replace "characters" with tokens (chunks of text, roughly 3/4 of a word). Replace "tiny amount of money" with actual tiny amounts of money (like $0.003 per 1,000 tokens for cheap models, $0.075 per 1,000 tokens for expensive ones).

That's an LLM API. You send text, you get text, you pay per token, and the model has no memory, so you send the whole conversation every time.

The analogy breaks in one important way: unlike your friend, you can also give the model a secret instruction sheet before the conversation starts (the system prompt). Your friend reads it first, follows those rules, but the user never sees it. That's how you control the model's behavior.

## How It Actually Works

### The Request-Response Flow

```
Your App                          LLM API (OpenAI/Anthropic/etc.)
   │                                          │
   │  POST /v1/messages                       │
   │  {                                       │
   │    model: "claude-sonnet-4-20250514",    │
   │    system: "You are a helpful...",       │
   │    messages: [                           │
   │      {role: "user", content: "Hi"},      │
   │      {role: "assistant", content: "Hey"},│
   │      {role: "user", content: "Question"} │
   │    ],                                    │
   │    max_tokens: 1024                      │
   │  }                                       │
   │─────────────────────────────────────────▶│
   │                                          │  Model processes tokens
   │                                          │  Generates response
   │  200 OK                                  │
   │  {                                       │
   │    content: [{text: "Answer..."}],       │
   │    usage: {                              │
   │      input_tokens: 42,                   │
   │      output_tokens: 128                  │
   │    }                                     │
   │  }                                       │
   │◀─────────────────────────────────────────│
   │                                          │
```

**What happens when you make an API call:** Your app sends an HTTP POST with a JSON body containing the model name, messages, and parameters. The API validates your key, checks your rate limits, feeds your text to the model, and returns the generated response plus metadata (how many tokens were used).

**Under the hood:** The model processes your entire input as a sequence of tokens. It generates the response one token at a time, where each new token is influenced by every token before it (your input + the tokens it has already generated). This is why longer conversations cost more: the model must process the full history to generate each new token. And this is why models have context limits: there's a maximum number of tokens they can process at once.

**Why it's designed this way:** Statelessness (no memory between requests) might seem annoying, but it makes the system massively scalable. Any server can handle any request from any user without needing to track sessions or state. It's the same reason HTTP is stateless: it's simpler and scales better than the alternative.

### Tokens: The Currency of LLM APIs

A token is not a word. It's a chunk of text, usually 3-4 characters. The model doesn't see characters or words. It sees tokens.

```
"Hello, how are you?"
 → ["Hello", ",", " how", " are", " you", "?"]
 → 6 tokens

"Pneumonoultramicroscopicsilicovolcanoconiosis"
 → ["Pne", "um", "ono", "ultra", "micro", "scop", "ics", "ili", "cov", "olcan", "ocon", "iosis"]
 → 12 tokens

"こんにちは"
 → ["こん", "にち", "は"]
 → 3 tokens (non-English text often uses more tokens per word)
```

**Why tokens matter for cost:**

Every API charges per token, separately for input (what you send) and output (what the model generates). Output tokens typically cost 3-5x more than input tokens.

```
Example costs (approximate, mid-2025):

Claude Haiku:    $0.25 / million input tokens,  $1.25 / million output tokens
Claude Sonnet:   $3.00 / million input tokens,  $15.00 / million output tokens
Claude Opus:     $15.00 / million input tokens,  $75.00 / million output tokens
GPT-4o:          $2.50 / million input tokens,  $10.00 / million output tokens
GPT-4o mini:     $0.15 / million input tokens,  $0.60 / million output tokens
```

A 10-message conversation where each message is ~100 tokens means you're sending ~1,000 input tokens per request (the full history). If your user has 50 such conversations a day on Sonnet, that's about $2.25/day in input alone. This adds up.

**Why tokens matter for context limits:**

Every model has a context window: the maximum total tokens (input + output) it can handle.

```
GPT-4o:           128K tokens (~96K words, ~300 pages)
Claude Sonnet:    200K tokens (~150K words, ~500 pages)
GPT-4o mini:      128K tokens
Claude Haiku:     200K tokens
```

Hit the limit and the API returns an error. Before that, you'll notice the model struggling to track information from early in the conversation, because attention degrades over very long contexts.

### The Messages Format

Every major LLM API uses the same basic structure: an array of messages with roles.

**Anthropic (Claude):**

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system="You are a helpful coding assistant.",  # system is separate
    messages=[
        {"role": "user", "content": "What is a closure in JavaScript?"},
        {"role": "assistant", "content": "A closure is a function that..."},
        {"role": "user", "content": "Can you show me an example?"}
    ]
)

print(response.content[0].text)
```

**OpenAI:**

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful coding assistant."},
        {"role": "user", "content": "What is a closure in JavaScript?"},
        {"role": "assistant", "content": "A closure is a function that..."},
        {"role": "user", "content": "Can you show me an example?"}
    ]
)

print(response.choices[0].message.content)
```

The three roles:

- **system:** Instructions the model follows for the entire conversation. Set by the developer. The user shouldn't see or override this.
- **user:** Messages from the human. What the user typed.
- **assistant:** Previous responses from the model. You include these to give the model "memory" of the conversation.

The key insight: **the conversation is stateless.** The model doesn't remember anything. If you send only the latest user message, the model has no idea what came before. You must include the full conversation history (alternating user/assistant messages) for the model to maintain context. Your app is responsible for storing and managing this history.

### Streaming: Why Waiting Is Bad UX

Without streaming, the user sends a message and stares at a loading spinner for 5-30 seconds while the model generates the entire response. With streaming, tokens appear one by one as they're generated, just like watching someone type. The total time is the same, but the perceived speed is dramatically better.

**How streaming works:**

Instead of one HTTP response, the API sends the response as a series of Server-Sent Events (SSE). Each event contains a small chunk of the response (usually one token).

```
Your App                          LLM API
   │                                │
   │  POST /v1/messages             │
   │  { ..., stream: true }         │
   │────────────────────────────▶   │
   │                                │
   │  data: {"type":"content_block_delta","delta":{"text":"The"}}
   │◀────────────────────────────   │
   │  data: {"type":"content_block_delta","delta":{"text":" clos"}}
   │◀────────────────────────────   │
   │  data: {"type":"content_block_delta","delta":{"text":"ure"}}
   │◀────────────────────────────   │
   │  ...dozens more events...      │
   │  data: {"type":"message_stop"} │
   │◀────────────────────────────   │
```

**Anthropic streaming example:**

```python
import anthropic

client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Explain closures"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

**OpenAI streaming example:**

```python
from openai import OpenAI

client = OpenAI()

stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Explain closures"}],
    stream=True
)

for chunk in stream:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="", flush=True)
```

**In a web UI**, you typically use `fetch` with a `ReadableStream` or an `EventSource` on the client side, and your backend proxies the SSE stream from the LLM API. The client appends each chunk to the displayed text, creating the typing effect.

### Function Calling / Tool Use

Sometimes you need the model to do more than generate text. You need it to check a database, call an API, perform a calculation, or take an action. Function calling (Anthropic calls it "tool use") lets the model tell your app "I need to run this function with these arguments" instead of just generating text.

**The loop:**

```
1. You send the model a message + a list of available tools
2. Model decides it needs to use a tool
3. Model responds with: "Call function X with arguments Y"
4. YOUR CODE runs the function and gets the result
5. You send the result back to the model
6. Model uses the result to generate its final response
```

The model never runs your code directly. It only says "I want to call this function with these arguments." Your app runs the function and feeds the result back. You're in the loop.

**Anthropic tool use example:**

```python
import anthropic
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a city",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "City name"},
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
            },
            "required": ["city"]
        }
    }
]

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Bangalore?"}]
)

# Check if the model wants to use a tool
if response.stop_reason == "tool_use":
    tool_block = next(b for b in response.content if b.type == "tool_use")

    # YOUR CODE runs the actual function
    weather_data = get_weather_from_api(tool_block.input["city"])

    # Send the result back to the model
    final_response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        tools=tools,
        messages=[
            {"role": "user", "content": "What's the weather in Bangalore?"},
            {"role": "assistant", "content": response.content},
            {
                "role": "user",
                "content": [
                    {
                        "type": "tool_result",
                        "tool_use_id": tool_block.id,
                        "content": json.dumps(weather_data)
                    }
                ]
            }
        ]
    )
    print(final_response.content[0].text)
```

### Structured Outputs: Reliable JSON

When you need JSON back from the model, "please return JSON" in the prompt is not reliable. The model might add markdown code fences, include explanatory text before the JSON, or produce invalid JSON with trailing commas.

**Anthropic approach** (tool use for structured output):

Define a "tool" that matches your desired JSON schema. The model's tool use response is guaranteed to match the schema.

```python
tools = [
    {
        "name": "extract_contact",
        "description": "Extract contact info from text",
        "input_schema": {
            "type": "object",
            "properties": {
                "name": {"type": "string"},
                "email": {"type": "string"},
                "phone": {"type": "string"},
                "company": {"type": "string"}
            },
            "required": ["name", "email"]
        }
    }
]

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    tool_choice={"type": "tool", "name": "extract_contact"},  # force this tool
    messages=[{"role": "user", "content": "Email from Sarah Chen (sarah@acme.co), VP at Acme Corp"}]
)

# tool_block.input is guaranteed valid JSON matching the schema
```

**OpenAI approach** (response_format):

```python
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={"type": "json_object"},
    messages=[
        {"role": "system", "content": "Extract contact info. Return JSON with: name, email, phone, company"},
        {"role": "user", "content": "Email from Sarah Chen (sarah@acme.co), VP at Acme Corp"}
    ]
)

data = json.loads(response.choices[0].message.content)
```

### Model Selection: Cost vs Quality

Not every task needs the best model. Most tasks don't.

```
Task Complexity Spectrum:

Simple                          Complex
  │                                │
  │  Classification                │  Complex code generation
  │  Summarization                 │  Multi-step reasoning
  │  Translation                   │  Nuanced analysis
  │  Data extraction               │  Creative writing
  │  Template filling              │  Architectural decisions
  │  Sentiment analysis            │  Ambiguous tasks
  │                                │
  ▼                                ▼
  Cheap model (Haiku, 4o-mini)     Expensive model (Opus, GPT-4o)
  $0.25/M input tokens             $15/M input tokens
  Fast (< 1s)                      Slower (3-10s)
```

**Rules of thumb:**

- If the task has a clear right answer (classification, extraction, yes/no), use the cheapest model that gets it right.
- If you're providing few-shot examples, cheaper models improve dramatically. An extraction task with 3 examples on Haiku is often as good as zero-shot on Sonnet.
- If the task requires judgment, nuance, or handling edge cases, use a better model.
- If you're building a user-facing chat, use mid-tier (Sonnet, 4o) for quality-at-reasonable-cost.
- If you're processing thousands of documents in batch, use the cheapest model that meets your accuracy threshold, then spot-check with a better model.

**Multi-model routing** is a real production pattern: use a cheap model to classify the input difficulty, then route easy tasks to Haiku and hard tasks to Sonnet. This can cut costs by 60-80% with minimal quality loss.

### Rate Limits, Retries, and Error Handling

Every API has rate limits. When you exceed them, you get a 429 (Too Many Requests) response.

```
Common HTTP status codes from LLM APIs:

200  OK              - Everything worked
400  Bad Request     - Your request is malformed (check your JSON)
401  Unauthorized    - Bad API key
403  Forbidden       - Your key doesn't have access to this model
404  Not Found       - Wrong endpoint or model name
429  Rate Limited    - Too many requests, slow down
500  Server Error    - Their problem, not yours
529  Overloaded      - API is under heavy load (Anthropic-specific)
```

**Rate limits come in two flavors:**
- **Requests per minute (RPM):** How many API calls you can make per minute.
- **Tokens per minute (TPM):** How many tokens you can process per minute.

You can hit either independently. Sending 1,000 small requests might hit RPM. Sending 5 requests each with 100K tokens of context might hit TPM.

**Exponential backoff** is the standard retry strategy:

```python
import time
import random

def call_with_retry(func, max_retries=5):
    for attempt in range(max_retries):
        try:
            return func()
        except anthropic.RateLimitError:
            if attempt == max_retries - 1:
                raise
            wait = (2 ** attempt) + random.uniform(0, 1)  # 1s, 2s, 4s, 8s, 16s + jitter
            print(f"Rate limited. Retrying in {wait:.1f}s...")
            time.sleep(wait)
        except anthropic.APIStatusError as e:
            if e.status_code >= 500:
                # Server error, retry
                wait = (2 ** attempt) + random.uniform(0, 1)
                time.sleep(wait)
            else:
                # Client error (400, 401, 403), don't retry
                raise
```

Key principles:
- **Do retry:** 429 and 5xx errors. These are transient.
- **Don't retry:** 400, 401, 403. These won't fix themselves.
- **Add jitter:** Random delay prevents all your retries from hitting at the same time (thundering herd problem).
- **Set timeouts:** LLM calls can hang. Set a 60-second timeout for non-streaming calls and 120 seconds for streaming.

Both the Anthropic and OpenAI SDKs have built-in retry logic with exponential backoff. Use it:

```python
# Anthropic - retries are built in (2 retries by default)
client = anthropic.Anthropic(max_retries=3)

# OpenAI - same
client = OpenAI(max_retries=3)
```

## The Mental Model

**Mental model 1: "It's just a fancy HTTP endpoint."** Despite all the AI mystique, an LLM API call is an HTTP POST that takes JSON and returns JSON. You can test it with curl. You can debug it with print statements. You can monitor it with the same tools you use for any API. Don't overthink it.

**Mental model 2: "Tokens are money moving in both directions."** Input tokens cost less but accumulate fast (because you resend the full conversation every time). Output tokens cost more but you control them with max_tokens. Every architectural decision in an LLM-powered app is, at some level, a decision about how many tokens you're willing to spend. Think of tokens the way you think about database queries: each one is cheap, but waste adds up.

**Mental model 3: "The model is a stateless function."** `response = f(messages)`. Same input, roughly the same output (at temperature 0, exactly the same). No hidden state, no session, no memory. Everything the model knows is in the messages array you send. If you want memory, you build it. If you want personality, you send it. If you want context, you include it. The model is a pure function of its input.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Token | A chunk of text (~3/4 of a word) that the model reads and generates | API pricing, context limits, usage dashboards |
| Context window | Maximum total tokens (input + output) a model can handle in one call | Choosing models, handling long conversations |
| System prompt | Hidden instructions that control the model's behavior | Building any AI feature |
| Messages array | The list of conversation turns you send to the API | Every API call |
| Completion | The model's generated response | API responses, documentation |
| Streaming | Receiving the response token by token instead of all at once | Chat UIs, real-time features |
| SSE (Server-Sent Events) | The protocol used to stream responses over HTTP | Frontend streaming implementation |
| Function calling / Tool use | The model requesting your code to run a specific function | Building AI agents, integrating external data |
| max_tokens | Cap on how many tokens the model can generate in its response | Every API call (required by Anthropic) |
| Temperature | Controls randomness: 0 = deterministic, 1 = creative | API parameters |
| Rate limit | Maximum number of requests or tokens allowed per time period | Production apps, batch processing |
| 429 error | "Too many requests" - you've hit the rate limit | Any app under load |
| Exponential backoff | Retry strategy where wait time doubles each attempt (1s, 2s, 4s, 8s) | Error handling code |
| Prompt caching | Reusing processed input tokens across calls to save cost and time | Apps with long, repeated system prompts |
| Embeddings | Converting text to numbers (vectors) for similarity search, not generation | Search, RAG, recommendations |
| Fine-tuning | Training a model on your specific data to customize its behavior | Rarely needed, last resort after prompt engineering |

## Common Patterns

### 1. Simple Completion

**When to use it:** One-off tasks like summarization, classification, translation, or text generation where there's no conversation.

**How it works:** Single request, single response. No conversation history needed.

**Code sketch:**

```python
def summarize(text: str, max_sentences: int = 3) -> str:
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",  # cheap model, simple task
        max_tokens=256,
        temperature=0,
        system=f"Summarize the text in {max_sentences} sentences. No preamble.",
        messages=[{"role": "user", "content": text}]
    )
    return response.content[0].text
```

### 2. Streaming Chat UI

**When to use it:** Any user-facing chat interface. If a human is waiting for the response, you should be streaming.

**How it works:** Your backend proxies the SSE stream from the LLM API to the frontend. The frontend appends each chunk to the displayed message.

**Code sketch (backend, FastAPI):**

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.post("/chat")
async def chat(request: ChatRequest):
    async def generate():
        async with client.messages.stream(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            messages=request.messages
        ) as stream:
            async for text in stream.text_stream:
                yield f"data: {json.dumps({'text': text})}\n\n"
        yield "data: [DONE]\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

**Code sketch (frontend):**

```typescript
const response = await fetch("/chat", {
  method: "POST",
  body: JSON.stringify({ messages }),
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const chunk = decoder.decode(value);
  const lines = chunk.split("\n").filter(l => l.startsWith("data: "));

  for (const line of lines) {
    const data = line.slice(6);
    if (data === "[DONE]") return;
    const { text } = JSON.parse(data);
    appendToMessage(text);  // update your UI state
  }
}
```

### 3. Function Calling with External APIs

**When to use it:** The model needs real-time data (weather, stock prices, database lookups) or needs to take actions (send email, create ticket, update record).

**How it works:** You define tools, the model decides when to use them, your code executes the actual function, and you feed the result back. This often requires a loop because the model might need multiple tool calls.

**Code sketch:**

```python
def run_agent(user_message: str, tools: list, messages: list) -> str:
    messages.append({"role": "user", "content": user_message})

    while True:
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            tools=tools,
            messages=messages,
        )

        if response.stop_reason == "end_turn":
            # Model is done, return the text response
            return next(b.text for b in response.content if hasattr(b, "text"))

        if response.stop_reason == "tool_use":
            # Model wants to call a function
            messages.append({"role": "assistant", "content": response.content})

            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result),
                    })

            messages.append({"role": "user", "content": tool_results})
            # Loop continues - model will process the results
```

### 4. Batch Processing Documents

**When to use it:** Processing hundreds or thousands of documents (classify emails, extract data from invoices, summarize articles).

**How it works:** Use the cheapest model that works, process in parallel with concurrency limits, handle failures gracefully.

**Code sketch:**

```python
import asyncio

CONCURRENCY = 10  # stay under rate limits

async def process_batch(documents: list[str]) -> list[dict]:
    semaphore = asyncio.Semaphore(CONCURRENCY)
    async_client = anthropic.AsyncAnthropic(max_retries=3)

    async def process_one(doc: str) -> dict:
        async with semaphore:
            response = await async_client.messages.create(
                model="claude-haiku-4-5-20251001",  # cheapest model for batch
                max_tokens=256,
                temperature=0,
                system="Extract: name, email, company. Return JSON.",
                messages=[{"role": "user", "content": doc}],
            )
            return json.loads(response.content[0].text)

    results = await asyncio.gather(
        *[process_one(doc) for doc in documents],
        return_exceptions=True
    )

    # Separate successes from failures
    successes = [r for r in results if not isinstance(r, Exception)]
    failures = [r for r in results if isinstance(r, Exception)]
    print(f"Processed {len(successes)}/{len(documents)}, {len(failures)} failures")

    return successes
```

### 5. Multi-Model Routing

**When to use it:** You want the best cost/quality tradeoff across a variety of inputs. Easy questions go to cheap models, hard questions go to expensive ones.

**How it works:** A fast, cheap classifier model decides the difficulty. Then you route to the appropriate model.

**Code sketch:**

```python
async def route_and_respond(user_message: str, history: list) -> str:
    # Step 1: Classify complexity with cheapest model
    classification = await client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=20,
        temperature=0,
        system="""Classify this message as "simple" or "complex".
Simple: greetings, factual lookups, yes/no questions, basic formatting.
Complex: multi-step reasoning, analysis, code generation, creative writing.
Respond with one word only.""",
        messages=[{"role": "user", "content": user_message}],
    )

    complexity = classification.content[0].text.strip().lower()

    # Step 2: Route to appropriate model
    model = "claude-haiku-4-5-20251001" if complexity == "simple" else "claude-sonnet-4-20250514"

    response = await client.messages.create(
        model=model,
        max_tokens=2048,
        messages=history + [{"role": "user", "content": user_message}],
    )

    return response.content[0].text
```

This pattern can reduce API costs by 60-80% when most of your traffic is simple queries.

## Mistakes Everyone Makes

### 1. Not handling rate limits

**What people do:** Fire API calls as fast as possible and crash when 429 responses come back.

**Why it seems right:** "It's an API, it should handle my requests."

**What actually happens:** Your app throws unhandled exceptions. Users see errors. If you're doing batch processing, half your documents fail silently. The API returns `429 Too Many Requests` with a `retry-after` header that your code ignores.

**What to do instead:** Use the SDK's built-in retry logic (`max_retries=3`). For batch processing, use a semaphore or queue to limit concurrency. Log rate limit events so you can right-size your concurrency settings.

### 2. Sending full conversation history without trimming

**What people do:** Append every message to the history array and send everything every time.

**Why it seems right:** "The model needs context to give good responses."

**What actually happens:** After 20-30 messages, you're sending 10K+ tokens of history per request. Costs escalate. Eventually you hit the context window limit and get an error. Or worse, the model starts ignoring early messages because attention degrades over very long inputs.

**What to do instead:** Implement a sliding window: keep the system prompt, the last N messages (10-20 is usually enough), and optionally a summary of older messages. For long conversations, summarize the first half and keep the second half verbatim.

```python
def trim_messages(messages: list, max_messages: int = 20) -> list:
    if len(messages) <= max_messages:
        return messages
    # Keep the first message (often important context) and the last N
    return [messages[0]] + messages[-max_messages:]
```

### 3. Hardcoding model names

**What people do:** Sprinkle `"gpt-4o"` or `"claude-sonnet-4-20250514"` throughout their codebase.

**Why it seems right:** "It's just a string, I'll change it later."

**What actually happens:** Model names change. New versions are released. You need to A/B test models. Now you're grep-ing through 40 files to find every instance. Some get missed. Different parts of your app use different models accidentally.

**What to do instead:** Define model names in one place (environment variable or config file). Reference the config everywhere.

```python
# config.py
import os

MODELS = {
    "fast": os.getenv("MODEL_FAST", "claude-haiku-4-5-20251001"),
    "balanced": os.getenv("MODEL_BALANCED", "claude-sonnet-4-20250514"),
    "powerful": os.getenv("MODEL_POWERFUL", "claude-opus-4-20250514"),
}
```

### 4. Ignoring token costs during development

**What people do:** Use the most expensive model for everything during development, send massive prompts, don't monitor usage.

**Why it seems right:** "I'm just testing, costs are low."

**What actually happens:** You build patterns that are expensive by design. Your system prompt is 3,000 tokens when it could be 500. You send 10 few-shot examples when 3 would do. You use Opus for classification when Haiku works fine. Then you launch, get users, and your first monthly bill is 10x what you budgeted.

**What to do instead:** Log token usage from day one. Add it to your development output: "This request used 1,247 input tokens and 389 output tokens ($0.005)." Optimize prompts for token efficiency before launch, not after the bill arrives. Use the cheapest model that works for each task.

### 5. Not streaming responses in user-facing apps

**What people do:** Use non-streaming API calls and show a loading spinner until the full response arrives.

**Why it seems right:** "Streaming is more complex to implement."

**What actually happens:** The user stares at a spinner for 5-15 seconds. That's an eternity in UX terms. Users think the app is broken and click away. Even if the response is great, the experience feels slow. Meanwhile, ChatGPT streams by default and feels instant even though it takes the same total time.

**What to do instead:** Always stream in user-facing apps. Both SDKs make streaming straightforward (see the streaming section above). The extra 20 lines of code are worth it. Non-streaming is fine for background processing and batch jobs where no human is waiting.

### 6. Not setting max_tokens appropriately

**What people do:** Set max_tokens to 4096 "just in case" for every request, or don't set it at all.

**Why it seems right:** "I want the model to have room to respond fully."

**What actually happens:** The model sometimes generates padding, repetitive conclusions, or unnecessary detail to fill the available space. Classification tasks that should return one word return a paragraph of explanation. You pay for output tokens you don't need.

**What to do instead:** Set max_tokens to match the expected response length. Classification: 10-20. Short answers: 256. Summaries: 512. Code generation: 2048-4096. This also acts as a safety valve against runaway responses.

### 7. Treating the API key like it's not a secret

**What people do:** Commit API keys to git, put them in frontend code, share them on Slack.

**Why it seems right:** "It's just a development key."

**What actually happens:** Someone finds it (bots scan GitHub constantly), runs up thousands of dollars in charges on your account, and you don't notice until the bill arrives. OpenAI and Anthropic have billing alerts, but the damage can happen fast.

**What to do instead:** API keys go in environment variables, never in code. Use `.env` files locally (and add `.env` to `.gitignore`). Use secrets management in production (Railway secrets, Vercel env vars, AWS Secrets Manager). Rotate keys immediately if exposed.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Stream every user-facing response.** No human should stare at a spinner while tokens generate. Streaming adds minimal complexity and transforms the user experience.

2. **Set max_tokens intentionally for every call.** Don't use a blanket 4096. Match it to the task: 20 for classification, 256 for summaries, 1024 for general responses. This saves money and prevents rambling.

3. **Use the cheapest model that meets your quality bar.** Start with Haiku/4o-mini. Move up only when you prove the cheap model can't handle it. Most extraction, classification, and summarization tasks don't need the expensive model.

4. **Log token usage from day one.** Track input tokens, output tokens, cost per request, and cost per user. You can't optimize what you don't measure. Even during development.

5. **Trim conversation history before sending.** Keep a sliding window of the last 15-20 messages. Summarize older history. Never send unbounded conversation arrays.

6. **Put model names and parameters in configuration, not code.** One config file, one source of truth. Change models by changing an env var, not by editing 30 files.

7. **Handle errors by category: retry transient, fail fast on permanent.** 429 and 5xx get retried with exponential backoff. 400 and 401 get surfaced immediately. Never retry a bad request.

8. **Never put API keys in frontend code or git.** Use environment variables. Add `.env` to `.gitignore`. Set up billing alerts. Rotate keys if exposed.

## The "Just Tell Me What to Do" Quickstart

Get a working API call to both OpenAI and Anthropic in under 5 minutes.

**Step 1: Install both SDKs**

```bash
pip install anthropic openai
```

**Step 2: Set API keys**

```bash
export ANTHROPIC_API_KEY="your-anthropic-key"
export OPENAI_API_KEY="your-openai-key"
```

Get keys at [console.anthropic.com](https://console.anthropic.com) and [platform.openai.com](https://platform.openai.com).

**Step 3: Create `llm_test.py`**

```python
import anthropic
from openai import OpenAI

question = "What's the difference between a list and a tuple in Python? Answer in 2 sentences."

# --- Anthropic ---
claude = anthropic.Anthropic()
claude_response = claude.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=256,
    temperature=0,
    messages=[{"role": "user", "content": question}]
)
print("=== Claude ===")
print(claude_response.content[0].text)
print(f"Tokens: {claude_response.usage.input_tokens} in, {claude_response.usage.output_tokens} out\n")

# --- OpenAI ---
gpt = OpenAI()
gpt_response = gpt.chat.completions.create(
    model="gpt-4o",
    max_tokens=256,
    temperature=0,
    messages=[{"role": "user", "content": question}]
)
print("=== GPT-4o ===")
print(gpt_response.choices[0].message.content)
print(f"Tokens: {gpt_response.usage.prompt_tokens} in, {gpt_response.usage.completion_tokens} out")
```

**Step 4: Run it**

```bash
python llm_test.py
```

You'll see both models answer the same question, plus the token counts. Compare the responses, the token usage, and (if you time it) the latency.

**Step 5: Try streaming**

Add this to the same file:

```python
print("\n=== Claude Streaming ===")
with claude.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=256,
    messages=[{"role": "user", "content": question}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
print()
```

Run again and watch the response appear token by token.

## How to Prompt AI Tools About This

### What context to give AI tools

When asking Claude Code, Cursor, or Copilot to write LLM API integration code, always include:
- Which provider (Anthropic, OpenAI, or both)
- Which SDK version (the APIs change frequently)
- Whether you need streaming or non-streaming
- Your runtime environment (Python, Node.js, edge function, etc.)
- Whether this is a one-off script or production code (production needs error handling, retries, logging)

### Example prompts that produce good results

**For basic integration:**
```
Write a Python function that calls Claude Sonnet via the Anthropic SDK.
It should take a system prompt and user message as arguments, stream the
response, and return the full text plus token usage. Use the latest SDK
patterns. Include error handling for rate limits and API errors.
```

**For a chat backend:**
```
Build a FastAPI endpoint that proxies streaming responses from Claude.
It should accept a messages array, maintain conversation history in memory
(keyed by session ID), trim history to the last 20 messages, and stream
SSE events to the client. Use the async Anthropic client.
```

**For function calling:**
```
Show me the complete request-response loop for Claude tool use in Python.
I need the model to call a "search_database" function that takes a query
string and returns a list of results. Show the full loop: initial request,
detecting tool_use, executing the function, sending results back, and
getting the final response.
```

### What to watch out for in AI-generated code

- **Outdated SDK patterns:** The Anthropic and OpenAI SDKs are updated frequently. AI might generate code for older versions. Check that import paths and method names match the current SDK.
- **Missing error handling:** AI tends to generate the happy path. Always ask explicitly for rate limit handling and timeout management.
- **Synchronous when you need async:** AI defaults to synchronous clients. If you're in an async framework (FastAPI, Next.js API routes), explicitly ask for async.
- **Wrong streaming implementation:** There are multiple ways to stream. Make sure the generated code matches your framework (SSE for web, async iterator for CLI).

### Key terms that improve output quality

- "Use the latest Anthropic SDK" or "Use openai>=1.0" (prevents deprecated patterns)
- "Include token usage logging" (gets cost tracking built in)
- "Handle 429 with exponential backoff" (gets proper retry logic)
- "Use async client" (gets the right client for async frameworks)
- "Stream with SSE" (gets proper server-sent events, not WebSockets)

## Ship It: Build This

### CLI Chat Tool with Multi-Provider Support

Build a command-line chat application that streams responses from either Claude or GPT, tracks conversation history and token usage, and lets you switch between providers with a flag.

**Why this project:** It exercises every concept in this guide. You'll implement streaming, conversation history management, token counting, error handling, and multi-model support. And you'll actually use it: a CLI chat tool is genuinely useful for quick questions and prompt testing.

**Rough architecture:**
- Single Python file, ~200 lines
- `--provider` flag: `anthropic` (default) or `openai`
- `--model` flag: override the default model for each provider
- `--temperature` flag: defaults to 0.7
- `--system` flag: optional system prompt
- Streams responses to the terminal in real time
- Maintains conversation history in memory (for multi-turn chat)
- Displays token usage after each response (input tokens, output tokens, estimated cost)
- `/clear` command to reset conversation history
- `/tokens` command to show total tokens used in this session
- `/switch` command to swap providers mid-conversation
- Handles Ctrl+C gracefully (doesn't crash, just cancels the current response)
- Handles rate limits with automatic retry and user-visible feedback

**Key features to build:**
- Streaming output with proper terminal handling (flush each token)
- Token counting and cost estimation per message and cumulative
- Conversation history with sliding window (last 20 messages)
- Graceful error handling (rate limits, network errors, invalid keys)
- Provider abstraction: same interface for both Anthropic and OpenAI

**Estimated time:** 3-4 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn prompt caching** when your system prompt is longer than 1,000 tokens and you're making many API calls. Anthropic's prompt caching can reduce input costs by up to 90% for repeated system prompts.
- **Learn the Batch API** when you need to process thousands of requests and don't need real-time results. Both Anthropic and OpenAI offer batch endpoints at 50% cost reduction with 24-hour turnaround.
- **Learn embeddings** when you need similarity search, clustering, or recommendations. Embeddings are a different API endpoint that converts text to vectors. See the [RAG guide](../rag/) for how this connects to retrieval.
- **Learn fine-tuning** when prompt engineering has hit its ceiling and you have hundreds of high-quality examples of the exact input/output pairs you want. This is a last resort, not a first step. Most people who think they need fine-tuning actually need better prompts.
- **Learn multi-modal inputs** when you need to process images, PDFs, or audio alongside text. Claude and GPT-4o both accept images in the messages array. The API is the same, you just add an image content block.
