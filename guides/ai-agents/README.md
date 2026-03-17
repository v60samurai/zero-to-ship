# AI Agents and Tool Use: Zero to Ship

> An AI that doesn't just answer questions but actually does things: searches the web, writes files, calls APIs, and decides what to do next on its own.

## Why Should You Care?

You've used ChatGPT or Claude. You type a question, you get an answer. That's a single LLM call. It's useful, but it's limited. The model can only work with what's in its head (training data) and what you paste into the prompt. It can't check a live website, query your database, run code to verify its answer, or take any action in the real world.

An agent changes that. An agent is an LLM in a loop: it reads the situation, decides what to do, uses a tool (search the web, run code, read a file), observes the result, and then decides what to do next. It keeps looping until the task is done. Claude Code (the tool you're probably using right now) is an agent. It reads your codebase, decides which files to edit, makes changes, runs tests, and fixes errors, all without you telling it every step.

This matters because the gap between "AI that answers questions" and "AI that completes tasks" is where most of the value is. Answering "how do I deploy to Vercel?" is helpful. Actually reading your project config, figuring out the right settings, creating the deployment script, and running it is transformative. Agents are how you get from the first to the second. If you're building AI-powered products, understanding agents is not optional anymore. It's the difference between building a chatbot and building something that actually works.

## The 30-Second Version

An agent is an LLM that runs in a loop. Each iteration, it looks at the current state, decides whether to use a tool (search, calculate, read a file, call an API) or give a final answer, executes the tool, reads the result, and loops again. You give it a goal, a set of tools, and guardrails. It figures out the steps. The key insight is that the LLM doesn't run the tools itself. It says "I want to call this function with these arguments," your code runs the function, and you feed the result back to the LLM for the next decision.

## The Real Explanation (ELI5)

Imagine you hire a new personal assistant. On their first day, you could use them in two ways.

**Way 1 (single LLM call):** You walk up to them and ask, "What's a good restaurant near MG Road in Bangalore?" They answer from memory. Maybe they're right, maybe they're outdated. They can't check Google Maps, read recent reviews, or call the restaurant to confirm hours. They just tell you what they remember.

**Way 2 (agent):** You walk up and say, "Book me a dinner reservation for two near MG Road tonight." Now they need to do several things: search for restaurants in the area, check which ones have availability tonight, read reviews to pick a good one, call the restaurant to book, and confirm the details back to you. They don't ask you for step-by-step instructions. They figure out the steps themselves. If the first restaurant is full, they try the next one. If they can't find the phone number, they check the website.

The key difference: Way 1 is a single question-answer. Way 2 is a goal with a loop. The assistant observes (what do I know so far?), thinks (what should I do next?), acts (makes a phone call, checks a website), and repeats until the job is done.

Now replace the assistant with an LLM. Replace "call the restaurant" with a function your code provides called `make_reservation(restaurant_id, date, party_size)`. Replace "check Google Maps" with a `search_web(query)` function. The LLM decides when to use which function and with what arguments. Your code actually executes them. That's an agent.

The analogy breaks at one important point: your human assistant has common sense and real-world awareness. The LLM agent doesn't. It can get stuck in loops, make nonsensical tool calls, or pursue a plan that clearly won't work. That's why guardrails (maximum iterations, cost caps, human approval for risky actions) are essential, not optional.

## How It Actually Works

### The Agent Loop

Every agent, no matter how fancy, follows the same fundamental loop:

```
                    ┌──────────────────────┐
                    │    START: User Goal   │
                    │  "Research X and      │
                    │   write a summary"    │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │    OBSERVE            │
                    │  Read current state:  │
                    │  - conversation so far│
                    │  - tool results       │
                    │  - progress toward    │
                    │    the goal           │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │    THINK             │
                    │  LLM decides:        │
                    │  - Am I done?        │
                    │  - What's missing?   │
                    │  - Which tool next?  │
                    └──────────┬───────────┘
                               │
                     ┌─────────┴─────────┐
                     │                   │
              ┌──────▼──────┐    ┌───────▼──────┐
              │  USE A TOOL │    │ GIVE ANSWER  │
              │  search()   │    │ Return final │
              │  read()     │    │ response to  │
              │  calculate()│    │ the user     │
              └──────┬──────┘    └──────────────┘
                     │
              ┌──────▼──────┐
              │   ACT       │
              │ YOUR CODE   │
              │ runs the    │
              │ function    │
              └──────┬──────┘
                     │
                     │ result goes back
                     │ to the LLM
                     │
                     └──────► (back to OBSERVE)
```

**What happens when an agent runs:** You send the LLM a system prompt (its role, available tools, constraints), plus the conversation history (all previous observations and actions). The LLM generates either a tool call ("I want to search for X") or a final answer ("Here's what I found"). If it's a tool call, your code executes the tool, appends the result to the conversation, and calls the LLM again. This repeats until the LLM decides it's done.

**Under the hood:** Each iteration is a separate API call. The LLM sees the full conversation (growing with each tool result) and picks the next action. There's no hidden state or persistent process. It's just repeated LLM calls with an expanding context window. That's why agents can be expensive: a 10-step agent is 10 API calls, each one sending the full history.

**Why it's designed this way:** The LLM can't actually execute code or access the internet. It can only generate text. By giving it a list of tool definitions (name, description, parameters), you let it express intent ("call search_web with query 'best restaurants MG Road'"), and your code translates that intent into real execution. This keeps the LLM sandboxed while still giving it real-world capabilities.

### Tool Use: The Mechanism

Tools are functions you define and offer to the LLM. Each tool has a name, a description (so the LLM knows when to use it), and a parameter schema (so the LLM knows what arguments to pass).

**Anthropic tool use example:**

```python
import anthropic
import json

client = anthropic.Anthropic()

tools = [
    {
        "name": "search_web",
        "description": "Search the web for current information. Use this when you need facts, data, or recent events that might not be in your training data.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "The search query"
                }
            },
            "required": ["query"]
        }
    },
    {
        "name": "read_url",
        "description": "Read the full text content of a webpage. Use this after search_web to get detailed information from a specific result.",
        "input_schema": {
            "type": "object",
            "properties": {
                "url": {
                    "type": "string",
                    "description": "The URL to read"
                }
            },
            "required": ["url"]
        }
    }
]
```

The tool description is critical. It's not just documentation. The LLM reads it to decide when and how to use the tool. A vague description leads to wrong tool calls. A good description includes what the tool does, when to use it, and when not to use it.

### The ReAct Pattern

ReAct (Reasoning + Acting) is the most common agent pattern. The LLM alternates between reasoning about the problem and taking actions with tools.

Here's what a ReAct agent's internal conversation looks like:

```
System: You are a research assistant. Use tools to find accurate information.
        Think step by step. Always verify claims with at least one source.

User: What's the population of Bangalore and how has it changed in the last decade?

Assistant (thinking): I need current population data for Bangalore.
Let me search for this.
→ Tool call: search_web("Bangalore population 2024 census data")

Tool result: [search results with several links about Bangalore population]

Assistant (thinking): I found some results. The top result mentions 13.6 million
for the metro area. Let me read the source to verify and get historical data.
→ Tool call: read_url("https://example.com/bangalore-demographics")

Tool result: [full article with population data from 2011 and 2024]

Assistant (thinking): Now I have 2011 census data (8.4 million city, 9.6 million
metro) and 2024 estimates (11.5 million city, 13.6 million metro). That's about
a 37% increase in the city proper over ~13 years. I have enough to answer.
→ Final response: "Bangalore's population has grown significantly..."
```

The key is that each "thinking" step is visible in the conversation. The LLM reasons about what it knows, what it still needs, and what tool to use next. This interleaving of thought and action is what makes agents effective at complex tasks.

### Planning: How Agents Handle Big Tasks

For simple tasks (look up one fact), the agent doesn't need much planning. For complex tasks (research a topic from multiple angles), the agent needs to break the work into steps.

**Linear planning:** The agent thinks through steps sequentially. "First I'll search for X, then read the top results, then summarize." Simple, easy to follow, but can miss better approaches.

```python
system = """You are a research agent. Before taking any action, write a brief plan:
1. What information do I need?
2. What's my search strategy?
3. How will I verify what I find?

Then execute the plan step by step. If a step fails or reveals new information
that changes the plan, update the plan before continuing."""
```

**Tree-of-thought planning:** For genuinely hard problems, the agent considers multiple approaches, evaluates which is most promising, and pursues that one. If it hits a dead end, it backtracks and tries another branch.

```python
system = """When faced with a complex question, consider 2-3 different approaches
before choosing one. For each approach, briefly assess:
- Likelihood of finding the answer
- Number of steps required
- Risk of dead ends

Choose the most promising approach. If it doesn't work after 3 attempts,
switch to the next approach."""
```

In practice, linear planning handles 90% of agent tasks. Tree-of-thought is for genuinely complex reasoning problems.

### Memory: How Agents Remember

Agents have three types of memory:

**Short-term memory (conversation history):** Everything in the current messages array. This is the agent's "working memory" for the current task. It grows with each tool call and result. Limited by the context window.

**Working memory (scratchpad):** A dedicated space where the agent writes notes to itself. Useful for tracking progress on multi-step tasks.

```python
system = """You have a scratchpad for tracking your research progress.
After each tool call, update your scratchpad:

<scratchpad>
- Goal: [what you're trying to find]
- Found so far: [key facts discovered]
- Still need: [what's missing]
- Sources: [URLs and credibility assessment]
</scratchpad>

This helps you stay focused and avoid redundant searches."""
```

**Long-term memory (vector store / database):** Information persisted between conversations. The agent can search past interactions, recall user preferences, or access a knowledge base. This is where RAG (retrieval-augmented generation) connects to agents. The agent has a `search_memory` tool that queries a vector database.

```python
tools = [
    {
        "name": "search_memory",
        "description": "Search your long-term memory for past conversations, user preferences, or previously researched topics. Use this before starting new research to avoid duplicating past work.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "save_to_memory",
        "description": "Save an important finding or user preference to long-term memory for future reference.",
        "input_schema": {
            "type": "object",
            "properties": {
                "content": {"type": "string"},
                "tags": {"type": "array", "items": {"type": "string"}}
            },
            "required": ["content"]
        }
    }
]
```

### Multi-Agent Systems

Sometimes one agent isn't enough. You need specialists.

**Orchestrator pattern:** One "manager" agent receives the task, breaks it into subtasks, and delegates each to a specialist agent. The manager collects results and synthesizes the final answer.

```
User: "Write a blog post about React Server Components with code examples"

Orchestrator Agent
  ├── Research Agent → searches web for latest RSC patterns and opinions
  ├── Code Agent → writes working code examples, runs them to verify
  └── Writing Agent → combines research + code into a polished blog post

Orchestrator reviews and returns the final post.
```

**Handoff pattern:** Agents pass the conversation to each other based on the topic. Common in customer support: a router agent determines intent, then hands off to a billing agent, technical support agent, or general agent.

```python
system = """You are a routing agent. Analyze the user's message and determine
which specialist should handle it:

- "billing_agent": questions about plans, pricing, invoices, refunds
- "technical_agent": bugs, errors, API issues, integration help
- "onboarding_agent": new user setup, getting started, tutorials

Respond with ONLY the agent name. Do not try to answer the question yourself."""
```

**When to use multi-agent vs single agent:**
- Single agent with many tools: tasks where one entity needs context of the whole problem.
- Multi-agent: tasks that are clearly separable into independent subtasks, or when different subtasks need different system prompts / tool sets / models.

The overhead of multi-agent coordination is real. Don't use it for tasks a single agent handles fine.

### Guardrails: Keeping Agents Safe

Agents without guardrails are dangerous. They can loop forever, burn through your API budget, or take destructive actions you didn't intend.

**1. Maximum iterations:**

```python
MAX_ITERATIONS = 15

for i in range(MAX_ITERATIONS):
    response = client.messages.create(...)

    if response.stop_reason == "end_turn":
        break

    if i == MAX_ITERATIONS - 1:
        # Force the agent to stop and summarize what it has
        messages.append({
            "role": "user",
            "content": "You've reached the maximum number of steps. Summarize what you've found so far and give your best answer with the information available."
        })
```

**2. Cost caps:**

```python
total_tokens = 0
MAX_TOKENS_BUDGET = 100_000  # about $0.30 on Sonnet

for i in range(MAX_ITERATIONS):
    response = client.messages.create(...)
    total_tokens += response.usage.input_tokens + response.usage.output_tokens

    if total_tokens > MAX_TOKENS_BUDGET:
        print(f"Budget exceeded: {total_tokens} tokens used")
        break
```

**3. Human-in-the-loop for destructive actions:**

```python
DANGEROUS_TOOLS = {"delete_file", "send_email", "execute_sql", "deploy"}

def execute_tool(name: str, args: dict) -> str:
    if name in DANGEROUS_TOOLS:
        print(f"\nAgent wants to: {name}({json.dumps(args, indent=2)})")
        approval = input("Approve? (y/n): ")
        if approval.lower() != "y":
            return "Action denied by user. Try a different approach."

    return tool_functions[name](**args)
```

**4. Tool-level validation:**

```python
def search_web(query: str) -> str:
    # Prevent the agent from searching for things it shouldn't
    if len(query) > 200:
        return "Error: query too long. Simplify your search."
    if any(term in query.lower() for term in BLOCKED_TERMS):
        return "Error: this search is not allowed."
    return _do_search(query)
```

## The Mental Model

**Mental model 1: "An agent is a while loop around an LLM call."** Strip away all the frameworks, abstractions, and buzzwords. An agent is: call the LLM, check if it wants to use a tool, run the tool if so, repeat. That's it. Every agent framework is a fancy wrapper around this loop. Understanding the loop means you can build agents from scratch and debug any framework.

**Mental model 2: "Tools are the agent's hands."** The LLM is a brain in a jar. Without tools, it can only think and talk. Tools let it interact with the world. The quality of an agent is determined by three things in order: the tools you give it, the instructions you write, and the model you use. Most people over-invest in model choice and under-invest in tool design.

**Mental model 3: "Every agent step costs money and time."** Each tool call is a full LLM API request. A 10-step agent task sends 10 API calls, each one carrying the full growing conversation. Agent tasks are 10-50x more expensive than single LLM calls. This means you should only use agents when the task genuinely needs a loop. If you can solve it in one LLM call, don't use an agent.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Agent | An LLM in a loop that can use tools and decide its own next steps | Building anything beyond simple Q&A |
| Tool / Function calling | Letting the LLM request your code to run specific functions | Any agent, API integrations |
| Agent loop | The observe-think-act-repeat cycle that drives agent behavior | Every agent framework |
| ReAct | A pattern where the agent alternates reasoning ("I should search for...") with actions (actually searching) | Research agents, complex tasks |
| Orchestrator | An agent that manages other agents, breaking tasks into subtasks | Multi-agent systems |
| Handoff | Passing a conversation from one agent to another based on the topic | Customer support, routing |
| Guardrails | Safety limits: max iterations, cost caps, approval gates, output validation | Production agents |
| Scratchpad | A section of the prompt where the agent keeps notes about its progress | Long-running agent tasks |
| Context window | Maximum text the LLM can process (agent history grows each step) | Cost management, long tasks |
| Human-in-the-loop | Requiring human approval before the agent takes risky actions | Production agents with write access |
| Tool description | The text that tells the LLM when and how to use a tool | Tool design (this is critically important) |
| Stop reason | The API field that tells you whether the LLM wants to use a tool or is done | Agent loop logic |
| Planning | The agent's ability to break a big task into smaller steps before executing | Complex, multi-step tasks |
| Tree-of-thought | Considering multiple approaches to a problem before picking one | Hard reasoning problems |
| Working memory | The conversation history that grows during an agent task | Every agent run |
| Long-term memory | Information stored between conversations (database, vector store) | Persistent agents, personalization |

## Common Patterns

### 1. Simple Tool-Calling Agent

**When to use it:** The task needs 1-3 tool calls. Look something up, transform it, return the result.

**How it works:** Basic agent loop with a small set of focused tools. Most tasks complete in 2-5 iterations.

**Code sketch:**

```python
import anthropic
import json

client = anthropic.Anthropic()

def run_agent(user_message: str, tools: list, tool_functions: dict) -> str:
    messages = [{"role": "user", "content": user_message}]

    for _ in range(10):  # max iterations
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4096,
            system="You are a helpful assistant. Use tools when needed.",
            tools=tools,
            messages=messages,
        )

        # Check if the agent is done
        if response.stop_reason == "end_turn":
            return next(
                (b.text for b in response.content if hasattr(b, "text")),
                "No response generated."
            )

        # Process tool calls
        messages.append({"role": "assistant", "content": response.content})
        tool_results = []

        for block in response.content:
            if block.type == "tool_use":
                result = tool_functions[block.name](**block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(result),
                })

        messages.append({"role": "user", "content": tool_results})

    return "Agent reached maximum iterations without completing."
```

### 2. Research Agent (Search + Summarize)

**When to use it:** You need to gather information from multiple sources, cross-reference facts, and produce a synthesis.

**How it works:** Agent searches the web, reads promising pages, evaluates source credibility, and compiles findings into a structured summary.

**Code sketch:**

```python
research_tools = [
    {
        "name": "search_web",
        "description": "Search the web. Returns titles, URLs, and snippets. Use specific, targeted queries.",
        "input_schema": {
            "type": "object",
            "properties": {"query": {"type": "string"}},
            "required": ["query"]
        }
    },
    {
        "name": "read_page",
        "description": "Read the full content of a webpage. Use after search_web to get details from a specific result.",
        "input_schema": {
            "type": "object",
            "properties": {"url": {"type": "string"}},
            "required": ["url"]
        }
    }
]

research_system = """You are a research agent. Your goal is to answer questions
accurately using web sources.

Process:
1. Search for the topic with 2-3 different queries to get diverse results
2. Read the most promising 3-5 pages
3. Cross-reference facts across sources
4. Produce a summary with inline citations

Your final answer must include:
- A clear summary (3-5 paragraphs)
- Key facts with sources
- Any conflicting information you found
- Confidence level (high/medium/low) with reasoning

If you can't find reliable information, say so. Never fabricate sources."""
```

### 3. Coding Agent (Write + Test + Fix)

**When to use it:** Generating code that needs to be correct, not just plausible. The agent writes code, runs it, sees errors, and fixes them.

**How it works:** The agent has tools to write files, execute code, and read error output. It iterates until the code passes tests or runs successfully.

**Code sketch:**

```python
coding_tools = [
    {
        "name": "write_file",
        "description": "Write content to a file. Use for creating or updating code files.",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"},
                "content": {"type": "string"}
            },
            "required": ["path", "content"]
        }
    },
    {
        "name": "run_command",
        "description": "Run a shell command and return stdout + stderr. Use for running tests, linting, or executing code. Timeout: 30 seconds.",
        "input_schema": {
            "type": "object",
            "properties": {
                "command": {"type": "string"}
            },
            "required": ["command"]
        }
    },
    {
        "name": "read_file",
        "description": "Read the contents of a file.",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"}
            },
            "required": ["path"]
        }
    }
]

coding_system = """You are a coding agent. Write code, test it, and fix errors.

Rules:
1. Always write a test BEFORE writing the implementation
2. Run the test to confirm it fails (red)
3. Write the implementation
4. Run the test again to confirm it passes (green)
5. If the test fails, read the error carefully, fix the code, and retry
6. Never mark a task as done until tests pass

If you're stuck after 3 fix attempts on the same error, explain what's
going wrong and ask for help."""
```

### 4. Workflow Automation Agent

**When to use it:** Multi-step business processes that follow a rough pattern but need judgment at each step (processing support tickets, reviewing applications, generating reports).

**How it works:** The agent follows a defined workflow but uses judgment to handle edge cases that would break a rigid automation.

**Code sketch:**

```python
workflow_tools = [
    {
        "name": "get_pending_tickets",
        "description": "Get unprocessed support tickets. Returns list of {id, subject, body, priority}.",
        "input_schema": {"type": "object", "properties": {}}
    },
    {
        "name": "classify_ticket",
        "description": "Update a ticket's category and priority based on analysis.",
        "input_schema": {
            "type": "object",
            "properties": {
                "ticket_id": {"type": "string"},
                "category": {"type": "string", "enum": ["billing", "technical", "account", "feature"]},
                "priority": {"type": "string", "enum": ["urgent", "high", "medium", "low"]},
                "summary": {"type": "string"}
            },
            "required": ["ticket_id", "category", "priority", "summary"]
        }
    },
    {
        "name": "draft_response",
        "description": "Save a draft response for human review. Do NOT send directly.",
        "input_schema": {
            "type": "object",
            "properties": {
                "ticket_id": {"type": "string"},
                "response": {"type": "string"}
            },
            "required": ["ticket_id", "response"]
        }
    }
]

workflow_system = """You are a support ticket processing agent.

For each ticket:
1. Read the ticket content carefully
2. Classify it (category + priority)
3. Draft a helpful response

Rules:
- Urgent tickets mentioning "down", "outage", or "can't access": priority = urgent
- Never draft a response that promises a timeline
- If a ticket is unclear, classify as medium priority and ask a clarifying question
- Process tickets one at a time, completing all steps before moving to the next"""
```

### 5. Multi-Agent Debate/Collaboration

**When to use it:** Tasks where multiple perspectives improve quality (code review, content editing, decision analysis).

**How it works:** Two or more agents with different roles examine the same input. An orchestrator synthesizes their perspectives.

**Code sketch:**

```python
async def multi_agent_review(code: str) -> dict:
    # Agent 1: Security reviewer
    security_review = await client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2048,
        system="""You are a security-focused code reviewer. Only flag genuine
        security issues: injection, auth bypass, data exposure, SSRF. Ignore
        style, performance, and minor issues. For each finding: severity
        (critical/high/medium), the vulnerable line, and the fix.""",
        messages=[{"role": "user", "content": f"Review this code:\n\n{code}"}],
    )

    # Agent 2: Performance reviewer
    perf_review = await client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2048,
        system="""You are a performance-focused code reviewer. Flag: N+1 queries,
        missing indexes, unnecessary allocations, blocking operations in async
        code, missing caching opportunities. Ignore security and style.""",
        messages=[{"role": "user", "content": f"Review this code:\n\n{code}"}],
    )

    # Orchestrator: synthesize
    synthesis = await client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2048,
        system="""Synthesize two code reviews into a single prioritized list.
        Remove duplicates. Order by severity. Format as a numbered list.""",
        messages=[{
            "role": "user",
            "content": f"Security review:\n{security_review.content[0].text}\n\nPerformance review:\n{perf_review.content[0].text}"
        }],
    )

    return {
        "security": security_review.content[0].text,
        "performance": perf_review.content[0].text,
        "synthesis": synthesis.content[0].text,
    }
```

## Mistakes Everyone Makes

### 1. Giving agents too many tools at once

**What people do:** Define 20+ tools and hand them all to the agent.

**Why it seems right:** "More tools means the agent can handle more situations."

**What actually happens:** The LLM gets confused about which tool to use. It picks the wrong one, passes wrong arguments, or oscillates between tools without making progress. Each tool description eats context window space, leaving less room for actual conversation. Tool selection accuracy drops sharply after about 10 tools.

**What to do instead:** Give the agent 3-7 tools max. If you need more, use an orchestrator pattern: one agent routes to specialists, each with their own focused tool set. Or dynamically select tools based on the current step: "For the search phase, only search tools are available. For the writing phase, only file tools are available."

### 2. No exit conditions (infinite loops)

**What people do:** Build an agent loop without a maximum iteration limit or cost cap.

**Why it seems right:** "The agent will figure out when it's done."

**What actually happens:** The agent gets stuck. It searches for something, doesn't find it, rephrases the query, searches again, reads a page, decides it needs more info, searches again. You come back to a $47 API bill and 200 failed iterations. This happens more often than you'd think, especially with open-ended tasks.

**What to do instead:** Always set three limits: max iterations (10-20 for most tasks), max tokens budget, and a time limit. When any limit is hit, force the agent to summarize what it has and return a partial answer. A partial answer is infinitely better than an infinite loop.

```python
MAX_ITERATIONS = 15
MAX_COST_CENTS = 50  # $0.50

for i in range(MAX_ITERATIONS):
    if estimated_cost > MAX_COST_CENTS:
        force_summarize_and_return()
        break
```

### 3. Not logging agent steps for debugging

**What people do:** Run the agent and only look at the final output.

**Why it seems right:** "I only care about the result."

**What actually happens:** The agent produces a wrong answer and you have no idea why. Was it a bad search query? Did it misread a page? Did it hallucinate a source? Without step-by-step logs, debugging an agent is like debugging code without stack traces. You're guessing.

**What to do instead:** Log every step: the LLM's reasoning, each tool call (name + arguments), each tool result, and the token count per step. Print it to console during development. Store it in a database in production. This is the single most important thing for agent reliability.

```python
for i in range(MAX_ITERATIONS):
    response = client.messages.create(...)
    print(f"\n--- Step {i + 1} ---")
    print(f"Tokens: {response.usage.input_tokens} in, {response.usage.output_tokens} out")
    for block in response.content:
        if block.type == "text":
            print(f"Thinking: {block.text[:200]}...")
        elif block.type == "tool_use":
            print(f"Tool: {block.name}({json.dumps(block.input)})")
```

### 4. Using agents for predictable tasks

**What people do:** Build an agent to do something that follows the exact same steps every time.

**Why it seems right:** "Agents are the future. Everything should be an agent."

**What actually happens:** The agent takes 10 seconds and costs $0.05 to do what a simple function call would do in 100ms for free. It sometimes skips steps or does them in the wrong order. It's less reliable, slower, and more expensive than hardcoded logic.

**What to do instead:** Use agents when the task requires judgment, when the steps vary based on input, or when the agent needs to handle unexpected situations. If the steps are always the same, write a function. "Fetch data from API, transform it, save to database" is a function. "Research a topic and decide what's worth reporting" is an agent.

**The rule:** If you can write the logic as an if/else tree, don't use an agent.

### 5. Skipping human approval for destructive actions

**What people do:** Give the agent tools to send emails, delete files, execute SQL, or deploy code, and let it use them freely.

**Why it seems right:** "The whole point of an agent is autonomous action."

**What actually happens:** The agent sends an email to a customer with wrong information. The agent deletes a file it shouldn't have. The agent runs a SQL query that modifies production data. These aren't hypothetical scenarios. They happen in production when agents encounter unexpected situations and take the "logical" (but wrong) next step.

**What to do instead:** Mark tools as "safe" (read-only: search, read file, calculate) or "dangerous" (write: send email, delete, deploy, execute SQL). Safe tools run automatically. Dangerous tools require human approval. In development, you can be more lenient. In production, every write action should require confirmation until you've built enough trust and logging.

### 6. Building a custom framework instead of starting simple

**What people do:** Spend two weeks building an agent framework with plugins, middleware, state machines, and configuration files before writing a single agent.

**Why it seems right:** "I need a proper architecture. This will save time later."

**What actually happens:** The framework doesn't match the actual needs that emerge when you start building real agents. You end up fighting your own abstractions. Meanwhile, a 50-line while loop would have shipped the feature last week.

**What to do instead:** Start with the raw agent loop (the while loop from the first code sketch in this guide). Build 3-4 agents. Notice what patterns repeat. Then extract those patterns into shared code. Framework after experience, never before.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Start with the simplest agent that could work.** One model, 3-5 tools, a basic loop, and a max iteration cap. Add complexity only when you prove it's needed.

2. **Write tool descriptions like you're explaining to a new employee.** Include what the tool does, when to use it, when NOT to use it, and what the arguments mean. The LLM reads these descriptions to make decisions.

3. **Log every agent step in production.** Tool calls, tool results, reasoning, token counts, and elapsed time. You cannot debug an agent without a trace. This is not optional.

4. **Set three hard limits on every agent: max iterations, max tokens, max time.** When any limit is hit, the agent must return its best partial answer. Infinite loops are the number one agent failure mode.

5. **Require human approval for any action that can't be undone.** Sending emails, writing to databases, deleting files, deploying code. Read-only actions can be automatic. Write actions need a gate until you trust the agent deeply.

6. **Use the cheapest model that works for each agent role.** The router agent that classifies queries doesn't need Opus. The research agent that synthesizes complex topics might. Multi-model routing isn't just for cost. It's for speed too. Haiku responds in under a second.

7. **Test agents with adversarial inputs.** What happens when the user asks something completely outside the agent's scope? When a tool returns an error? When search results are empty? Agents must handle failure gracefully, not loop endlessly.

8. **Don't use an agent when a function will do.** If the steps are always the same, write a function. Agents are for tasks that require judgment, adaptation, or variable-length reasoning. Using an agent for a fixed workflow is like hiring a consultant to flip a light switch.

## The "Just Tell Me What to Do" Quickstart

Build a working agent in under 10 minutes. This agent can search the web and read pages.

**Step 1: Install the SDK**

```bash
pip install anthropic
```

**Step 2: Set your API key**

```bash
export ANTHROPIC_API_KEY="your-key-here"
```

**Step 3: Create `agent.py`**

```python
import anthropic
import json
import urllib.request
import urllib.parse

client = anthropic.Anthropic()

# Simple tool implementations
def search_web(query: str) -> str:
    """Use DuckDuckGo instant answer API (no API key needed)."""
    url = f"https://api.duckduckgo.com/?q={urllib.parse.quote(query)}&format=json&no_html=1"
    try:
        with urllib.request.urlopen(url, timeout=10) as resp:
            data = json.loads(resp.read())
            results = []
            if data.get("AbstractText"):
                results.append(f"Summary: {data['AbstractText']}")
                results.append(f"Source: {data.get('AbstractURL', 'N/A')}")
            for topic in data.get("RelatedTopics", [])[:5]:
                if isinstance(topic, dict) and "Text" in topic:
                    results.append(f"- {topic['Text']}")
            return "\n".join(results) if results else "No results found."
    except Exception as e:
        return f"Search error: {e}"

def calculate(expression: str) -> str:
    """Evaluate a math expression safely."""
    try:
        allowed = set("0123456789+-*/.() ")
        if all(c in allowed for c in expression):
            return str(eval(expression))
        return "Error: invalid expression"
    except Exception as e:
        return f"Calculation error: {e}"

# Tool definitions for Claude
tools = [
    {
        "name": "search_web",
        "description": "Search the web using DuckDuckGo. Returns summaries and related topics. Use for factual questions about real-world topics.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "The search query"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "calculate",
        "description": "Evaluate a mathematical expression. Use for any arithmetic, percentages, or conversions. Example: '(100 * 0.15) + 50'",
        "input_schema": {
            "type": "object",
            "properties": {
                "expression": {"type": "string", "description": "Math expression to evaluate"}
            },
            "required": ["expression"]
        }
    }
]

tool_functions = {
    "search_web": lambda **kwargs: search_web(kwargs["query"]),
    "calculate": lambda **kwargs: calculate(kwargs["expression"]),
}

# The agent loop
def run_agent(question: str) -> str:
    messages = [{"role": "user", "content": question}]
    total_tokens = 0

    for step in range(10):
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2048,
            system="You are a research assistant. Use tools to find accurate, current information. Always cite your sources.",
            tools=tools,
            messages=messages,
        )

        total_tokens += response.usage.input_tokens + response.usage.output_tokens
        print(f"  [Step {step + 1}] tokens: {response.usage.input_tokens}+{response.usage.output_tokens}")

        # Check if done
        if response.stop_reason == "end_turn":
            print(f"  [Total tokens: {total_tokens}]")
            return next(
                (b.text for b in response.content if hasattr(b, "text")),
                "No response."
            )

        # Process tool calls
        messages.append({"role": "assistant", "content": response.content})
        tool_results = []

        for block in response.content:
            if block.type == "tool_use":
                print(f"  [Tool] {block.name}({json.dumps(block.input)})")
                result = tool_functions[block.name](**block.input)
                print(f"  [Result] {result[:150]}...")
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result,
                })

        messages.append({"role": "user", "content": tool_results})

    return "Agent reached max iterations."


if __name__ == "__main__":
    question = input("Ask me anything: ")
    print()
    answer = run_agent(question)
    print(f"\n{'='*60}\n{answer}")
```

**Step 4: Run it**

```bash
python agent.py
```

Try: "What's the population of the three largest cities in India, and what's their combined population?"

Watch the agent search, extract numbers, calculate, and synthesize.

**Step 5: Experiment**

- Add a third tool (read a URL, check the time, convert currencies). See how the agent incorporates it.
- Lower max iterations to 3. Watch the agent rush to an answer.
- Remove one tool and ask a question that needs it. Watch the agent adapt (or fail).
- Make the system prompt more specific. See how it changes the agent's approach.

## How to Prompt AI Tools About This

### What context to give AI tools

When asking Claude Code, Cursor, or Copilot to build agents, always include:
- Which LLM provider (Anthropic, OpenAI, or both)
- The specific tools the agent needs (describe what each tool does)
- Maximum iterations and budget constraints
- Whether this is a prototype or production code (production needs logging, error handling, cost tracking)
- What the agent should do when it gets stuck (return partial results, ask for help, try a different approach)

### Example prompts that produce good results

**For building a basic agent:**
```
Build a Python agent using the Anthropic SDK that can search the web
and read web pages. It should take a user question, research it using
2-3 searches, read the most relevant pages, and return a summary with
sources. Max 10 iterations. Log every tool call. Track token usage.
Use Claude Sonnet.
```

**For adding tools to an existing agent:**
```
I have an agent loop that uses Claude tool use. Add a new tool called
"query_database" that takes a SQL query string and returns results.
The tool should only allow SELECT queries (reject INSERT/UPDATE/DELETE).
Show me the tool definition and the executor function. The agent should
use this tool when users ask about data in our database.
```

**For debugging an agent:**
```
My agent gets stuck in a loop when search results are empty. It keeps
rephrasing the same query. Here's my agent code: [paste]. Here's the
log output showing the loop: [paste]. How do I add a check that detects
repeated tool calls and forces the agent to give up or try a completely
different approach?
```

### What to watch out for in AI-generated agent code

- **Missing iteration limits:** AI often generates agent loops without max_iterations. Always add one.
- **No error handling for tool calls:** What happens when a tool throws an exception? AI-generated code often crashes. Wrap every tool call in try/except.
- **Oversized tool descriptions:** AI tends to write essay-length descriptions. Keep them to 2-3 sentences.
- **No logging:** AI generates clean code that's impossible to debug. Always add step-by-step logging.
- **Synchronous when you need async:** For multi-agent systems, make sure the generated code uses async properly so agents can run in parallel.

### Key terms that improve output quality

- "Include step-by-step logging" (gets observable agent behavior)
- "Add max iterations and cost caps" (gets proper guardrails)
- "Use tool_choice to force tool use when needed" (gets reliable tool calling)
- "Handle tool errors gracefully" (gets resilient agents)
- "Return partial results on timeout" (gets graceful degradation)

## Ship It: Build This

### Research Agent with Web Search and Structured Output

Build a research agent that takes a question, searches the web, reads relevant pages, cross-references information, and produces a structured summary with sources.

**Why this project:** It exercises every concept in this guide: the agent loop, tool use, planning (deciding what to search), memory (tracking what's been found), guardrails (max iterations, cost tracking), and structured output (the final summary format). A research agent is also genuinely useful. You'll use it.

**Rough architecture:**
- Single Python file, ~250 lines
- Three tools: `search_web` (returns search results), `read_page` (fetches and extracts text from a URL), `save_finding` (stores a fact + source to the agent's scratchpad)
- System prompt that enforces a research process: search from multiple angles, read at least 2-3 sources, cross-reference facts
- Structured final output: title, summary (3-5 paragraphs), key findings (bullet points with sources), confidence level, and a "what I couldn't find" section
- Step-by-step logging: every tool call, every result, token usage per step
- Guardrails: max 15 iterations, max 50K tokens budget, 2-minute timeout
- The agent returns partial results if any limit is hit

**Key features to build:**
- Web search using DuckDuckGo or SerpAPI
- Page reading with HTML-to-text extraction (use `trafilatura` or `beautifulsoup4`)
- A scratchpad mechanism (save_finding tool that accumulates findings)
- Token tracking and cost estimation
- Structured output as JSON at the end
- CLI interface: `python research_agent.py "your question here"`

**Estimated time:** 4-5 hours for a vibe coder using AI tools.

## Go Deeper (Optional)

- **Learn the Claude Agent SDK** when you want to build production agents without writing the loop yourself. It provides agent primitives, guardrails, and multi-agent orchestration out of the box.
- **Learn LangGraph or CrewAI** when you need complex multi-agent workflows with conditional branching, parallel execution, and state management. These frameworks shine for workflows that are too complex for a simple while loop but too structured for fully autonomous agents.
- **Learn evaluation (evals) for agents** when your agent is in production and you need to measure reliability. Agent evals track task completion rate, tool call accuracy, cost per task, and error recovery. This is harder than evaluating single LLM calls but essential for production.
- **Learn MCP (Model Context Protocol)** when you want your agent to connect to external tools and data sources using a standardized protocol instead of writing custom integrations for each one.
- **Learn fine-tuning for tool selection** when your agent consistently picks the wrong tool despite good descriptions. Fine-tuning on tool-selection examples can improve accuracy, but this is a last resort after optimizing descriptions and adding examples to the system prompt.
