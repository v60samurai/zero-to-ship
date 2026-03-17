# CLAUDE.md — zero-to-ship

This project is a collection of ELI5 technical guides that take someone from "what is this?" to "I just built something with it." Every guide is written for the vibe coding era — people who build with AI tools (Cursor, Claude Code, Lovable, v0) and need to understand concepts well enough to ship, not pass an exam.

## Project Structure

```
zero-to-ship/
├── CLAUDE.md
├── README.md
├── guides/
│   ├── websockets/
│   │   └── README.md
│   ├── docker/
│   │   └── README.md
│   ├── oauth/
│   │   └── README.md
│   └── [topic]/
│       └── README.md
└── .github/
    └── ...
```

Each guide lives in `guides/[topic-slug]/README.md`. One topic, one folder, one file. If a guide needs images or code examples that can't be inline, they go in the same folder.

## Guide Creation Rules

When I ask you to create a guide on any topic, follow this structure and philosophy exactly.

### Philosophy

1. **Analogy first, jargon second.** Every concept gets a real-world analogy before the technical explanation. If you can't explain it with a restaurant, a post office, or a phone call, you don't understand it well enough.

2. **Progressive disclosure.** Start dumb simple. Layer complexity. The reader should be able to stop at any section and walk away with something useful.

3. **Vibe coder context.** The reader uses AI coding tools daily. They don't need to memorize syntax. They need mental models — the "why" and "when" so they can prompt AI tools effectively and understand what the AI generates.

4. **Ship-oriented.** Every guide ends with something the reader can build or do. Not "further reading." An actual project or action.

5. **Honest about complexity.** Don't pretend hard things are easy. Say "this part is genuinely complex, here's why" and then break it down anyway.

### Guide Structure (follow this exactly)

```markdown
# [Topic]: Zero to Ship

> One-line description a 10-year-old would understand.

## Why Should You Care?

2-3 paragraphs. Real-world motivation. What breaks without this? What becomes possible with it? Why is this relevant NOW for someone building with AI tools?

No "in today's rapidly evolving landscape" garbage. Concrete scenarios.

## The 30-Second Version

The entire concept in one short paragraph. If someone reads ONLY this and nothing else, they walk away with the core idea. Think of it as the tweet-length explanation.

## The Real Explanation (ELI5)

Analogy-driven explanation. Use a real-world scenario the reader has experienced. Walk through the analogy completely before introducing any technical terms.

Then bridge: "Now replace [analogy element] with [technical element]..."

## How It Actually Works

Now go technical, but keep it clear. Use diagrams in text (ASCII or mermaid-compatible). Walk through the flow step by step.

Label the layers:
- **What happens when...** (the user-visible flow)
- **Under the hood...** (what the system is actually doing)
- **Why it's designed this way...** (the tradeoff that led to this design)

## The Mental Model

Give the reader 2-3 mental models or rules of thumb they can carry forever. These are the things that let them reason about edge cases without looking anything up.

Format: "Think of [concept] as [mental model]. This means whenever you see [situation], you know [implication]."

## Key Concepts (Jargon Decoder)

A glossary-style section. Every important term in the topic, explained in one plain-English sentence each. No term should reference another term the reader hasn't seen yet. Order matters — build up.

Format as a table:
| Term | Plain English | When You'll See It |
|------|--------------|-------------------|

## Common Patterns

3-5 most common ways this technology is used in real projects. For each:
- **Pattern name**
- **When to use it** (one sentence)
- **How it works** (brief explanation)
- **Code sketch** (minimal pseudocode or real code showing the shape, not production-ready)

## Mistakes Everyone Makes

5-7 common mistakes, ordered from beginner to intermediate. For each:
- What people do wrong
- Why it seems right
- What actually happens
- What to do instead

This section is gold. It saves people hours of debugging. Be specific with error messages and symptoms.

## Best Practices (The Rules That Prevent 90% of Problems)

5-8 numbered rules that experienced practitioners follow instinctively. These are the habits that separate someone who "knows" the technology from someone who uses it well.

For each rule:
- One sentence stating the rule clearly
- One sentence explaining why (the consequence of breaking it)

Keep it scannable. A reader should be able to skim this section in 60 seconds and carry these rules into their next project. No elaboration beyond the rule and the why. If a rule needs more context, it probably belongs in "Mistakes Everyone Makes" instead.

Order from most fundamental to most nuanced. The first 3 rules should cover what a beginner needs. The last 2-3 should be things even intermediate users forget.

## The "Just Tell Me What to Do" Quickstart

Step-by-step instructions to get a working example running in under 10 minutes. Assume the reader has a Mac with Node.js/Python installed and uses VS Code or Cursor. Every command is copy-pasteable.

This is NOT a tutorial. It's the fastest path to a working thing the reader can poke at and modify.

## How to Prompt AI Tools About This

This section is unique to vibe coding era guides. Teach the reader:
- What context to give Claude/Cursor/Copilot when working with this technology
- Example prompts that produce good results
- What to watch out for in AI-generated code for this topic (common AI mistakes)
- Key terms to use in prompts that improve output quality

## Ship It: Build This

One concrete project the reader can build that uses this technology meaningfully. Not a toy example. Something they'd actually want to show someone.

Include:
- What they're building (one paragraph)
- Why this project specifically (it exercises the important parts)
- Rough architecture (bullet points)
- Estimated time: X hours for a vibe coder using AI tools

## Go Deeper (Optional)

Only if the topic has genuinely important advanced aspects. Not a link dump. 3-5 specific things with one sentence each on why they matter and when to learn them.

Format: "Learn [X] when you need to [specific situation]."
```

### Writing Rules

- Plain English. Fifth-grade reading level for explanations, technical precision for code and specifications.
- No em dashes. Use commas, periods, or parentheses instead.
- No "passionate," "excited," "deep dive," "unlock," "at scale" as filler, "ecosystem," "landscape," or any word that exists only to sound impressive.
- Complete sentences. No punchy fragments used as emphasis tricks.
- First person is fine when sharing opinions ("I'd recommend..." or "In my experience..."). Frame it as the guide's voice.
- Analogies must be precise. If the analogy breaks down at an important point, say so explicitly.
- Code examples use the most common/practical language for that topic. If the topic is language-agnostic, default to JavaScript/TypeScript (most vibe coders use it).
- Every heading should make sense in a table of contents. Someone scanning headings should understand the guide's arc.

### Quality Bar

Before considering a guide done, verify:
- [ ] A non-technical person can read the first 3 sections and get the idea
- [ ] A vibe coder can read the whole thing and confidently work with AI on this topic
- [ ] Every analogy is accurate (doesn't mislead at any important point)
- [ ] The quickstart actually works (commands are real, dependencies are current)
- [ ] The "Mistakes Everyone Makes" section has specific symptoms/errors, not vague warnings
- [ ] The AI prompting section has real, copy-pasteable example prompts
- [ ] No section is filler. Every paragraph earns its place.

### Updating the Index

After creating any new guide, update the root `README.md` to include the new guide in the topic list with a one-line description.

## Commands

- When I say "guide: [topic]" — create a full guide following the structure above
- When I say "outline: [topic]" — create just the outline with bullet points for what each section would cover (for my review before writing)
- When I say "improve: [topic]" — review an existing guide and suggest specific improvements
- When I say "index" — regenerate the root README.md with all current guides listed
