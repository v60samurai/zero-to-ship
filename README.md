# zero-to-ship

Technical guides that take you from "what is this?" to "I just built something with it."

Written for the vibe coding era. You don't need to memorize syntax. You need mental models that let you prompt AI tools effectively, understand what they generate, and debug when things break.

Every guide follows the same structure: analogy first, then the real explanation, then the gotchas, then a project you can actually ship.

## Guides

| Topic | What You'll Learn |
|-------|------------------|
| [Git](guides/git/) | Version control, branches, commits, and the workflow that keeps your codebase safe |
| [RAG](guides/rag/) | How to make AI answer questions from your own documents using embeddings, vector search, and LLMs |
| [Prompt Engineering](guides/prompt-engineering/) | How to write prompts that get consistent, useful results from AI models, from basic techniques to production patterns and prompt injection defense |
| [LLM APIs](guides/llm-apis/) | How to use LLM APIs from OpenAI, Anthropic, and others: tokens, streaming, function calling, structured outputs, and cost management |
| [AI Agents](guides/ai-agents/) | How AI agents work: the observe-think-act loop, tool use, planning, memory, multi-agent systems, and guardrails for production |
| [APIs and REST](guides/apis-rest/) | How APIs and REST work: HTTP methods, status codes, authentication, pagination, rate limiting, and building your own API |
| [Databases and SQL](guides/databases-sql/) | How databases and SQL work: tables, queries, JOINs, indexes, relationships, ORMs, migrations, and the mistakes that kill performance |
| [Authentication and OAuth](guides/auth-oauth/) | How authentication and OAuth work: sessions, JWTs, social login, password hashing, and why you should use an auth library |
| [TypeScript](guides/typescript/) | How TypeScript works: types, interfaces, generics, React patterns, reading errors, and why strict mode catches bugs before your users do |

*Coming soon — guides will be listed here as they're created.*

<!-- Template for adding guides:
| [Topic](guides/topic-slug/) | One-line description | -->

## Who This Is For

You build things with Cursor, Claude Code, Lovable, v0, or similar tools. You ship fast. But sometimes you hit a concept (WebSockets, OAuth, Docker, SQL joins, whatever) and realize you're prompting AI without understanding what it's doing. These guides fix that gap in 20-30 minutes per topic.

## How to Use These

1. **Got 30 seconds?** Read "The 30-Second Version" at the top of any guide.
2. **Got 10 minutes?** Read through "The Mental Model" section. You'll have enough to work with AI tools confidently.
3. **Got 30 minutes?** Read the whole guide and do the quickstart.
4. **Want to go deep?** Build the project in "Ship It" at the end.

## Contributing

Each guide lives in `guides/[topic-slug]/README.md`. If you spot an error, an analogy that misleads, or outdated quickstart commands, open a PR.
