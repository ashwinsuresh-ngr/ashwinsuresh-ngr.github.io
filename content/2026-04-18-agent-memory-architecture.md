Title: Agent Memory Architecture
Date: 2026-04-18
Category: GenAI
Tags: GenAI, AI-agents, memory, architecture, LLM
Slug: agent-memory-architecture

An agent working through a long, multi-step task needs to remember what it's already tried, what it's learned, and what still needs doing — but an LLM's context window, as covered in the tokens post earlier in this series, is finite. Agent memory architecture is the set of design patterns for managing what an agent remembers, how, and for how long, so it can operate coherently across tasks that outgrow what fits in a single prompt. Here's how it actually works.

## Why Memory Is a Genuinely Hard Problem for Agents

As covered in the training vs. inference post, an LLM doesn't learn or retain anything between calls on its own — every relevant piece of context has to be explicitly included in the prompt each time. For a single-turn interaction, that's straightforward. For an agent running dozens of steps across a long task, naively including the entire history of everything that's happened quickly exceeds the context window, wastes tokens and cost, and — as covered in the context engineering post — can actually degrade output quality if the model has to sift through excessive, low-relevance detail to find what actually matters.

## The Main Types of Agent Memory

**Working memory (short-term).** The immediate context an agent needs for its current step — the current sub-task, the most recent tool results, the plan it's actively executing. This typically lives directly in the prompt for each step and is naturally bounded by the context window.

**Episodic memory.** A record of what happened during the current task or session — actions taken, results observed, decisions made — letting the agent avoid repeating failed approaches or losing track of progress across a long sequence of steps.

**Long-term memory.** Information that persists across separate sessions or tasks entirely — user preferences, prior interactions, learned facts about a specific domain or user — allowing an agent to "remember" a person or context from a previous conversation, not just within the current one.

**Semantic memory.** A structured knowledge base the agent can query — often implemented using the vector search and RAG techniques covered earlier in this series — letting the agent retrieve relevant facts or documents on demand rather than needing everything pre-loaded into its context.

## Common Architectural Patterns

**Sliding window.** Keeping only the most recent N steps or messages in context, discarding older ones. Simple to implement, but risks losing important information from earlier in a long task if it falls outside the window.

**Summarization.** Periodically condensing older parts of a task's history into a compact summary, preserving the important decisions and outcomes while freeing up context space — directly connecting to the "conversation history management" concept covered in the context engineering post.

**Structured state tracking.** Rather than keeping a raw log of everything that happened, maintaining an explicit, structured representation of the task's current state — what's been completed, what remains, key facts gathered so far — and regenerating the relevant prompt context from that structured state each step, rather than replaying the full history.

**Retrieval-based memory.** Storing episodic or long-term information externally (often as embeddings in a vector database, as covered in the RAG with MongoDB post) and retrieving only the specific pieces relevant to the current step, rather than keeping everything in active context at once.

## A Practical Example

Consider an agent working through a multi-day research task, checking in across several separate sessions:

- **Within a single session**, working memory holds the current sub-task and recent tool results directly in context.
- **Across the session**, episodic memory tracks what's been researched so far, what leads turned out to be dead ends, and what's still open — likely maintained as structured state rather than raw conversation history.
- **Across sessions**, long-term memory persists the overall research goal, key findings so far, and user preferences about format or depth — retrieved and reintroduced into context at the start of each new session.
- **Throughout**, semantic memory — a vector store of retrieved documents — lets the agent pull in specific source material relevant to whatever it's currently investigating, without needing every document permanently in context.

## Trade-Offs in Memory Design

**Completeness vs. context budget.** More retained memory generally improves an agent's coherence and avoids redundant work, but costs more tokens (and money, and latency) per step — the same token-cost trade-offs covered in the tokens and controlling-responses posts apply directly here.

**Precision vs. summarization loss.** Summarizing history saves space but risks losing specific details that later turn out to matter — a genuine trade-off with no universally correct answer, tuned based on how much precision a given task actually needs.

**Freshness vs. staleness.** Long-term memory that isn't periodically reviewed or updated can drift out of sync with reality — a stored user preference or fact that's since changed but never gets refreshed can actively mislead an agent's future decisions.

## Why This Connects Directly to Reliability

As covered in the building reliable prompts post, a well-engineered agent isn't just about a good prompt at each individual step — it's about the entire system, including what information is actually available to the model when it makes each decision. Poor memory architecture is a common, underappreciated source of agent failures: an agent repeating a failed approach because it forgot trying it, or losing track of the overall goal partway through a long task, is usually a memory design problem, not a reasoning problem.

## The Bottom Line

Agent memory architecture is about deliberately deciding what an agent remembers, at what level of detail, and for how long — balancing the coherence and reliability benefits of retained context against the real constraints of context window size, cost, and the risk that too much undifferentiated history actually degrades decision quality. Working memory, episodic memory, long-term memory, and retrieval-based semantic memory each serve a different role, and combining them deliberately — rather than either naively keeping everything or discarding too much — is what lets an agent operate coherently across tasks that span far more than a single prompt can hold.
