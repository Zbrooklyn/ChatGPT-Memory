# ChatGPT Memory — Master Source of Truth

**Version:** 1.0  
**Last verified:** September 8, 2026  
**Scope:** ChatGPT consumer chat-mode memory  
**Purpose:** Maintain one authoritative record of what is known, what is historically documented, what is inferred, and what remains unknown about ChatGPT memory.

---

# 1. Evidence Standard

Every claim belongs to one of four classes.

### CONFIRMED CURRENT
Explicitly documented by OpenAI as part of the current ChatGPT memory system.

### CONFIRMED HISTORICAL
Documented by OpenAI patents, older product documentation, or earlier architectures, but not proven to remain unchanged in the current implementation.

### INFERRED
Strongly suggested by documented behavior and architecture but not explicitly confirmed by OpenAI.

### UNKNOWN
Implementation detail for which there is currently insufficient public evidence.

A patent proves that OpenAI designed or contemplated an architecture. It does **not** prove that Dreaming V3 currently implements that architecture exactly.

---

# 2. Executive Conclusion

The current ChatGPT memory system is **not simply saved facts, RAG over old chats, a vector database, or an enormous prompt containing previous conversations**.

The public evidence establishes at least two major capabilities:

1. **Background memory synthesis** through OpenAI's Dreaming architecture.
2. **Retrieval/search of historical conversations and other eligible sources** when relevant to the current interaction.

OpenAI describes current memory as a:

> “continually updated synthesis of context from your past chats”

and describes Dreaming as a background process capable of learning across many conversations and synthesizing ChatGPT's memory state.

Separately, OpenAI says ChatGPT searches past conversations to find relevant context and can pull information from past chats, saved memories and, where available, files and connected Gmail.

The exact internal relationship between Dreaming and runtime historical retrieval has not been disclosed.

---

# 3. Current Architecture — What We Can Prove

Conceptually:

```text
                     USER HISTORY / EVIDENCE
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
     Past chats         Saved memories             Files
        │                      │                      │
        └───────── eligible connected sources ──────┘
                               │
                   ┌───────────┴───────────┐
                   │                       │
                   ▼                       ▼
              DREAMING              HISTORICAL
             BACKGROUND              RETRIEVAL
             SYNTHESIS                / SEARCH
                   │                       │
                   ▼                       │
          synthesized memory               │
                state                     │
                   │                       │
                   └──────────┬────────────┘
                              │
                current conversation +
              instructions/preferences
                              │
                              ▼
                       CHATGPT MODEL
                              │
                              ▼
                           RESPONSE
```

**Status:** The individual capabilities are confirmed. Their exact internal orchestration is inferred.

---

# 4. Evolution of ChatGPT Memory

## April 2024 — Saved Memories

**CONFIRMED CURRENT/HISTORICAL**

The original memory system operated much more like an assistant's notepad.

Saved memories were written during conversations and strongly depended on explicit signals such as:

“Remember that…”

They were useful but incomplete and could become stale.

The legacy saved-memory system still exists as an alternative memory mode.

Saved memories are stored separately from chat history and are considered when generating future responses unless removed.

---

## April 2025 — Dreaming V0

**CONFIRMED HISTORICAL**

OpenAI introduced the first version of **Dreaming**.

Unlike Saved Memories, Dreaming:

- operates as a background process;
- references chat history;
- learns from many conversations;
- automatically curates memory;
- can learn information that wasn't accompanied by an explicit “remember this” request;
- synthesizes ChatGPT's memory state.

OpenAI says Dreaming V0 supplemented Saved Memories and produced a substantial improvement, but wasn't sufficient as a standalone memory system.

---

## June 4, 2026 — Dreaming V3

**CONFIRMED CURRENT**

OpenAI launched what it calls:

**Dreaming V3**

OpenAI describes it as a substantially more capable and compute-efficient memory architecture built on Dreaming.

Its design goals are:

1. Carry useful context forward.
2. Follow preferences and constraints.
3. Stay current as circumstances change.

OpenAI designed it for:

- hundreds of millions of users;
- multi-year histories;
- improved correctness;
- reduced staleness;
- better scalability.

Dreaming is now described by OpenAI as the shared memory foundation going forward.

---

# 5. Memory Is Synthesized State

**CONFIRMED CURRENT**

Current ChatGPT memory is explicitly described as a continually updated **synthesis** rather than a list of individual observations.

This is critical.

Conceptually:

```text
Historical evidence
       ↓
Memory synthesis
       ↓
Current understanding
       ↓
New evidence / time passes
       ↓
Reconciliation
       ↓
Revised understanding
```

This means the synthesized memory should be understood as **derived state**.

The visible Memory Summary is therefore not equivalent to the complete underlying memory.

OpenAI explicitly says the Memory Summary may contain less information than the underlying synthesis.

---

# 6. The Memory Summary Is Not the Memory Database

**CONFIRMED CURRENT**

The Memory Summary is a human-readable management interface over the system's memory.

OpenAI says:

- it provides a high-level view;
- the underlying synthesis can be broader;
- less relevant information may be omitted;
- some information may be inappropriate to display there;
- it populates as sufficient history accumulates;
- it can be refreshed.

Therefore:

```text
Memory Summary ≠ full memory
Memory Summary ≠ complete retrieved context
Memory Summary ≠ raw history
```

---

# 7. Runtime Historical Retrieval Exists

**CONFIRMED CURRENT**

On May 5, 2026, OpenAI stated that ChatGPT had improved its ability to:

- pull relevant context from previous chats;
- use Saved Memories;
- reference files where available;
- reference connected Gmail where available;
- search past conversations faster to locate appropriate context.

This matters because background Dreaming alone cannot explain all memory behavior.

We therefore know the product contains both:

```text
SYNTHESIS:
many experiences
      ↓
persistent generalized understanding

AND

RETRIEVAL:
current problem
      ↓
find relevant historical experience
```

The precise implementation of retrieval is unknown.

---

# 8. Semantic vs. Episodic Memory

**INFERRED TERMINOLOGY — HIGH CONFIDENCE**

A useful conceptual model is:

### Semantic/user memory

Knowledge generalized across interactions:

- preferences;
- constraints;
- ongoing projects;
- working style;
- stable facts;
- current situation.

Likely heavily associated with Dreaming.

### Episodic memory

Specific prior experiences:

- a particular conversation;
- a particular project decision;
- why a previous approach failed;
- what happened during an earlier task.

Likely associated with historical conversation retrieval.

OpenAI does not currently use “semantic memory” and “episodic memory” as official Dreaming V3 terminology.

The distinction nevertheless matches the documented capabilities extremely well.

---

# 9. Time Is a First-Class Memory Problem

**CONFIRMED CURRENT**

Dreaming is designed to revise information as time passes.

OpenAI's example:

```text
Before trip:
"You're going to Singapore in July."

Later:
"You went to Singapore in July 2026."
```

This establishes that memory is not simply an append-only facts database.

It can reinterpret state based on temporal change.

A serious reproduction of this architecture therefore needs some concept of:

- when something became true;
- whether it is currently true;
- whether it is an event;
- whether it is expected to expire;
- whether it should become historical rather than disappear.

The exact internal temporal representation is unknown.

---

# 10. Memory Uses Importance / Relevance

**CONFIRMED CURRENT**

OpenAI says the system tracks information it determines is relevant or important enough to retain.

The Memory Summary can also omit less relevant information.

Therefore some form of **salience selection** exists.

Exact implementation is unknown.

Possibilities include:

- learned importance;
- recency;
- recurrence;
- explicit user emphasis;
- current-task relevance;
- retrieval frequency;
- confidence;
- source reliability;
- temporal validity.

No exact weighting formula has been disclosed.

---

# 11. OpenAI Trains Models Specifically to Use Memory

**CONFIRMED CURRENT**

OpenAI has a dedicated **Personalization-Memory** research team.

Its published role description says the team develops general-purpose memory and personalization capabilities across ChatGPT and other agentic products.

Its research includes:

- reinforcement learning;
- dataset creation;
- evaluations;
- post-training;
- user signals;
- human data;
- frontier-model memory usage.

This establishes an important architectural principle:

**Good ChatGPT memory is not merely a storage/retrieval problem.**

The answering model itself is being trained to make better decisions about memory.

Those decisions include conceptually:

```text
Should this information matter?
Is it still valid?
Should I use it here?
Would mentioning it be inappropriate?
How strongly should it constrain my answer?
Does new evidence supersede it?
```

---

# 12. Historical OpenAI Patent

**CONFIRMED HISTORICAL**

OpenAI OpCo filed U.S. patent application:

**US20250200361A1 — Selective learning of information for the generation of personalized responses by a generative response engine**

Priority date: December 13, 2023  
Filed: June 3, 2024  
Published: June 19, 2025

This is currently the deepest publicly available implementation-level evidence about OpenAI's memory architecture.

It predates Dreaming V3.

It must therefore be treated as **architectural ancestry**, not a specification of the current production backend.

---

# 13. Patent Architecture: Personalization Notepad

**CONFIRMED HISTORICAL**

The patent describes a compact **personalization notepad**.

The model can:

1. detect candidate information;
2. decide whether the information deserves retention;
3. write appropriate information into the notepad;
4. use the notepad to influence future responses.

The system was intentionally selective because remembering everything would:

- consume too much space;
- degrade performance;
- potentially create an uncomfortable user experience;
- make memory harder for the user to inspect.

This strongly resembles the 2024 Saved Memories architecture.

---

# 14. Learned Memory Selection

**CONFIRMED HISTORICAL**

The patent says OpenAI contemplated training the generative model itself to decide whether information should be stored.

It describes:

- reinforcement learning;
- probability/scoring of candidate memories;
- reward functions;
- external scores;
- human labelers;
- heuristics combined with learned behavior.

This is important because it establishes that OpenAI's memory philosophy was never merely:

```text
extract every noun and preference
```

Instead:

```text
candidate information
       ↓
learned judgment
       ↓
worth remembering?
       ↓
yes / no
```

Current OpenAI research roles show that reinforcement learning and post-training remain central to personalization/memory research.

Whether Dreaming V3 uses the patent's exact classifier/scoring implementation is unknown.

---

# 15. Historical Asynchronous Consolidation

**CONFIRMED HISTORICAL**

The patent describes **offline asynchronous consolidation**.

Related memory notes can be:

- identified;
- combined;
- rewritten;
- deduplicated;
- deleted when no longer needed.

The process may run repeatedly or when memory approaches a configured capacity.

The patent explicitly describes it as generally occurring offline without user interaction.

This is strikingly similar in principle to what OpenAI later publicly named **Dreaming**.

Likely lineage:

```text
2023/24:
offline asynchronous memory consolidation

        ↓

2025:
Dreaming V0

        ↓

2026:
Dreaming V3
```

**The lineage is inferred. The individual systems are documented.**

---

# 16. Historical Memory Retention / Reinforcement

**CONFIRMED HISTORICAL**

Under memory-pressure conditions, the patent describes giving priority to:

- newer notes;
- recently accessed notes.

Older memories that remain useful could be reordered or effectively refreshed to avoid deletion.

Conceptually:

```text
retention value
     =
recency
+
continued usefulness/access
```

We do **not** know whether Dreaming V3 retains this exact mechanism.

---

# 17. Historical “Deep Memory”

**CONFIRMED HISTORICAL — VERY IMPORTANT**

The patent explicitly describes another memory layer separate from the personalization notepad:

**deep memory**

OpenAI describes the personalization notepad as small and human-readable.

Deep memory can contain much more data and need not be human-readable.

The purpose is specifically to remember **past threads and projects**.

The patent describes:

```text
conversation/session
       ↓
identify topics
       ↓
store topics over time
       ↓
process topics into embeddings
       ↓
associate embeddings with user account
       ↓
new prompt
       ↓
retrieve relevant embeddings
       ↓
use them to influence response
```

OpenAI describes the distinction conceptually as:

```text
Personalization notepad
≈ assistant remembers information about you

Deep memory
≈ assistant remembers projects it previously worked on with you
```

This is our strongest public evidence that OpenAI's memory research deliberately separated **user knowledge** from **past-interaction/project knowledge**.

---

# 18. Current Retrieval vs Historical Deep Memory

We have two independent pieces of evidence.

### Historical patent

```text
past sessions
→ topics
→ embeddings
→ relevant historical retrieval
```

### Current ChatGPT

```text
past conversations
→ search
→ relevant historical context
```

**INFERENCE — HIGH CONFIDENCE**

Current past-chat retrieval likely evolved from or shares architectural ideas with OpenAI's earlier deep-memory work.

**UNKNOWN**

We cannot claim the current system still:

- extracts “topics” exactly as described;
- stores the same embeddings;
- uses the same embedding model;
- uses the same index;
- retrieves the same representation.

---

# 19. Memory Capacity Is Finite

**CONFIRMED CURRENT**

OpenAI reports increasing memory capacity for Plus and Pro users as part of the Dreaming V3 rollout.

It also reduced the compute necessary to serve Dreaming to Free users by approximately **5×**.

**UNKNOWN**

OpenAI has not publicly defined “memory capacity” numerically.

It might refer to:

- synthesized-state size;
- number of memories;
- storage;
- retrieval index;
- context budget;
- historical coverage;
- compute budget;
- some combination of these.

Do not state a token count or record count without new evidence.

---

# 20. Multiple Source Types Contribute

**CONFIRMED CURRENT**

Depending on product configuration and availability, personalization can draw context from:

- past chats;
- Saved Memories;
- files;
- connected Gmail;
- explicit instructions/custom instructions.

The exact source set varies by product, account, permissions, and available integrations.

---

# 21. Provenance Exists

**CONFIRMED CURRENT**

ChatGPT exposes **Memory Sources** so users can see some information that contributed to personalization.

However, the visible sources should not be interpreted as a complete dump of everything internally considered.

Therefore:

```text
visible provenance
    ⊆
actual context/evidence considered
```

The exact internal provenance representation is unknown.

---

# 22. Legacy Saved Memories Are Separate

**CONFIRMED CURRENT**

Saved Memories remain a distinct legacy system.

OpenAI says the Saved Memory “notepad” is stored separately from chat history.

Deleting a conversation therefore does not necessarily eliminate a Saved Memory originally derived from it.

OpenAI may retain logs of deleted Saved Memories temporarily for safety/debugging.

The current improved-memory experience and the legacy Saved Memories experience should not be treated as architecturally identical.

---

# 23. Memory Is Not Just Retrieval

This conclusion is now strongly established.

A pure retrieval system could find:

> “Eddie said X six months ago.”

But it would struggle when:

- X later became false;
- multiple conversations collectively imply Y;
- an old preference was replaced;
- a project changed phase;
- a planned event occurred;
- several observations need to become one stable understanding.

Dreaming addresses those problems through consolidation and state synthesis.

---

# 24. Memory Is Not Just Synthesis Either

A single synthesized profile could know:

> “The user works on long-running AI infrastructure projects.”

But it could lose details such as:

> “In the September 7 Discord architecture review, this particular mechanism was rejected because of a scheduler/idempotency issue.”

Runtime historical retrieval addresses that problem.

Therefore the strongest current model is:

```text
SYNTHESIZED STATE
      +
EPISODIC/HISTORICAL RETRIEVAL
      +
CURRENT CONVERSATION
      +
EXPLICIT INSTRUCTIONS
      +
MODEL TRAINED TO USE ALL OF THEM
```

---

# 25. Likely Current Architecture

**INFERENCE — NOT SOURCE TRUTH**

Based on all public evidence, the architecture I would currently bet on is:

```text
                   RAW LONG-TERM HISTORY
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
     HISTORICAL INDEX               DREAMING
      / RETRIEVAL                  CONSOLIDATION
             │                           │
             │                    CURRENT USER STATE
             │                    ├─ facts
             │                    ├─ preferences
             │                    ├─ constraints
             │                    ├─ projects
             │                    ├─ temporal state
             │                    └─ learned patterns
             │                           │
             └─────────────┬─────────────┘
                           │
                       relevance
                       selection
                           │
                current conversation
                           │
                           ▼
                  POST-TRAINED MODEL
                           │
                 appropriateness/use
                       decision
                           │
                           ▼
                        RESPONSE
```

This diagram is a reconstruction, not an OpenAI-published implementation diagram.

---

# 26. The Most Important Architectural Principle

The biggest lesson from Dreaming is:

**Do not treat memory as the original source of truth.**

Use historical evidence as the durable source.

Treat memory as a continuously maintained interpretation of that evidence.

Conceptually:

```text
SOURCE EVIDENCE
chats / files / interactions
       │
       ▼
DERIVED MEMORY STATE
       │
       ▼
new evidence / time / corrections
       │
       ▼
RECONCILIATION
       │
       ▼
UPDATED MEMORY STATE
```

This allows the system to correct itself rather than endlessly accumulating contradictory facts.

---

# 27. Why ChatGPT Memory Feels Better Than Typical Agent Memory

Most agent-memory implementations approximate:

```text
conversation
→ chunk
→ embed
→ vector DB
→ similarity search
```

ChatGPT appears to combine substantially more:

```text
persistent history
        +
learned salience
        +
background consolidation
        +
synthesized user state
        +
historical retrieval
        +
temporal reconciliation
        +
explicit preferences/constraints
        +
scope/access control
        +
provenance
        +
user corrections
        +
frontier-model post-training
```

The retrieval database alone is therefore unlikely to explain ChatGPT's memory quality.

---

# 28. What We Still Do Not Know

The following remain **UNKNOWN** unless new evidence appears:

### Storage
- Dreaming V3 database technology
- database schema
- number of records per user
- whether a graph representation exists
- whether synthesized state is prose, structured data, embeddings, model state, or hybrid

### Retrieval
- current embedding model
- vector-database technology
- chunking strategy
- whether retrieval is message-, topic-, summary-, conversation-, or multi-level
- top-K
- reranker
- scoring formula
- query generation
- retrieval token budget

### Dreaming
- exact model used
- exact prompts
- trigger schedule
- frequency
- whether processing is incremental
- whether idle compute is used
- Dreaming V1 architecture
- Dreaming V2 architecture
- precise V3 differences

### State management
- conflict-resolution algorithm
- confidence scores
- temporal representation
- expiration rules
- salience formula
- decay algorithm
- reinforcement-through-access behavior in current V3
- provenance schema

### Runtime
- exact amount of memory inserted into a prompt
- whether synthesized state is retrieved selectively or loaded broadly
- exact ordering of memory/instructions/history
- how retrieved episodes and Dreaming output are merged
- whether multiple context candidates are generated/ranked

### Capacity
- exact Free memory capacity
- exact Plus memory capacity
- exact Pro memory capacity
- whether “capacity” means tokens, records, bytes, retrieval budget, or another measure

### Infrastructure
- physical storage technology
- indexing infrastructure
- refresh pipeline
- queue architecture
- background-compute architecture

### Code
- Dreaming V3 source code
- production system prompts
- public Dreaming API

Do not fill these gaps with assumptions.

---

# 29. Source Hierarchy

When two claims conflict, use this priority:

### Tier 1 — Current primary OpenAI product/research documentation

**Dreaming: Better memory for a more helpful ChatGPT**  
OpenAI, June 4, 2026.  
https://openai.com/index/chatgpt-memory-dreaming/

**Current Memory FAQ**  
OpenAI Help Center.  
https://help.openai.com/en/articles/8590148

**ChatGPT Release Notes**  
OpenAI Help Center.  
https://help.openai.com/en/articles/6825453-chatgpt-release-notes

### Tier 2 — Current OpenAI organizational/research evidence

**Personalization-Memory team**  
Documents RL, datasets, evaluations and post-training work on memory/personalization.  
https://openai.com/careers/research-engineer-research-scientist-personal-agi-personalization-san-francisco/

### Tier 3 — OpenAI patents

**US20250200361A1**  
Provides valuable implementation-level historical architecture including selective memory, asynchronous consolidation and deep memory.  
https://patents.google.com/patent/US20250200361A1/en

Patents never override current product documentation.

### Tier 4 — External experiments/reverse engineering

Useful only when reproducible.

Must be labeled observational rather than official.

### Tier 5 — Community speculation

Never promote to source truth without independent evidence.

---

# 30. Change-Control Rule

This document is the canonical memory research record.

New discoveries should never silently overwrite prior conclusions.

Every update should record:

```text
date
source
new evidence
confidence class
what changed
what previous belief it replaces
remaining uncertainty
```

If evidence contradicts the current document:

1. preserve the old claim;
2. identify the contradiction;
3. determine which source has greater authority;
4. revise the conclusion;
5. record the change.

---

# 31. Current Bottom Line

As of September 8, 2026, the best evidence supports this conclusion:

**ChatGPT memory is a learned long-term personalization system built around Dreaming-based background synthesis, historical-context retrieval, temporal reconciliation, relevance selection, explicit user controls, and frontier-model post-training.**

OpenAI's earlier patent additionally demonstrates that its memory research explicitly explored:

- a compact human-readable personalization notepad;
- learned selection of what deserves memory;
- reinforcement learning and human feedback;
- offline asynchronous consolidation;
- merging and deleting memories;
- recency/access-based retention;
- a separate deep-memory system;
- session/topic embeddings;
- retrieval of relevant past projects.

Those historical mechanisms should be considered important architectural ancestry but **must not be represented as confirmed Dreaming V3 internals unless OpenAI supplies further evidence**.

The central design principle is:

> **History is evidence. Memory is synthesized state. Retrieval recovers episodes. The model decides how to use both.**

That is our current source of truth.
