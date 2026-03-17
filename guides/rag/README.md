# RAG (Retrieval-Augmented Generation): Zero to Ship

> Teaching an AI to answer questions by first looking things up in your own files, like a student allowed to use their notes during an exam.

## Why Should You Care?

You've used ChatGPT or Claude. They're impressively smart. But ask them about your company's internal docs, your product's changelog, or the Slack thread from last Tuesday, and they have no idea. They were trained on internet data from months ago. They don't know your stuff.

The brute-force fix is to paste your documents into the chat. That works until you have 500 pages of documentation, or your data changes weekly, or you need to answer questions across 50 different files. You hit context window limits, spend a fortune on tokens, and still get mediocre answers because the model is drowning in irrelevant text.

RAG solves this. Instead of dumping everything into the prompt, you build a system that finds the relevant pieces of your data first, then hands only those pieces to the LLM. The model gets exactly the context it needs, nothing it doesn't, and generates answers grounded in your actual data. This is how every "chat with your docs" product works. Notion AI, GitHub Copilot's codebase search, customer support bots that actually know your product, internal knowledge bases that answer real questions. They're all RAG under the hood.

## The 30-Second Version

RAG is a two-step process: retrieve, then generate. First, you search your own documents to find the chunks most relevant to the user's question. Then you paste those chunks into the AI's prompt and ask it to answer based on that context. The AI gets to "look up" your data before responding, so it gives answers grounded in your actual information instead of making things up.

## The Real Explanation (ELI5)

Imagine you're a new employee on your first day. Someone asks you a question about company policy. You're smart, you speak well, but you simply don't know the answer because you've never read the employee handbook.

Now imagine you have a really helpful colleague sitting next to you. When someone asks a question, your colleague doesn't answer it themselves. Instead, they know the handbook inside out. They flip to the exact pages that are relevant, hand you just those pages, and say "the answer is in here somewhere." You read those few pages and give a confident, accurate answer.

That's RAG. The LLM is you (the smart new employee). Your documents are the handbook. The retrieval system is the helpful colleague who finds the right pages. The generation step is you reading those pages and forming an answer.

Now replace "flipping to the right pages" with a similarity search across your documents, and replace "pages" with chunks of text that have been converted into mathematical representations. The colleague finds relevant chunks not by memorizing keywords, but by understanding what concepts are similar to the question being asked.

One place the analogy breaks down: unlike a colleague who reads sequentially, the retrieval system converts everything into numbers (embeddings) ahead of time. When a question comes in, it converts the question into numbers too, then finds the chunks whose numbers are closest. This is what makes it fast enough to search millions of documents in milliseconds.

## How It Actually Works

RAG has two phases: ingestion (preparing your data ahead of time) and query (answering questions at runtime).

### Phase 1: Ingestion (happens once, or when data changes)

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Your Docs   │────►│   Chunking   │────►│  Embedding   │────►│   Vector     │
│  (PDF, MD,   │     │  Split into  │     │  Convert to  │     │   Database   │
│   HTML...)   │     │  small pieces│     │  numbers     │     │   Store for  │
│              │     │  (chunks)    │     │  (vectors)   │     │   searching  │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
```

**What happens:** Your documents get split into smaller pieces (chunks), each chunk gets converted into a list of numbers (an embedding vector), and those vectors get stored in a database optimized for similarity search.

**Under the hood:** The embedding model (like OpenAI's `text-embedding-3-small` or Cohere's `embed-english-v3.0`) reads each chunk and produces a vector, a list of 256 to 3072 numbers. These numbers encode the meaning of the text. Chunks about similar topics end up with similar numbers, regardless of the exact words used.

**Why it's designed this way:** You can't search by meaning in a regular database. SQL can find exact keyword matches, but it can't find "passages about refund policies" when the text says "customers may return items within 30 days." Converting to vectors lets you search by concept, not just keywords.

### Phase 2: Query (happens every time a user asks a question)

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  User's      │────►│  Embed the   │────►│  Search for  │────►│  Build       │
│  Question    │     │  question    │     │  similar     │     │  prompt with │
│              │     │  (same model)│     │  chunks      │     │  retrieved   │
│              │     │              │     │  (top-k)     │     │  context     │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ┌──────────────┐
                                                               │  LLM         │
                                                               │  generates   │
                                                               │  answer from │
                                                               │  context     │
                                                               └──────────────┘
```

**What happens:** The user's question gets converted into a vector using the same embedding model. The vector database finds the chunks whose vectors are closest (most semantically similar). Those chunks get inserted into the LLM's prompt alongside the question. The LLM generates an answer based on the provided context.

**Under the hood:** "Closest" is measured by cosine similarity, a way of calculating how similar two vectors point in the same direction. A score of 1.0 means identical meaning, 0.0 means completely unrelated. You typically retrieve the top 3-10 most similar chunks (called "top-k retrieval").

**Why it's designed this way:** The LLM has a limited context window (128K tokens for Claude, 128K for GPT-4o). You can't fit all your docs in there. Retrieval picks the needle from the haystack so the LLM only sees what matters.

## The Mental Model

**Think of RAG as an open-book exam.** The LLM is the student. Your documents are the textbook. The retrieval system is the act of flipping to the right pages. This means: the quality of your answers depends just as much on finding the right pages as on the student's intelligence. A brilliant student with the wrong pages will give a confident, well-written, wrong answer.

**Think of embeddings as coordinates on a meaning map.** Just like GPS coordinates let you find places near each other geographically, embedding vectors let you find text near each other conceptually. "How do I cancel my subscription?" and "Steps to end your membership" are far apart in word-space but right next to each other in embedding-space.

**Think of chunking as cutting a book into index cards.** If your cards are too big (entire chapters), you'll retrieve a lot of irrelevant text alongside the useful parts. If they're too small (individual sentences), you'll lose context that makes the sentence meaningful. The art of chunking is finding the right card size for your specific content.

## Key Concepts (Jargon Decoder)

| Term | Plain English | When You'll See It |
|------|--------------|-------------------|
| Embedding | A list of numbers that represents the meaning of a piece of text. Similar meanings produce similar numbers. | When converting your documents or queries into vectors |
| Vector | The actual list of numbers an embedding model produces. Typically 256-3072 numbers long. | When storing or searching embeddings |
| Vector database | A database built specifically to find "nearby" vectors quickly. Regular databases can't do this efficiently. | When choosing where to store your embeddings (Pinecone, Chroma, pgvector) |
| Chunk | A small piece of a larger document. You break docs into chunks because embedding models and LLMs have size limits. | When preparing documents for ingestion |
| Cosine similarity | A way to measure how similar two vectors are, on a scale from -1 to 1. Higher means more similar. | When the retrieval system ranks which chunks match a query |
| Top-k | The number of most-similar chunks you retrieve. k=5 means "give me the 5 best matches." | When configuring how many chunks to feed the LLM |
| Context window | The maximum amount of text an LLM can process in a single request (prompt + response combined). | When deciding how many chunks to include in your prompt |
| Reranking | A second pass that re-scores retrieved chunks using a more accurate (and slower) model. Improves precision. | When basic similarity search isn't returning good enough results |
| Hybrid search | Combining keyword search (BM25) with semantic/vector search. Catches both exact matches and conceptual matches. | When pure semantic search misses results that contain specific terms |
| Ingestion | The process of loading, chunking, embedding, and storing your documents. Happens before any queries. | When setting up your RAG system or updating documents |
| Faithfulness | Whether the LLM's answer is actually supported by the retrieved chunks, not hallucinated. | When evaluating RAG quality |
| Retrieval recall | The percentage of relevant chunks that your retrieval system actually finds. | When debugging why your RAG gives incomplete answers |

## Common Patterns

### 1. Basic Q&A Over Documents

**When to use it:** You have a collection of documents (docs, wikis, PDFs) and want users to ask questions and get answers with source citations.

**How it works:** Ingest all documents, embed them, store in a vector database. On each query, retrieve top-k chunks, inject into prompt, generate answer. Include source references so the user can verify.

**Code sketch:**

```python
# Ingestion
from langchain_community.document_loaders import DirectoryLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

loader = DirectoryLoader("./docs", glob="**/*.md")
docs = loader.load()

splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(docs)

vectorstore = Chroma.from_documents(chunks, OpenAIEmbeddings(), persist_directory="./chroma_db")

# Query
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})
relevant_chunks = retriever.invoke("How do refunds work?")

prompt = f"""Answer based only on the following context:

{chr(10).join(chunk.page_content for chunk in relevant_chunks)}

Question: How do refunds work?"""
```

### 2. Conversational RAG (Chat with History)

**When to use it:** Users have multi-turn conversations where follow-up questions reference earlier messages ("What about for enterprise customers?" after asking about pricing).

**How it works:** Before retrieval, the system rewrites the user's latest message to be self-contained by incorporating chat history. "What about for enterprise?" becomes "What is the pricing for enterprise customers?" This rewritten query then goes through normal RAG retrieval.

**Code sketch:**

```python
# Rewrite the follow-up question to be self-contained
rewrite_prompt = f"""Given this conversation history:
{chat_history}

Rewrite this follow-up question to be self-contained:
"{user_message}"

Self-contained question:"""

standalone_question = llm.invoke(rewrite_prompt)

# Now do normal RAG with the rewritten question
chunks = retriever.invoke(standalone_question)
answer = generate_with_context(chunks, standalone_question)
```

### 3. Hybrid Search (Keyword + Semantic)

**When to use it:** When your documents contain specific identifiers, product codes, error messages, or technical terms that semantic search alone might miss. "Error code E-4012" needs exact matching, not conceptual similarity.

**How it works:** Run both a traditional keyword search (BM25) and a vector similarity search in parallel. Combine the results using Reciprocal Rank Fusion (RRF) or a weighted merge. This catches both exact term matches and conceptual matches.

**Code sketch:**

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever

# Keyword search
bm25_retriever = BM25Retriever.from_documents(chunks)
bm25_retriever.k = 5

# Semantic search
vector_retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# Combine with equal weighting
hybrid_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, vector_retriever],
    weights=[0.4, 0.6]  # favor semantic slightly
)

results = hybrid_retriever.invoke("Error E-4012 connection timeout")
```

### 4. Agentic RAG (LLM Decides What to Retrieve)

**When to use it:** When a single retrieval step isn't enough. The LLM might need to search multiple collections, reformulate queries, or decide it needs more context before answering.

**How it works:** Instead of a fixed retrieve-then-generate pipeline, the LLM gets retrieval as a tool it can call. It decides when to search, what to search for, and whether the results are sufficient. If the first search doesn't return good results, it can rephrase and try again.

**Code sketch:**

```python
from langchain.tools import Tool
from langchain.agents import create_react_agent

# Give the LLM retrieval as a tool
search_tool = Tool(
    name="search_docs",
    description="Search internal documentation. Use specific terms.",
    func=lambda query: retriever.invoke(query)
)

# The agent decides when and how to search
agent = create_react_agent(
    llm=llm,
    tools=[search_tool],
    prompt=agent_prompt
)

# Agent might search multiple times, refine queries, etc.
result = agent.invoke({"input": "Compare our refund policy with our exchange policy"})
# Agent internally: search "refund policy" -> search "exchange policy" -> synthesize
```

## Mistakes Everyone Makes

### 1. Chunks that are too big or too small

**What people do wrong:** Using a single chunk size without thinking about the content. Either setting chunk_size=4000 (dumping half a document) or chunk_size=100 (losing all context).

**Why it seems right:** Bigger chunks mean more context for the LLM, which sounds helpful. Or smaller chunks mean more precise retrieval, which also sounds helpful.

**What actually happens:** Chunks too big (>2000 tokens) dilute the embedding. The vector represents an average of too many topics, so similarity search gets fuzzy. You retrieve chunks that are "kinda related" to everything but precisely relevant to nothing. Chunks too small (<100 tokens) lose context. "The policy was updated in 2024" means nothing without knowing which policy.

**What to do instead:** Start with 500-1000 tokens per chunk with 10-20% overlap. Test with your actual data and questions. If answers are vague, try smaller chunks. If answers lack context, try larger ones or increase overlap.

### 2. Ignoring document updates

**What people do wrong:** Building a RAG system once, ingesting documents, and never updating the vector database when documents change.

**Why it seems right:** The ingestion pipeline works, the system gives good answers, it feels "done."

**What actually happens:** Users ask about the latest pricing, but the system still has the old pricing document embedded. The LLM confidently gives outdated information. Worse, if documents are deleted but their chunks remain in the vector store, the system returns information from documents that no longer exist.

**What to do instead:** Track document versions. Use a hash or last-modified timestamp for each source document. On updates, delete old chunks for that document and re-ingest the new version. Build this into your pipeline from day one, not as an afterthought. Most vector databases support filtering by metadata, so tag each chunk with its source document ID.

### 3. Not using metadata filtering

**What people do wrong:** Treating the vector database as a single undifferentiated bucket. Every chunk goes in with no tags, no source info, no dates.

**Why it seems right:** Semantic search should find the right content regardless of metadata, right?

**What actually happens:** A user asks about the 2024 Q4 earnings report, and retrieval returns chunks from Q2 2023 because the language is similar. Or a multi-tenant app returns chunks from a different customer's documents. Or internal-only information leaks into customer-facing answers.

**What to do instead:** Always store metadata with each chunk: source file, date, category, access level, tenant ID, whatever is relevant. At query time, pre-filter by metadata before doing similarity search. "Find chunks similar to 'revenue growth' WHERE year=2024 AND doc_type='earnings'" is vastly more precise than naked similarity search.

### 4. Stuffing too many chunks into the prompt

**What people do wrong:** Setting top-k to 20 and injecting all 20 chunks into the prompt, reasoning that more context means better answers.

**Why it seems right:** If 5 chunks are good, 20 must be better. The model has a 128K context window, might as well use it.

**What actually happens:** Two problems. First, irrelevant chunks actively confuse the LLM. Research shows that adding irrelevant context degrades answer quality, even when the right context is also present (the "lost in the middle" problem). Second, you burn tokens and money. Each query with 20 large chunks costs significantly more than one with 5 targeted chunks.

**What to do instead:** Start with k=3-5. Add a reranking step if you need better precision. A reranker (like Cohere Rerank or a cross-encoder model) scores each retrieved chunk against the actual query and re-orders them by true relevance. Retrieve 20 with vector search, rerank, keep the top 5.

### 5. Not evaluating retrieval separately from generation

**What people do wrong:** Testing the whole system end-to-end only. "I asked a question and got a bad answer" without diagnosing whether retrieval failed or generation failed.

**Why it seems right:** The user only sees the final answer, so that's what matters.

**What actually happens:** You can't fix what you can't diagnose. If the answer is wrong because retrieval returned irrelevant chunks, no amount of prompt engineering on the generation side will help. If retrieval was perfect but the LLM hallucinated anyway, re-tuning your chunking strategy wastes time.

**What to do instead:** Build separate evaluation for each stage. For retrieval: given a question, are the retrieved chunks actually relevant? Measure retrieval recall and precision. For generation: given the right chunks, does the LLM produce a faithful, complete answer? Use tools like RAGAS, DeepEval, or manual review with a test set of question-answer-source triples.

### 6. Using the wrong embedding model for your content

**What people do wrong:** Defaulting to whatever embedding model the tutorial used without considering the content type, language, or domain.

**Why it seems right:** "OpenAI's embedding model is good at everything."

**What actually happens:** General-purpose embedding models can struggle with highly specialized content (legal documents, medical records, codebases). They may not handle non-English text well. And different models have very different dimension sizes and performance characteristics.

**What to do instead:** Check the MTEB leaderboard for embedding model benchmarks. For English general-purpose, OpenAI's `text-embedding-3-small` or Cohere's models work well. For multilingual, look at models specifically trained on your language. For code, use code-specific embedding models. Test with your actual data: embed 10 queries, check if the top-5 retrieved chunks are actually relevant.

### 7. Skipping the "no answer" case

**What people do wrong:** Building a system that always generates an answer, even when the retrieved chunks don't contain relevant information.

**Why it seems right:** Returning "I don't know" feels like failure. Users want answers.

**What actually happens:** The LLM fills in gaps with its training data (hallucination) or creatively misinterprets tangentially related chunks. The user gets a confident, well-formatted wrong answer. This is worse than no answer because the user trusts it.

**What to do instead:** Explicitly instruct the LLM in your system prompt: "If the provided context doesn't contain enough information to answer the question, say so. Do not make up information." Also set a similarity score threshold, if the best chunk has a cosine similarity below 0.3, skip retrieval entirely and respond that the topic isn't covered in the available documents.

## Best Practices (The Rules That Prevent 90% of Problems)

1. **Use the same embedding model for ingestion and querying.** Vectors from different models live in completely different mathematical spaces. Mixing them means your similarity search is comparing apples to car engines.

2. **Always include overlap between chunks.** A sentence at the end of chunk N might be the setup for the key information at the start of chunk N+1. 10-20% overlap (e.g., 200 tokens for 1000-token chunks) prevents important information from being split across a boundary and lost.

3. **Store metadata with every chunk.** Source document, page number, section heading, date, category. You will need to filter, cite sources, or debug retrieval quality. Metadata makes all three possible.

4. **Set a similarity score threshold.** Don't return chunks that scored below a minimum relevance. Returning irrelevant context is worse than returning no context, because it causes the LLM to hallucinate with false confidence.

5. **Evaluate retrieval and generation independently.** When the final answer is bad, you need to know which stage failed. Build a test set of 20-50 question-answer pairs and check retrieval accuracy separately from answer accuracy.

6. **Handle document updates from day one.** Track which chunks came from which source document. When the source changes, delete old chunks and re-ingest. Stale data is worse than no data because it's confidently wrong.

7. **Instruct the LLM to refuse when context is insufficient.** Your system prompt must tell the model to say "I don't have information about that" rather than guessing. Test that it actually does this by asking questions outside your document scope.

8. **Start simple, add complexity only when measurements show you need it.** Basic RAG (chunk, embed, retrieve, generate) works surprisingly well. Don't add reranking, hybrid search, or agentic retrieval until you've measured that basic retrieval is the bottleneck. Most RAG problems are chunking problems, not architecture problems.

## The "Just Tell Me What to Do" Quickstart

This gets a working RAG system running in under 10 minutes. You'll chat with a folder of Markdown files using LangChain, Chroma, and OpenAI.

**Prerequisites:** Python 3.10+, an OpenAI API key.

```bash
# Create project
mkdir rag-quickstart && cd rag-quickstart

# Set up virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install langchain langchain-chroma langchain-openai langchain-community

# Set your API key
export OPENAI_API_KEY="sk-your-key-here"

# Create a docs folder with some sample content
mkdir docs
```

Create a couple sample markdown files to test with:

```bash
cat > docs/refund-policy.md << 'EOF'
# Refund Policy

All purchases are eligible for a full refund within 30 days of purchase.
After 30 days, we offer store credit only.

To request a refund, email support@example.com with your order number.
Refunds are processed within 5-7 business days.

Enterprise customers on annual plans can request a prorated refund
at any time during their subscription period.
EOF

cat > docs/pricing.md << 'EOF'
# Pricing

## Free Tier
- Up to 100 requests per month
- Basic support via email
- 1 user

## Pro Plan - $29/month
- Unlimited requests
- Priority support
- Up to 5 users
- API access

## Enterprise - Custom pricing
- Dedicated infrastructure
- SLA guarantees
- Unlimited users
- Custom integrations
- Dedicated account manager
EOF
```

Now create the RAG app:

```bash
cat > app.py << 'PYEOF'
from langchain_community.document_loaders import DirectoryLoader, TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 1. Load documents
loader = DirectoryLoader("./docs", glob="**/*.md", loader_cls=TextLoader)
docs = loader.load()
print(f"Loaded {len(docs)} documents")

# 2. Split into chunks
splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=100)
chunks = splitter.split_documents(docs)
print(f"Split into {len(chunks)} chunks")

# 3. Create vector store
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=OpenAIEmbeddings(model="text-embedding-3-small"),
    persist_directory="./chroma_db"
)
print("Vector store created")

# 4. Set up retrieval chain
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

prompt = ChatPromptTemplate.from_template("""Answer the question based only on the following context.
If the context doesn't contain enough information, say "I don't have information about that in the available documents."

Context:
{context}

Question: {question}

Answer:""")

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

def format_docs(docs):
    return "\n\n---\n\n".join(
        f"[Source: {doc.metadata.get('source', 'unknown')}]\n{doc.page_content}"
        for doc in docs
    )

chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# 5. Interactive loop
print("\nRAG system ready. Ask questions about your docs (type 'quit' to exit):\n")
while True:
    question = input("You: ").strip()
    if question.lower() in ("quit", "exit", "q"):
        break
    if not question:
        continue

    answer = chain.invoke(question)
    print(f"\nAssistant: {answer}\n")
PYEOF
```

Run it:

```bash
python app.py
```

Try these questions:
- "What's your refund policy for enterprise customers?"
- "How much does the Pro plan cost?"
- "Do you offer phone support?" (should say it doesn't have that info)

## How to Prompt AI Tools About This

### Context to give Claude/Cursor when building RAG

Tell the AI:
- What kind of documents you're working with (PDFs, markdown, HTML, database records)
- How large your document collection is (10 files vs 10,000)
- What kinds of questions users will ask (specific factual vs open-ended analytical)
- Whether you need multi-turn conversation or single-shot Q&A
- Your hosting constraints (local vs cloud, budget for API calls)

### Example prompts that produce good results

```
"Build a RAG pipeline using LangChain and Chroma that ingests a folder of
markdown files, chunks them at 800 tokens with 150 token overlap using
RecursiveCharacterTextSplitter, embeds with OpenAI text-embedding-3-small,
and answers questions with GPT-4o-mini. Include source citations in answers
and handle the case where no relevant context is found."
```

```
"I have a RAG system that returns irrelevant chunks. My documents are
technical API docs with code examples. Current setup: chunk_size=1000,
no overlap, using text-embedding-ada-002. Suggest improvements to
chunking strategy, embedding model choice, and retrieval configuration."
```

```
"Add a reranking step to my RAG retriever. Retrieve top-20 from Chroma,
rerank with Cohere rerank-v3.5, keep top-5. Show me the code using
LangChain's ContextualCompressionRetriever."
```

### What to watch out for in AI-generated RAG code

- **Deprecated LangChain imports.** LangChain restructured its packages in 2024. AI tools trained on older data will use `from langchain.vectorstores import Chroma` instead of `from langchain_chroma import Chroma`. Check that imports use the `langchain_*` partner packages.
- **Missing persist.** AI-generated code often creates an in-memory vector store that disappears when the script ends. Make sure `persist_directory` is set if you're using Chroma.
- **Hardcoded API keys.** AI loves putting `api_key="sk-..."` directly in the code. Always use environment variables.
- **No error handling for empty retrieval.** The AI rarely adds logic for what happens when zero chunks are retrieved or all chunks score below a relevance threshold.
- **Chunk size in characters vs tokens.** LangChain's `chunk_size` parameter is in characters by default, not tokens. AI-generated code often treats the number as if it's tokens. 1000 characters is roughly 200-250 tokens.

### Key terms that improve output quality

Use these in your prompts for more precise results: "RecursiveCharacterTextSplitter" (not just "split"), "cosine similarity threshold" (not "relevance"), "retrieval recall" (when discussing evaluation), "cross-encoder reranking" (for precision), "parent document retriever" (for maintaining context), "metadata filtering" (not "tagging").

## Ship It: Build This

**What you're building:** A local knowledge base app that lets you chat with a folder of your own markdown files. Drop any collection of `.md` files into a folder (project docs, meeting notes, a wiki export, your personal knowledge base), and get a command-line chat interface that answers questions with source citations. It handles document updates, shows relevance scores, and gracefully refuses to answer when the content doesn't cover the question.

**Why this project specifically:** It exercises every important part of the RAG pipeline: document loading, chunking with overlap, embedding, vector storage, retrieval with metadata, prompt construction with source attribution, and the crucial "I don't know" case. It also handles the document update problem that most tutorials skip.

**Rough architecture:**
- LangChain for orchestration
- Chroma for local vector storage (no cloud services needed)
- OpenAI embeddings and GPT-4o-mini for generation (or swap for any provider)
- File hash tracking to detect document changes and re-ingest only what changed
- Metadata filtering to show source file and chunk position in citations
- Similarity score threshold to skip retrieval when nothing is relevant

**Estimated time:** 2-3 hours for a vibe coder using AI tools. The quickstart above gets you 60% of the way there. The remaining work is adding document change detection, proper source citations, score thresholds, and a cleaner interface.

## Go Deeper (Optional)

**Learn multimodal RAG** when you need to search across images, tables, and diagrams, not just text. Relevant for technical documentation, research papers, or any content where key information lives in figures.

**Learn GraphRAG** when your data has complex relationships (org charts, knowledge graphs, interconnected entities) and questions require reasoning across multiple connected documents. Microsoft's GraphRAG approach builds a knowledge graph from your documents and uses community summaries for broader analytical questions.

**Learn fine-tuning your embedding model** when off-the-shelf embeddings don't capture domain-specific similarity well enough. Common in legal, medical, or highly technical domains where general models miss nuances.

**Learn query routing and decomposition** when your system has multiple knowledge bases or your questions are complex enough to require multiple retrieval steps. The LLM decomposes "Compare X with Y" into separate retrievals and synthesizes results.

**Learn production RAG observability** when you're deploying to real users. Tools like LangSmith, Phoenix (Arize), or Langfuse let you trace every retrieval and generation step, spot failures, and monitor quality over time.
