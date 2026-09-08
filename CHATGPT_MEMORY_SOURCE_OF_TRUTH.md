# ChatGPT Memory — Master Source of Truth

**Version:** 2.2  
**Last verified:** September 8, 2026  
**Supersedes:** Version 2.1  
**Scope:** ChatGPT consumer chat-mode memory, plus directly relevant OpenAI system designs, historical implementations, adjacent OpenAI memory implementations, and empirical/client observations.  
**Purpose:** Maintain one canonical, evidence-graded account of what is publicly known about ChatGPT memory, what can reasonably be inferred, and what remains unknown.

---

# 1. Bottom Line

The strongest public evidence supports this conclusion:

**ChatGPT memory is not one memory store. It is a layered personalization architecture combining background synthesis of long-term user state, runtime retrieval of specific historical interactions, persistent instructions/memories, scope and permission controls, relevance gating, source provenance, and models specifically post-trained to use those signals.**

OpenAI's current Dreaming documentation establishes a background process that learns across many conversations and **synthesizes ChatGPT's memory state**. Separately, current product documentation establishes that ChatGPT **searches past conversations** and retrieves relevant historical context. Recent OpenAI patents independently describe essentially the same hybrid: **synthesized concepts from past conversation threads + searchable summaries/indexes of past interactions**.

Current best reconstruction:

```text
                         DURABLE EVIDENCE
         chats | files | apps | memories | instructions
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
       SYNTHESIS / DREAMING          HISTORICAL INDEX
       generalized current           specific previous
          user state                   interactions
                │                           │
                │                    search / retrieval
                │                           │
                └─────────────┬─────────────┘
                              │
                      PERSONALIZATION
                          STATE
                              │
                 SCOPE / ACCESS CONTROL
                              │
                       RELEVANCE GATE
                              │
                       CONTEXT PACKAGE
                              │
                     POST-TRAINED MODEL
                              │
                           RESPONSE
                              │
                    provenance / feedback
                              │
                       future evidence
```

**The complete diagram is a reconstruction. Its major individual components are independently supported by current OpenAI documentation and contemporary OpenAI system designs.**

---

# 2. Evidence Standard

Every claim in this document belongs to one of these classes.

## A — Current Product Fact

Explicitly documented in current OpenAI ChatGPT documentation, release notes, or product announcements.

This is the highest authority for present ChatGPT behavior.

## B — Contemporary OpenAI System Design

Recent OpenAI patents or technical system descriptions closely corresponding to current functionality.

Strong architectural evidence, but a patent does not prove every described embodiment runs in production.

## C — Historical OpenAI Design

Earlier OpenAI patents, product generations, or implementation descriptions.

Useful for lineage, but not automatically current.

## D — OpenAI Adjacent Implementation

Current public OpenAI memory implementations outside ChatGPT itself, primarily Codex.

These reveal OpenAI engineering patterns but do not prove identical ChatGPT code.

## E — Empirical / Client Observation

Reproducible studies, exports, frontend/network artifacts, client reverse engineering, direct interviews hosted outside OpenAI, or black-box observations.

Useful, but subordinate to current OpenAI documentation.

## F — Inference

Architectural conclusions formed by combining evidence.

Must remain labeled as inference.

## U — Unknown

No sufficient public evidence.

Unknowns must not be filled with plausible assumptions.

---

# 3. Source Precedence

When sources conflict, use this order:

1. Newer, feature-specific current OpenAI documentation.
2. Current OpenAI general documentation.
3. Contemporary OpenAI system design/patents.
4. Older OpenAI documentation.
5. Historical OpenAI patents/designs.
6. Current adjacent OpenAI implementations.
7. Direct interviews with OpenAI personnel hosted externally.
8. Empirical/client observations.
9. Community speculation.

**Freshness matters even among official OpenAI sources.**

For example, early 2026 Health documentation described stronger compartmentalization than later 2026 Health documentation. The newer/current Health documentation controls current-state conclusions.

---

# 4. Product Evolution

## 2024 — Saved Memories

**Class: A/C**

The first persistent-memory system operated broadly like a user-specific notepad. Saved memories could contain facts or preferences ChatGPT judged useful for future conversations.

The current Memory FAQ still exposes legacy Saved Memories. OpenAI describes saved memories as separate from chat history and available in future responses until removed.

Conceptually:

```text
conversation
    ↓
selected useful information
    ↓
saved-memory notepad
    ↓
future conversations
```

This was not the final architecture.

## 2025 — Saved Memories + Dreaming V0

**Class: A/C**

OpenAI says Dreaming V0 supplemented Saved Memories.

Dreaming introduced a different mechanism: a **background process** that could learn across many conversations and synthesize a memory state instead of relying entirely on discrete explicit memory entries.

OpenAI's published evolution is:

```text
2024 — Saved Memories
2025 — Saved Memories + Dreaming V0
2026 — Dreaming V3
```

## 2026 — Dreaming V3 / Improved Memory

**Class: A**

On June 4, 2026, OpenAI announced a substantially more capable and compute-efficient memory architecture built on Dreaming.

Its explicit goals are:

1. Carry useful context forward.
2. Follow preferences and constraints.
3. Stay current over time.

OpenAI says the architecture targets staleness, contradictions, correctness, hundreds of millions of users, multi-year histories, and scalability.

---

# 5. Dreaming Is Background Synthesis

**Class: A**

OpenAI explicitly describes Dreaming as a **background process** that:

- references chat history;
- learns across many conversations;
- automatically curates useful information;
- can learn things that arise naturally without explicit “remember this” commands;
- **synthesizes ChatGPT's memory state**.

Therefore current memory is not well modeled as:

```text
fact → append row → memory
```

A better model is:

```text
many historical observations
          ↓
background interpretation
          ↓
consolidated current understanding
```

Primary source: https://openai.com/index/chatgpt-memory-dreaming/

---

# 6. Memory Is Derived State

**Class: A/F**

Current OpenAI documentation describes memory as a **continually updated synthesis of context from past chats**.

Therefore:

```text
RAW HISTORY ≠ SYNTHESIZED MEMORY STATE
```

More accurately:

```text
historical evidence
      ↓
synthesis
      ↓
derived current state
      ↓
new evidence / time / corrections
      ↓
reconciliation
      ↓
revised state
```

This is a foundational architectural distinction.

Primary source: https://help.openai.com/en/articles/8590148

---

# 7. Memory Summary Is Not the Complete Memory

**Class: A**

OpenAI explicitly says the Memory Summary does **not necessarily include everything ChatGPT remembers**. The underlying synthesis may be broader than the visible summary.

Therefore:

```text
Memory Summary ≠ complete memory
Memory Summary ≠ raw conversation history
Memory Summary ≠ complete runtime context
Memory Summary ≠ complete retrieval index
```

Best interpretation:

**Memory Summary = human-facing projection of a broader synthesized state.**

Primary source: https://help.openai.com/en/articles/8590148

---

# 8. ChatGPT Separately Retrieves Specific Past Conversations

**Class: A**

This capability is independently documented from Dreaming.

In January 2026 OpenAI announced improved recovery of **specific details from past chats**, with the original conversation surfaced as a source when relevant.

In May 2026 OpenAI said ChatGPT became faster at **searching past conversations** and could pull relevant context from past chats, saved memories, files, and connected Gmail where available.

Therefore:

```text
DREAMING SYNTHESIS
        ≠
PAST-CHAT SEARCH
```

Both exist.

Primary source: https://help.openai.com/en/articles/6825453-chatgpt-release-notes

---

# 9. Semantic Memory + Episodic Retrieval

**Class: F — High confidence terminology**

OpenAI does not officially use these cognitive-science labels for Dreaming V3, but they fit the documented capabilities.

## Semantic / synthesized memory

Generalized current understanding such as:

- facts;
- preferences;
- constraints;
- ongoing projects;
- relationships;
- recurring patterns;
- current temporal state.

Likely dominated by Dreaming.

## Episodic memory

Specific prior experiences such as:

- a particular conversation;
- a particular project decision;
- what happened previously;
- why a prior approach failed;
- what was discussed on a specific occasion.

Likely supported by historical conversation retrieval.

Recent OpenAI patents independently describe both synthesized concepts and searchable past interactions, which makes this distinction substantially better grounded than a pure analogy.

---

# 10. Contemporary OpenAI Patent: Personalization State

**Class: B**

OpenAI's 2026 patent **Application programming interface with generative response engine state management** describes a **personalization state** that can contain:

- information directly supplied by the user;
- information inferred from user prompts;
- summaries of past conversation threads;
- a searchable index of past conversation threads;
- a persisted memory file;
- synthesized concepts extracted from previous interactions;
- search of previous interactions for information relevant to the current thread.

This is unusually close to current observed ChatGPT behavior.

Conceptually:

```text
PERSONALIZATION STATE
├── explicit information
├── inferred information
├── persisted memory
├── conversation summaries
├── searchable historical index
└── synthesized concepts
```

Source: https://patents.justia.com/patent/12591766

---

# 11. Memory Is a Distinct Context Type in OpenAI System Design

**Class: B**

The same state-management patent describes conversation metadata that can distinguish input components such as:

- user-provided text;
- model-provided text;
- system prompt;
- personalization state/memory;
- tool or action data.

This is strong evidence that, in contemporary OpenAI architecture, memory is treated as a **distinct typed context component** rather than indistinguishable transcript text.

Source: https://patents.justia.com/patent/12591766

---

# 12. Personalization State Can Enter the Model Context Window

**Class: B**

The state-management patent describes model context windows containing personalization state alongside system and conversation state.

OpenAI's 2026 Project-management patent similarly describes sending user-account personalization state into a generative response engine's context window.

What remains unknown is **how much** of current ChatGPT's memory/personalization state is loaded on a given turn.

Sources:

- https://patents.justia.com/patent/12591766
- https://patents.justia.com/patent/12699964

---

# 13. Contemporary OpenAI Patent Explicitly Defines the Hybrid

**Class: B — Very strong**

OpenAI's August 2026 patent **Project management for generative response engine contexts** describes memory as including:

- persisted data from prior interactions;
- **synthesized concepts extracted from past conversation threads**;
- the ability to **search past interactions** for information relevant to the current conversation.

This is the strongest architecture-level corroboration of the core model:

```text
SYNTHESIZED MEMORY
       +
SEARCHABLE HISTORY
```

Source: https://patents.justia.com/patent/12699964

---

# 14. Runtime Personalization Is Optional

**Class: A/B**

OpenAI's April 2026 Fast Answers behavior establishes that memory is not necessarily invoked on every request.

When a query does not require personalization and ChatGPT has enough confidence to answer directly, the fast path can avoid referencing past chats or memory.

Therefore:

```text
persistent memory exists
       ≠
memory is activated every turn
```

Some runtime decision boundary exists around whether personalization is needed.

Its exact implementation is unknown.

Primary source: https://help.openai.com/en/articles/6825453-chatgpt-release-notes

---

# 15. Relevance Gating Has Deep OpenAI Precedent

**Class: B/C**

OpenAI's custom-instructions patent describes persistent user information being evaluated for relevance to the current request.

Described possible mechanisms include:

- model-generated relevance scores;
- thresholds;
- structured-data matching;
- embedding similarity;
- keyword/entity overlap;
- rules or heuristics;
- historical relevance patterns.

This does not prove Dreaming uses those exact techniques.

It establishes a longstanding OpenAI design principle:

> Persistent personalization should be selectively applied when relevant, not blindly injected into every response.

Source: https://patents.google.com/patent/US12430518B2/en

---

# 16. Persistence and Activation Are Different

**Class: A/B/F**

Two questions must be separated.

## Persistence

Does ChatGPT possess or have access to the information somewhere?

## Activation

Was that information selected to influence this response?

Fast Answers demonstrate that memory can persist without being activated.

Sources and historical retrieval demonstrate selective activation.

This distinction is essential when testing memory.

---

# 17. Sources / Provenance

**Class: A**

Current ChatGPT can expose personalization sources such as:

- custom instructions;
- past chats;
- files;
- memories.

The Sources UI can help explain why information was used and lets users correct relevant state.

But OpenAI explicitly warns that visible Sources may not show **every source or factor** that shaped a response.

Therefore:

```text
visible sources
      ⊆
actual considered/used context
```

The internal provenance representation is undisclosed.

Primary source: https://help.openai.com/en/articles/8590148

---

# 18. User Feedback Is Part of the Memory Loop

**Class: A**

Users can correct, modify, or remove remembered information and underlying sources.

This creates a feedback/reconciliation loop:

```text
history
  ↓
memory
  ↓
response
  ↓
user correction
  ↓
future state
```

How relevance/correction feedback changes ranking, metadata, or synthesis internally is unknown.

Primary source: https://help.openai.com/en/articles/8590148

---

# 19. Time Is a First-Class Memory Problem

**Class: A**

Dreaming explicitly updates memory as time passes.

OpenAI's example transforms an upcoming trip into a historical trip once its date has passed.

This is more than simple expiration. It is **temporal reinterpretation**.

A serious implementation therefore needs some representation of concepts such as:

- event time;
- current validity;
- planned/current/historical state;
- supersession;
- temporal relevance.

The production representation remains unknown.

Primary source: https://openai.com/index/chatgpt-memory-dreaming/

---

# 20. Memory Stores More Than Facts

**Class: A**

OpenAI explicitly evaluates memory across multiple information types.

## Context

Useful facts from prior conversations and long-running work.

## Preferences and constraints

Including:

- direct behavioral instructions;
- explicit user preferences;
- lifestyle constraints;
- implicit preferences inferred from context.

Therefore the synthesized user model is broader than a key-value profile.

Primary source: https://openai.com/index/chatgpt-memory-dreaming/

---

# 21. Memory Uses Importance / Salience

**Class: A**

The current Memory FAQ says ChatGPT tracks details it determines are **most important**.

Thus some salience-selection mechanism exists.

Possible factors may include recency, repetition, explicit importance, task relevance, usefulness, confidence, source provenance, or temporal validity, but the production formula is undisclosed.

Primary source: https://help.openai.com/en/articles/8590148

---

# 22. Models Are Specifically Trained to Use Memory

**Class: A**

OpenAI has a dedicated Personalization-Memory research team.

Published role descriptions say the team works on:

- reinforcement learning;
- dataset creation;
- evaluations;
- post-training;
- frontier-model memory and personalization behavior.

Therefore ChatGPT memory quality is not merely a database/retrieval problem. The model itself is trained to decide how memory should affect a response.

Source: https://openai.com/careers/research-engineer-research-scientist-personal-agi-personalization-san-francisco/

---

# 23. Legacy Saved Memories Remain Separate

**Class: A**

Current ChatGPT still exposes legacy Saved Memories.

OpenAI says saved memories are stored separately from chat history and can be used in future responses until removed.

Deleting the originating chat therefore does not necessarily delete the corresponding saved-memory entry.

This architecture should not be conflated with Dreaming V3.

Primary source: https://help.openai.com/en/articles/8590148

---

# 24. Historical Patent: Personalization Notepad

**Class: C**

OpenAI's 2024-filed patent **Selective learning of information for the generation of personalized responses by a generative response engine** describes a compact, human-readable **personalization notepad**.

The model can:

1. identify candidate user information;
2. decide whether it deserves retention;
3. write appropriate information into the notepad;
4. use it in later responses.

This maps closely to the original Saved Memories generation.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 25. Learned Memory-Write Selection

**Class: C**

The historical patent explicitly discusses training the model to decide whether information deserves memory through mechanisms including:

- reinforcement learning;
- a probability/score that information should be saved;
- reward functions;
- human labelers;
- heuristics.

This establishes early OpenAI interest in **learned salience**, not indiscriminate fact extraction.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 26. Cross-Conversation Inference

**Class: C**

The patent says relevant information may emerge from **a series of prompts over time and across threads**, even when no single prompt explicitly states the conclusion.

Conceptually:

```text
observation A
     +
observation B
     +
observation C
     ↓
derived state D
```

This is strong historical precedent for Dreaming's current cross-conversation synthesis.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 27. Historical Memory Capacity Was Explicitly Bounded

**Class: C**

The patent gives illustrative limits ranging from tens/hundreds of notes to token budgets from hundreds through 100,000 tokens in some embodiments.

It explicitly connects bounded memory to response latency and responsiveness.

These are **patent examples, not current ChatGPT limits**.

Their architectural significance is:

```text
finite fast memory
      +
consolidation
      +
deep retrieval
```

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 28. Historical Persistent Storage and Runtime Loading

**Class: C**

The historical patent says the personalization notepad may be:

- persistently stored in a database associated with the user's account;
- loaded into memory associated with an instance of the response engine when a session begins.

This is one of the few direct OpenAI descriptions of a persistence-to-runtime path.

It does not prove Dreaming V3 uses the same implementation.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 29. Historical Asynchronous Consolidation

**Class: C — Important lineage**

The historical patent describes an **offline/asynchronous consolidation process** that can:

- identify related concepts;
- merge them;
- rewrite them into a compact note;
- remove redundancy;
- free memory capacity.

This looks strongly ancestral to Dreaming:

```text
2024 patent: asynchronous consolidation
           ↓
2025: Dreaming V0
           ↓
2026: Dreaming V3
```

The lineage is an inference. The individual systems are documented.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 30. Historical Retention / Reinforcement

**Class: C**

Under memory pressure, the historical patent describes prioritizing:

- newer memories;
- recently accessed memories.

An old but useful memory can effectively be refreshed/reordered so it survives longer.

Whether current Dreaming V3 uses the same mechanism is unknown.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 31. Historical “Deep Memory”

**Class: C — Major finding**

The historical patent distinguishes a compact personalization notepad from a separate **deep memory**.

### Personalization notepad

- compact;
- efficient;
- human-readable;
- information about the user.

### Deep memory

- larger;
- need not be human-readable;
- remembers past sessions/projects.

The patent describes a process resembling:

```text
past sessions
     ↓
extract/store topics
     ↓
convert topics to embeddings
     ↓
associate embeddings with user account
     ↓
new request
     ↓
retrieve relevant embeddings
     ↓
use prior-session information
```

This is the strongest historical implementation-level ancestor of current episodic conversation retrieval.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 32. Historical Multiple Memory Identities

**Class: C**

The historical patent contemplates multiple personalization identities under one account, such as:

```text
personal identity → memory state A
work identity     → memory state B
```

This does not prove Projects use the same implementation, but it establishes early OpenAI interest in namespaced memory.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 33. Historical Privacy / Write Gate

**Class: C**

The patent describes a “track topics” control where disabling topic tracking can prevent both:

- storage of session topics;
- determination of whether information from the thread should enter the personalization notepad.

This is historical precedent for separately governing memory read/write/learning behavior.

Source: https://patents.google.com/patent/US20250200361A1/en

---

# 34. Temporary Chat Proves Read and Write Are Separate

**Class: A — Very important**

Current Temporary Chat behavior distinguishes memory read from memory write.

A non-personalized Temporary Chat does not use or create memory.

A personalized Temporary Chat can use existing personalization but does **not** create or update memory while it remains temporary.

Therefore:

```text
MEMORY READ
     ≠
MEMORY WRITE
```

Primary sources:

- https://help.openai.com/en/articles/8914046-temporary-chat-faq
- https://help.openai.com/en/articles/6825453-chatgpt-release-notes

---

# 35. Saving a Temporary Chat Changes Its Memory Eligibility

**Class: A**

When a Temporary Chat is saved, it becomes a normal chat and from then on follows normal account personalization/memory behavior.

This implies memory eligibility can depend on **source lifecycle state**, not only message contents.

Source: https://help.openai.com/en/articles/8914046-temporary-chat-faq

---

# 36. Project Memory Is a Real Namespace

**Class: A**

With project-only memory:

- memories outside the project are not referenced;
- chats can reference other chats inside the same project;
- project chats cannot reference conversations outside the project;
- outside chats cannot reference project conversations.

This establishes real memory/access scoping.

Primary source: https://help.openai.com/en/articles/10169521-projects-in-chatgpt

---

# 37. Project Memory Is Not a List of Memory Items

**Class: A**

OpenAI says project memory is not exposed as a conventional list of individual memory records in the same way personal memory is.

ChatGPT can instead use context directly from other conversations inside the project.

Further evidence:

```text
memory ≠ only explicit memory records
```

Primary source: https://help.openai.com/en/articles/10169521-projects-in-chatgpt

---

# 38. Project Context Is Prioritized

**Class: A**

For eligible users, OpenAI says project chats can reference prior chats within a project and prioritize project chats/files for project-related questions.

This gives us at least one documented retrieval prior:

**project relevance.**

The exact ranking formula remains unknown.

Primary source: https://help.openai.com/en/articles/10169521-projects-in-chatgpt

---

# 39. Health Memory — Historical vs Current

This corrects the V1 source of truth.

## Early 2026 behavior

**Class: Historical A**

The initial Health product emphasized strong compartmentalization between Health and ordinary ChatGPT.

Source: https://openai.com/index/introducing-chatgpt-health/

## Later/current 2026 behavior

**Class: Current A**

The Health product evolved. Current documentation allows users to choose whether relevant connected Health information can be used across conversations.

Current documentation also distinguishes **connected/synced Health data** from conversational memories: synced Health data does not itself directly create memories, while conversations can create memories when account memory is enabled.

Sources:

- https://openai.com/index/health-in-chatgpt/
- https://help.openai.com/en/articles/20001036-what-is-chatgpt-health

Therefore the old blanket statement that Health can never influence normal ChatGPT is stale.

---

# 40. Memory Governance Is a First-Class Layer

**Class: A/F**

Current ChatGPT proves multiple independent memory scopes/policies, including:

- general account memory;
- project-only memory;
- shared/workspace boundaries;
- temporary personalized chat;
- temporary non-personalized chat;
- connected-source permissions;
- Health-specific permissions.

A correct architecture therefore needs explicit governance questions:

```text
Can this source be read?
Can this source create memory?
Can this state leave its scope?
Which other scopes may reference it?
Does workspace policy override user preference?
```

---

# 41. Different Evidence Sources Have Different Lifecycles

**Class: A**

Chats, files, memories, projects, and Temporary Chats have different persistence/deletion rules.

Examples:

- saved memories are stored separately from chat history;
- deleting a chat does not automatically remove an independently stored saved-memory entry;
- Library/project files have separate retention semantics;
- Temporary Chats have their own retention lifecycle.

This reinforces that durable evidence is **not one store with one lifecycle**.

Primary source: https://help.openai.com/en/articles/8983778-chat-and-file-retention-policies-in-chatgpt

---

# 42. Fully Removing Knowledge Requires Removing Its Sources

**Class: A**

Current OpenAI guidance says that fully removing something ChatGPT may know can require deleting it from all remaining sources where it appears, including chats, archived chats, files, memory state, and relevant connected sources.

This is strong evidence that synthesized memory is **derived from underlying evidence** and can potentially be reconstructed when source material remains.

Primary source: https://help.openai.com/en/articles/8590148

---

# 43. Current OpenAI Codex Memory — Adjacent Evidence

**Class: D**

OpenAI's public Codex repository includes a current memory system.

This is **not proof of ChatGPT's implementation**, but its architecture is highly informative.

Codex uses a multi-stage process resembling:

```text
raw rollouts
     ↓
Phase 1 extraction
     ↓
raw/candidate memories
     ↓
Phase 2 global consolidation
     ↓
MEMORY.md
memory_summary.md
rollout summaries
skills
```

Source: https://github.com/openai/codex/blob/main/codex-rs/memories/write/templates/memories/consolidation.md

---

# 44. Codex Treats Raw History as Evidence

**Class: D**

Codex's consolidation instructions treat raw rollouts as immutable evidence and higher-order memory files as derived artifacts.

That directly embodies:

```text
history = evidence
memory = derived interpretation
```

Source: https://github.com/openai/codex/blob/main/codex-rs/memories/write/templates/memories/consolidation.md

---

# 45. Codex Uses Hierarchical Memory

**Class: D**

Codex separates:

### `memory_summary.md`

Dense, high-signal routing context intended to remain readily available.

### `MEMORY.md`

A deeper searchable registry.

### rollout summaries/raw evidence

Opened only when more exact historical evidence is required.

Conceptually:

```text
small always-available routing memory
             +
deeper searchable consolidated state
             +
underlying source evidence
```

Source: https://github.com/openai/codex/blob/main/codex-rs/ext/memories/templates/memories/read_path.md

---

# 46. Codex Has a Memory Relevance Boundary

**Class: D**

Codex's read path explicitly decides whether memory should be used for a request. Self-contained tasks can skip memory, while historical/project-dependent tasks trigger retrieval.

This closely parallels ChatGPT's documented Fast Answer memory bypass.

Source: https://github.com/openai/codex/blob/main/codex-rs/ext/memories/templates/memories/read_path.md

---

# 47. Empirical Evidence from Real ChatGPT Users

**Class: E**

A 2026 study, **The Algorithmic Self-Portrait: Deconstructing Memory in ChatGPT**, analyzed 2,050 observed memory entries from 80 users.

The study reports that roughly:

- **96%** of observed memory entries were created without an explicit user memory command;
- **84%** were directly grounded in conversational context.

This study primarily reflects the pre-Dreaming-V3 saved-memory era and should not be treated as direct reverse engineering of V3.

Source: https://arxiv.org/abs/2602.01450

---

# 48. Client / Reverse-Engineering Evidence

**Class: E — use cautiously**

Independent client and frontend analysis has observed a legacy-looking ChatGPT memory surface under:

```text
/backend-api/memories
```

with memory/accounting fields in some generations.

Separate 2026 frontend analysis has reported a newer About You / Dreaming-summary path under the `memories/about_you/summary` family.

This suggests—but does not prove—that legacy Saved Memories and newer Dreaming-summary functionality have distinct product/API surfaces.

Observational references:

- https://github.com/B4PT0R/codex-backend-sdk/blob/main/docs/backend-api.md
- https://husain-zaidi.com/chatgpt-dreaming/

These observations must never override primary OpenAI documentation.

---

# 49. Memory-Capacity Client Observations

**Class: E — highly qualified**

Historical reverse-engineering reports have observed fields resembling:

```text
memory_max_tokens
memory_num_tokens
```

with dramatically different values across product generations/accounts.

These observations are useful only as evidence that **some memory surface had token-denominated accounting**.

They do **not** prove that the same number of tokens enters the model context window, nor that the field represents Dreaming V3's synthesized state.

The semantics may have changed across generations.

Do not convert these client observations into current production capacity claims.

---

# 50. Current Best Three-Level Memory Hierarchy

**Class: F — High confidence**

The evidence strongly supports thinking in three conceptual levels.

## Level 0 — Raw evidence

```text
chats
files
connected sources
explicit settings
historical interactions
```

## Level 1 — Episodic representation

```text
indexed / summarized / searchable prior experiences
```

## Level 2 — Synthesized semantic state

```text
facts
preferences
constraints
projects
relationships
temporal state
learned patterns
```

Runtime:

```text
current request
      ↓
scope + relevance decision
      ↓
Level 2 relevant state
      +
Level 1 relevant episodes
      ↓
context package
      ↓
model
```

This model converges with:

- Dreaming documentation;
- current past-chat search;
- 2026 OpenAI patents;
- the historical deep-memory patent;
- current Codex architecture.

---

# 51. Better Mental Model: “Memory Compiler”

**Class: F**

Calling ChatGPT memory “RAG” undersells Dreaming.

A more useful analogy is a compiler:

```text
SOURCE MATERIAL
historical evidence
       ↓
BACKGROUND COMPILER
Dreaming:
reconcile
consolidate
generalize
update
compress
       ↓
COMPILED USER STATE
       ↓
RUNTIME RETRIEVER / LINKER
find relevant historical episodes
       ↓
CONTEXT PACKAGE
       ↓
MODEL EXECUTION
```

This explains how the system can:

- infer something not explicitly stated;
- update state as time passes;
- recover a specific old conversation;
- ignore irrelevant personal information;
- preserve continuity without loading all history.

“Memory compiler” is our terminology, not OpenAI's.

---

# 52. Persistence vs Retrieval vs Synthesis vs Activation

These are distinct operations.

## Persistence

Information still exists somewhere.

## Retrieval

The system locates potentially relevant historical evidence.

## Synthesis

The system transforms multiple pieces of evidence into generalized/current state.

## Activation

A memory or retrieved episode is actually selected to influence this response.

Apparent memory failures can occur at different layers:

```text
not persisted
vs
not indexed
vs
not retrieved
vs
not synthesized correctly
vs
blocked by scope
vs
not activated
vs
model ignored or misused it
```

---

# 53. Memory Failure Modes

A complete architecture must account for at least these categories.

### Capture failure

Useful information never becomes eligible for memory.

### Synthesis failure

Evidence exists but is generalized incorrectly.

### Staleness

Old state survives after circumstances change.

### Contradiction

Incompatible claims coexist.

### Retrieval failure

The correct historical episode exists but is not found.

### Ranking failure

It is found but ranked below less useful information.

### Scope failure

Correct information is unavailable because of project/workspace/privacy boundaries.

### Over-personalization

Memory influences a response where it should not.

### Under-personalization

Relevant memory is ignored.

### Provenance failure

Memory affects an answer without a usable source/explanation path.

### Deletion/reconstruction failure

Derived memory disappears but survives in source evidence and later reappears.

### Temporal failure

Future/current/historical state is interpreted incorrectly.

### Model-use failure

Correct memory reaches the model but is applied badly.

---

# 54. What We Can State With Very High Confidence

1. ChatGPT has a background memory-synthesis system called Dreaming.
2. The current announced generation is Dreaming V3.
3. Dreaming learns across many conversations.
4. Dreaming synthesizes a memory state.
5. Memory Summary is only a high-level representation of broader memory.
6. ChatGPT separately searches previous conversations.
7. Specific past chats can be surfaced as sources.
8. Relevant historical information can come from chats, saved memories, files, and connected sources.
9. Memory can update itself as time passes.
10. Memory represents preferences and constraints, not only facts.
11. ChatGPT tracks information it determines is important.
12. Personalization can be bypassed for some requests.
13. Memory read and memory write can be controlled independently.
14. Project memory creates real conversation-access boundaries.
15. Project memory is not simply a visible list of memory records.
16. Visible Sources are partial rather than complete internal traces.
17. OpenAI explicitly post-trains frontier models for memory/personalization behavior.
18. Contemporary OpenAI system designs repeatedly describe personalization state as synthesized concepts plus searchable historical interactions.
19. Contemporary OpenAI designs treat personalization state as a distinct context component.
20. OpenAI designs allow personalization state to enter model context windows.
21. Historical OpenAI design contained compact user-readable memory plus deeper episodic memory.
22. Historical deep memory explicitly used topics → embeddings → prior-session retrieval.
23. OpenAI has long explored learned/RL-based memory-write decisions.
24. OpenAI has long explored asynchronous memory consolidation.
25. Current Codex independently uses hierarchical consolidation plus selective deeper retrieval.

---

# 55. What We Must NOT Claim

Do not say:

### “ChatGPT just uses a vector database.”

Unsupported.

### “Memory Summary is the complete memory.”

False according to OpenAI.

### “Everything remembered is injected into every prompt.”

Contradicted by relevance/scoping behavior and memory-bypass paths.

### “Dreaming V3 definitely uses the 2024 patent's topic-embedding architecture.”

Unproven.

### “Patent architecture equals production architecture.”

Incorrect evidence handling.

### “Health is permanently isolated from normal ChatGPT.”

Stale after later 2026 Health changes.

### “Temporary Chat cannot use memory.”

Outdated. Personalized Temporary Chat can read existing personalization while remaining non-writing.

### “Deleting a Memory Summary deletes everything ChatGPT knows.”

False when underlying sources remain.

### “Visible Sources reveal every memory/chat considered.”

False according to OpenAI documentation.

### “Memory capacity equals context-window tokens.”

Unsupported.

---

# 56. Remaining Hard Unknowns

These remain **U — UNKNOWN**.

## Dreaming implementation

- exact Dreaming V3 model;
- Dreaming system prompt;
- Dreaming V1 architecture;
- Dreaming V2 architecture;
- exact V0→V3 implementation changes;
- exact trigger schedule;
- whether processing is incremental, periodic, event-driven, or hybrid;
- whether Dreaming runs only after conversations or through multiple background triggers.

## Storage

- production database technology;
- Dreaming record schema;
- whether synthesized state is prose, structured records, graphs, vectors, model-state representations, or a hybrid;
- number of records per user;
- exact source-provenance representation.

## Historical retrieval

- current embedding model;
- whether every chat is embedded;
- whether indexing occurs at message, chunk, summary, topic, conversation, or multiple levels;
- vector/index technology;
- query-rewrite mechanism;
- top-K;
- reranker;
- ranking formula;
- retrieval token budget;
- whether retrieval iterates during generation.

## Synthesis

- salience formula;
- contradiction-resolution algorithm;
- confidence representation;
- source-quality weighting;
- temporal-state schema;
- supersession rules;
- memory decay;
- whether retrieval/use reinforces long-term retention.

## Runtime context

- exact personalization token budget;
- exact memory ordering relative to system/developer/current-chat context;
- whether all synthesized state is loaded or selected first;
- how synthesized state and retrieved episodes are merged;
- whether candidate context packages are scored;
- whether current ChatGPT produces personalized/non-personalized candidate responses internally.

## Capacity

- exact Free capacity;
- exact Go capacity;
- exact Plus capacity;
- exact Pro capacity;
- what current “memory capacity” measures;
- relationship between persistent capacity and runtime context size.

## Feedback

- what relevance feedback changes internally;
- whether feedback modifies ranking, synthesis, metadata, or training signals;
- whether corrections trigger immediate or asynchronous resynthesis.

## Infrastructure

- queues;
- consolidation locks;
- distributed indexing architecture;
- refresh pipeline;
- consistency model;
- cache architecture;
- tenant/shard representation.

## Code

- Dreaming V3 production source;
- production prompts;
- public Dreaming API.

---

# 57. Experimental Questions That Could Reduce the Unknowns

Document research is approaching diminishing returns. Controlled experiments are the next major evidence source.

## Dreaming latency

Plant a novel fact and measure how long it takes to appear in synthesized memory across new chats.

## Cross-chat synthesis

Spread one inference across multiple conversations so no individual conversation states the conclusion explicitly.

Test whether Dreaming synthesizes the conclusion.

## Contradiction handling

Establish fact A, later establish not-A, then measure:

- immediate answer behavior;
- Memory Summary update;
- old-chat retrieval behavior;
- eventual reconciliation.

## Temporal transformation

Create dated future state and test:

```text
planned → current → historical
```

## Repetition / salience

Introduce equally important facts with different repetition frequencies and compare retention/retrieval probability.

## Access reinforcement

Retrieve an old item repeatedly and test whether repeated use changes later retrieval probability.

## Episodic vs semantic retrieval

Compare generalized preference questions with requests for a specific historical decision.

## Project boundaries

Repeat identical facts inside and outside project-only memory and measure information flow.

## Temporary Chat

Test:

```text
non-personalized temporary
personalized temporary
saved temporary
regular chat
```

## Source deletion

Delete:

1. summary only;
2. chat only;
3. file only;
4. every source.

Observe whether knowledge reconstructs.

## Model differences

Compare identical memory/history state across Instant, Thinking, and other model configurations.

## Scale

Build controlled histories with hundreds or thousands of distinct facts/episodes and measure retrieval degradation, salience, and temporal bias.

---

# 58. Architectural Lessons for Reproducing ChatGPT-Like Memory

If the goal is to reproduce the **principles** rather than copy undocumented internals, a serious architecture needs:

```text
DURABLE EVIDENCE
       ↓
EPISODIC INDEX
       +
BACKGROUND CONSOLIDATION
       ↓
SYNTHESIZED CURRENT STATE
       ↓
TEMPORAL / CONFLICT RECONCILIATION
       ↓
SCOPE + PERMISSION FILTER
       ↓
RELEVANCE GATE
       ↓
SELECTIVE EPISODIC RETRIEVAL
       ↓
CONTEXT ASSEMBLY
       ↓
MODEL TRAINED TO USE MEMORY
       ↓
RESPONSE
       ↓
PROVENANCE + FEEDBACK
```

Required properties:

- preserve auditable evidence;
- treat memory as derived state;
- separate retrieval from synthesis;
- do not use memory on every request;
- revise state over time;
- reconcile contradictions;
- keep source provenance;
- enforce scope-specific read/write boundaries;
- feed user corrections back into state;
- consolidate/forget low-value material;
- train the answering model to use memory appropriately.

---

# 59. Core Design Principle

The central insight from all available evidence is:

> **History is evidence. Memory is synthesized state. Retrieval recovers episodes. Governance controls access. Relevance controls activation. Context management assembles what the model sees. The model decides how to use it.**

This is the current highest-level architectural source of truth.

---

# 60. Canonical Architecture

```text
                           SOURCE EVIDENCE
                  ┌─────────────┼─────────────┐
                  │             │             │
                chats         files         apps
                  │             │             │
                  └────── instructions ───────┘
                                │
              ┌─────────────────┴─────────────────┐
              │                                   │
              ▼                                   ▼
        EPISODIC PIPELINE                  SEMANTIC PIPELINE
      index / summarize / search          Dreaming synthesis
              │                                   │
              │                            deduplicate
              │                            reconcile
              │                            temporal update
              │                                   │
              ▼                                   ▼
     historical representations          current user state
              │                                   │
              └───────────────┬───────────────────┘
                              │
                     SCOPE / PERMISSION
                              │
                     RELEVANCE / ROUTING
                              │
                 ┌────────────┴────────────┐
                 │                         │
          relevant semantic          relevant episodic
               state                    evidence
                 │                         │
                 └────────────┬────────────┘
                              │
                       CONTEXT PACKAGE
                              │
                    current conversation
                    system instructions
                    tools/action context
                              │
                              ▼
                     POST-TRAINED MODEL
                              │
                              ▼
                           RESPONSE
                              │
                  sources / user correction
                              │
                              ▼
                       FUTURE EVIDENCE
```

---

# 61. Confidence Summary

| Architectural claim | Confidence |
|---|---|
| Background synthesis exists | Confirmed current |
| Dreaming V3 exists | Confirmed current |
| Cross-conversation synthesis exists | Confirmed current |
| Past-chat search exists | Confirmed current |
| Specific episodic retrieval exists | Confirmed current |
| Memory Summary is partial | Confirmed current |
| Temporal revision exists | Confirmed current |
| Preference/constraint memory exists | Confirmed current |
| Memory-bypass/relevance path exists | Confirmed current |
| Read/write controls can differ | Confirmed current |
| Project memory is scoped | Confirmed current |
| Provenance/sources exist | Confirmed current |
| Visible Sources are incomplete | Confirmed current |
| Model post-training for memory exists | Confirmed current |
| Synthesized + searchable-history architecture exists in OpenAI designs | Confirmed OpenAI design |
| Memory is treated as distinct context in OpenAI designs | Confirmed OpenAI design |
| Deep embedding memory existed historically | Confirmed historical |
| Async consolidation existed historically | Confirmed historical |
| RL-based memory-write selection existed historically | Confirmed historical |
| Current Codex uses hierarchical memory | Confirmed adjacent implementation |
| Current Dreaming uses exact historical embedding mechanism | Unknown |
| Current Dreaming DB/schema | Unknown |
| Exact retrieval/ranking algorithm | Unknown |
| Exact runtime memory token budget | Unknown |
| Exact Dreaming prompt/model | Unknown |

---

# 62. Primary Evidence Registry

## Current ChatGPT

### OpenAI — Dreaming: Better memory for a more helpful ChatGPT

Primary source for Dreaming V3, background synthesis, its history, preferences/constraints, continuity, and temporal updating.

https://openai.com/index/chatgpt-memory-dreaming/

### OpenAI — Memory FAQ

Primary source for Memory Summary behavior, Sources, legacy Saved Memories, correction/deletion behavior, and underlying source relationships.

https://help.openai.com/en/articles/8590148

### OpenAI — ChatGPT Release Notes

Primary source for 2026 past-chat retrieval, Fast Answers, cross-source retrieval, and Temporary Chat personalization changes.

https://help.openai.com/en/articles/6825453-chatgpt-release-notes

### OpenAI — Projects in ChatGPT

Primary source for project memory, project-only boundaries, and project-context behavior.

https://help.openai.com/en/articles/10169521-projects-in-chatgpt

### OpenAI — Temporary Chat FAQ

Primary current source for Temporary Chat memory-read/no-memory-write behavior.

https://help.openai.com/en/articles/8914046-temporary-chat-faq

### OpenAI — ChatGPT Health

Historical and current sources for Health-related context/memory behavior.

https://openai.com/index/introducing-chatgpt-health/

https://openai.com/index/health-in-chatgpt/

https://help.openai.com/en/articles/20001036-what-is-chatgpt-health

### OpenAI — Chat and File Retention Policies

Primary source for source lifecycles and deletion semantics.

https://help.openai.com/en/articles/8983778-chat-and-file-retention-policies-in-chatgpt

## OpenAI Research / Training

### OpenAI — Personalization-Memory research role

Primary evidence that OpenAI works on RL, datasets, evaluation, and post-training for memory/personalization.

https://openai.com/careers/research-engineer-research-scientist-personal-agi-personalization-san-francisco/

## Contemporary OpenAI Architecture

### Patent 12,591,766 — Application programming interface with generative response engine state management

Strong evidence for personalization state, conversation summaries, searchable history, synthesized concepts, typed context, and context-window integration.

https://patents.justia.com/patent/12591766

### Patent 12,699,964 — Project management for generative response engine contexts

Strong evidence for persisted memory, synthesized concepts, historical search, context management, and scoped project personalization.

https://patents.justia.com/patent/12699964

### Patent 12,430,518 — Custom model instructions with language models

Strong adjacent evidence for persistent personalization, relevance scoring, session caching, and selective application.

https://patents.google.com/patent/US12430518B2/en

## Historical OpenAI Memory Design

### US20250200361A1 — Selective learning of information for personalized responses

Primary historical source for personalization notepad, RL memory selection, persistent storage, asynchronous consolidation, retention, multiple identities, and deep memory using topic embeddings.

https://patents.google.com/patent/US20250200361A1/en

## Adjacent OpenAI Implementation

### OpenAI Codex — Memory consolidation

Current public OpenAI implementation of raw evidence → extraction → consolidation → hierarchical memory.

https://github.com/openai/codex/blob/main/codex-rs/memories/write/templates/memories/consolidation.md

### OpenAI Codex — Memory read path

Current public implementation of relevance-gated memory lookup and progressive disclosure.

https://github.com/openai/codex/blob/main/codex-rs/ext/memories/templates/memories/read_path.md

## Empirical Evidence

### The Algorithmic Self-Portrait: Deconstructing Memory in ChatGPT

Study of 2,050 observed memory entries from 80 ChatGPT users; useful evidence about automatic memory creation before Dreaming V3.

https://arxiv.org/abs/2602.01450

## Client Observation

### Independent Dreaming frontend analysis

Secondary evidence for a separate About You / summary frontend surface.

https://husain-zaidi.com/chatgpt-dreaming/

### Reverse-engineered ChatGPT backend documentation

Secondary evidence for a legacy `/backend-api/memories` product surface.

https://github.com/B4PT0R/codex-backend-sdk/blob/main/docs/backend-api.md

Client observations must never override primary product documentation.

---

# 63. Change Log

## Version 2.0 — September 8, 2026

Added:

- 2026 state-management patent;
- 2026 project/context-management patent;
- typed personalization-state evidence;
- explicit hybrid synthesized-memory + searchable-history architecture;
- Fast Answer memory bypass;
- relevance-gating evidence;
- current Temporary Chat read/write separation;
- current project namespace behavior;
- corrected Health architecture;
- source-lifecycle distinctions;
- current Codex hierarchical-memory architecture;
- empirical 80-user study;
- client-observed legacy/Dreaming surfaces;
- persistence-vs-activation distinction;
- three-level memory hierarchy;
- failure taxonomy;
- experimental reverse-engineering agenda;
- expanded evidence classification.

Corrected:

- Health is no longer represented as universally isolated from ordinary ChatGPT.
- Temporary Chat is no longer represented as universally unable to read memory.
- Dreaming is no longer modeled merely as “summary + RAG.”
- OpenAI's hybrid synthesized-state + historical-search architecture is now supported by multiple contemporary OpenAI system patents, not only inference.

---

# 64. Version 2.0 Final Conclusion — Preserved

As of September 8, 2026:

**The strongest public evidence indicates that ChatGPT memory is a layered long-term personalization system rather than a single memory store.**

It combines:

- durable historical evidence;
- background synthesis through Dreaming;
- continuously updated generalized user state;
- specific historical conversation retrieval;
- preferences and constraints;
- temporal reconciliation;
- salience selection;
- scoped read/write permissions;
- source provenance;
- explicit user correction mechanisms;
- runtime relevance gating;
- context management;
- frontier models post-trained to use memory appropriately.

Contemporary OpenAI patents independently describe personalization state containing **both synthesized concepts and searchable historical interactions**, while historical OpenAI patents show clear lineage from a compact user-readable memory plus deeper episodic memory.

The best architectural abstraction is:

> **History is evidence. Dreaming compiles history into current state. Retrieval recovers relevant episodes. Governance determines what can flow where. Relevance determines what gets activated. Context management assembles what the model sees. The model decides how to use it.**

---

# 65. Evidence Preservation Rule — No Evidence Compression

**Class: Governance rule for this repository**

This source of truth must preserve **both conclusions and the evidence that produced them**.

A later summary must not silently erase a lower-confidence observation merely because the architectural conclusion can be stated more compactly.

For every materially relevant discovery, preserve when available:

```text
date
source title
source URL
evidence class
current vs historical status
specific observation or finding
architectural implication
caveat / confidence boundary
contradictions with other evidence
open questions
```

Rules:

1. **Never delete a prior finding merely because it is superseded.** Mark it historical, stale, contradicted, or superseded.
2. **Do not replace exact observed values with vague summaries.** Preserve the values and separately explain why they may not generalize.
3. **Do not promote patents to production facts.** Preserve them as system-design evidence.
4. **Do not promote client/network observations to official API contracts.** Preserve them as observations.
5. **Do not let architectural summaries replace raw evidence.** Both belong in this file.
6. **When an official source changes, preserve the old rule and record the newer rule that supersedes it.**
7. **When source quality differs, preserve the evidence but rank its authority explicitly.**
8. **Unknown remains a valid result.** Absence of public evidence must never be filled with an implementation guess.

This rule exists specifically to prevent future revisions of this file from losing evidence through over-compression.

---

# 66. Direct Interview with OpenAI Memory / Personalization Leads

**Class: E — direct statements from OpenAI personnel, externally hosted**

On October 6, 2025, the Limitless podcast published an interview introducing **Kristina Kaplan and Sameer Ahmed as leading memory and personalization at OpenAI**.

Important statements/design intent from the interview:

- Sameer said ChatGPT memory predates both of them joining the team.
- He described the 2024 generation with a notebook analogy: ChatGPT could write selected things down and later consult them, but that did not amount to a full understanding of the user.
- Kristina described the April 2025 update as an effort to make cross-conversation memory more natural and assistant-like rather than merely carrying a small notebook of facts.
- Sameer said the team looks at prior art in human/cognitive memory when thinking about the problem.
- He also emphasized that ChatGPT memory still lagged human memory in areas including understanding the gist of people, interaction fidelity, and memory triggers.
- Both framed memory as foundational to a persistent assistant that understands the user across interactions rather than restarting from zero.
- In their discussion of Pulse/personalization, they described memory as foundational to an assistant that can understand what matters to the user and eventually do useful work on the user's behalf.
- They repeatedly framed the target as an assistant/representative aligned with the user, not a literal digital clone of the user.

This interview is valuable for **design intent**, but it is not an OpenAI-hosted technical specification and does not establish backend implementation details.

Source:

https://limitless.fm/episodes/we-interviewed-the-team-behind-openais-1-feature/transcript

---

# 67. Additional 2026 OpenAI Patent Independently Repeats the Personalization Architecture

**Class: B — contemporary corroborating OpenAI system design**

A third 2026 OpenAI patent independently repeats the personalization-state architecture already visible in Patents 12,591,766 and 12,699,964.

**Patent:** 12,706,918  
**Title:** *Seamless consumer integration of access to a generative response engine*  
**Filed:** June 17, 2025  
**Issued:** August 11, 2026  
**Assignee:** OpenAI OpCo, LLC  
**Inventors:** David Cummings, Athyuttam Eleti, Miqdad Jaffer

Although the patent's primary subject is application/API integration rather than memory, its system description says personalization state can include:

- information received directly from the user account;
- information inferred from user prompts;
- summaries of prior conversation threads;
- a searchable index of prior conversation threads;
- a persisted memory file from previous interactions/sessions;
- synthesized concepts extracted from past conversation threads;
- the ability to search past interactions for information relevant to a current conversation;
- model-directed writes of facts/data judged useful for later sessions.

It also describes conversation metadata that labels personalization state separately from user text, model text, system prompts, and tool/action data.

This matters because the same **synthesized state + searchable interaction history + typed personalization context** architecture appears repeatedly across separate contemporary OpenAI system patents.

It still does **not** prove that current Dreaming V3 production uses every described embodiment.

Source:

https://patents.justia.com/patent/12706918

---

# 68. Historical Personalization Patent — Full Metadata and Previously Compressed Details

**Class: C — historical OpenAI design**

Canonical U.S. publication:

**US20250200361A1 — Selective learning of information for the generation of personalized responses by a generative response engine**

Known patent-family metadata:

- Applicant / assignee lineage: OpenAI OpCo, LLC.
- U.S. application: **18/732,157**.
- Priority: U.S. provisional **63/609,558**, filed December 13, 2023.
- U.S. filing date: June 3, 2024.
- Publication date: June 19, 2025.
- International publication: **WO2025128397A1**.
- Inventors include Prasad Chakka, Dave Cummings, Noah Deutsch, William Fedus, Tarun Gogineni, Yuchen He, Joanne Jang, Lien Mamitsuka, Warren Ouyang, Yilei Qian, John Schulman, Javi Soto Bustos, Anton Tananaev, Jonathan Ward, Marvin Zhang, and Benjamin Zweig.

Previously compressed implementation details that must remain preserved:

## Example notepad capacities

The patent gives **illustrative** note-count examples including values such as:

```text
20
50
100
200
500 notes
```

and illustrative token-budget examples including values such as:

```text
100
200
500
1,000
2,000
4,000
10,000
100,000 tokens
```

Some claims/embodiments also refer to a memory/notepad size on the order of **4,000 notes**.

These are patent examples only. They are **not current ChatGPT limits**.

## Alternative memory representations

The patent does not restrict persistent personalization to one prose notepad. It contemplates alternative representations including embeddings, keys, weights, and more complex stored information such as documents, images, or full conversation/thread material.

## User-readable management

The patent contemplates user-facing inspection and natural-language editing/deletion of remembered information, including the ability for a user to ask the system to remove or modify retained information.

## Personalized-vs-non-personalized candidate selection

One described embodiment can generate personalized and non-personalized candidate response portions, score/evaluate them, and choose a preferred result.

**This is only a possible historical embodiment. It is not evidence that current ChatGPT generates two answers on every turn.**

## Cross-thread inference example

The patent explicitly allows multiple prompts spread over time/threads to collectively imply a user state not directly stated in any single message. One example describes several observations collectively supporting an inference that the user is having a bad week.

## Persistence/runtime path

The patent describes persistent account-associated storage and loading the personalization state into a response-engine instance for low-latency use when a session begins.

## Deep memory separation

Deep-memory embeddings are described separately from the compact personalization notepad, reinforcing that user-profile knowledge and specific past-thread/project knowledge were intentionally modeled as different memory functions.

Primary source:

https://patents.google.com/patent/US20250200361A1/en

International family source:

https://patents.google.com/patent/WO2025128397A1/en

---

# 69. Dreaming V3 Quantitative Rollout Facts

**Class: A**

Two quantitative facts from the Dreaming V3 rollout must remain explicit rather than being compressed into “more scalable.”

## Plus / Pro capacity

OpenAI said the 2026 memory rollout provided **twice as much memory capacity** for Plus and Pro users.

OpenAI did **not** define the unit behind “memory capacity.” It must not be equated with runtime prompt/context tokens without further evidence.

## Compute efficiency

OpenAI said improvements to Dreaming reduced the cost/compute needed to serve the memory system to Free users by approximately **5×**.

This supports the conclusion that Dreaming is a separately operated background memory system with material serving/compute cost, but it does not reveal its model, schedule, or infrastructure.

OpenAI also positioned Dreaming as the shared/scalable memory foundation going forward.

Sources:

https://openai.com/index/chatgpt-memory-dreaming/

https://help.openai.com/en/articles/6825453-chatgpt-release-notes

---

# 70. Exact Historical Client Memory-Capacity Observations

**Class: E — raw observations, very low authority relative to official documentation**

The raw values matter as historical evidence even though their semantics are unresolved.

## September 3, 2024 observation

A user inspecting ChatGPT's memory endpoint reported a response containing approximately:

```text
memory_max_tokens: 8000
memory_num_tokens: 525
```

Source:

https://www.reddit.com/r/ChatGPT/comments/1f879xy

This is a community observation, not an OpenAI contract.

## November–December 2025 observations

A reverse-engineering/utility-script discussion reported responses from:

```text
GET https://chatgpt.com/backend-api/memories?include_memory_entries=false
```

with examples such as:

```text
memory_max_tokens: 5000000
memory_num_tokens: 8927
```

and another observed response with approximately:

```text
memory_max_tokens: 5000000
memory_num_tokens: 8225
```

Participants associated the 5,000,000 value with some Plus accounts; the same discussion also contained reports of substantially smaller values on other account types. Those plan-specific claims were not independently verified.

Source:

https://linux.do/t/topic/1175093

## 2026 third-party SDK documentation

A reverse-engineered ChatGPT backend SDK documents an example response with:

```text
memory_max_tokens: 12000
memory_num_tokens: 123
```

and notes that individual memory items may expose fields such as `conversation_id`, `created_timestamp`, `gizmo_id`, `last_updated`, and `labels`.

Source:

https://github.com/B4PT0R/codex-backend-sdk/blob/main/docs/backend-api.md

## Migration-tool observation

A separate migration toolkit reports using `include_memory_entries=true` and shows an example export header around:

```text
9,323 / 5,000,000 tokens
```

with 203 exported memory entries, split by the tool into 199 active and 4 older/less-relevant entries.

The active/older terminology may be the migration tool's interpretation rather than a native ChatGPT schema and must be treated accordingly.

Source:

https://github.com/Siamsnus/GPT2Claude-Migration-Kit/blob/main/CHANGELOG.md

## Interpretation boundary

The variation from approximately 8,000 → 5,000,000 → 12,000 across observations strongly warns against interpreting `memory_max_tokens` as a stable universal runtime context limit.

Possible interpretations include persistent-memory accounting, account/product-specific budgets, an indexed corpus limit, legacy memory capacity, or semantics that changed across generations.

**None of these observations prove how many memory tokens are inserted into the answering model's prompt.**

---

# 71. Exact Client-Surface Observations for Legacy Memory and Dreaming

**Class: E — reverse engineering / frontend observation**

The following exact paths/parameters have been publicly observed and should be preserved without treating them as supported APIs.

## Legacy-looking memory surface

Observed request family:

```text
GET /backend-api/memories
```

Recent reverse-engineered clients have shown queries such as:

```text
/backend-api/memories?exclusive_to_gizmo=false&include_memory_entries=false
```

The parameter names suggest at least some client-facing scoping between ordinary/gizmo memory and whether individual memory entries are returned.

Observed source:

https://github.com/WangTianYou537/OAI-Reg/blob/main/chatgpt_login.py

## Dreaming / About You summary surface

Independent 2026 frontend analysis reports that opening/regenerating the Dreaming-era About You / Memory Summary uses:

```text
/backend-api/memories/about_you/summary/regenerate
```

Source:

https://husain-zaidi.com/chatgpt-dreaming/

## Architectural implication

The existence of distinct legacy `/memories` and newer `/memories/about_you/summary/...` client surfaces strengthens—but does not prove—the conclusion that improved-memory/Dreaming functionality is not simply a UI rename of the original saved-memory table.

The two surfaces may still share storage or services internally. That relationship remains unknown.

---

# 72. Current OpenAI Codex Memory — Uncompressed Production Mechanics

**Class: D — current adjacent OpenAI implementation, not ChatGPT proof**

The public Codex source reveals substantially more implementation detail than the earlier high-level summary preserved.

## Phase 1 — per-thread extraction

Individual rollouts/threads are processed into DB-backed stage-1 memory output records. Public code/README material describes fields/state including raw memory content, rollout summary information, thread identity, source-update time, generation time, and job state.

Failed jobs use retry/backoff behavior rather than hot-looping.

## State database

Current source shows a memory store backed by SQLite pools and explicit tables/state for stage-1 outputs and memory jobs.

This is **Codex-specific implementation evidence**, not proof that ChatGPT Dreaming uses SQLite.

## Usage tracking

When a stage-1 memory is cited/used, Codex can increment:

```text
usage_count
```

and update:

```text
last_usage
```

This provides a concrete current OpenAI implementation of memory reinforcement through use.

## Phase 2 — global consolidation

Current Codex documentation says Phase 2:

- claims a single global phase-2 lock/lease before mutating the memory workspace;
- loads a bounded set of eligible stage-1 outputs;
- ignores memories whose `last_usage` is outside the configured `max_unused_days` window;
- falls back to generation/source time for fresh memories that have never been used;
- ranks eligible memories primarily by `usage_count`, then by recent usage/generation recency;
- consolidates selected inputs into the higher-level memory workspace/artifacts.

The state code describes ranking behavior equivalent to:

```text
usage_count DESC
then recent last_usage/source time DESC
```

before returning a stable selected set.

## Retention / pruning

Codex can prune stale unselected stage-1 outputs based on a configurable maximum unused period while preserving relevant baseline/job-watermark state.

## Configuration surface

Current public Codex config includes controls such as:

```text
generate_memories
use_memories
dedicated_tools
disable_on_external_context
max_raw_memories_for_consolidation
max_unused_days
max_rollout_age_days
max_rollouts_per_startup
```

This demonstrates that memory generation, memory use, deep tools, retention, consolidation breadth, and rollout eligibility are separately configurable in a current OpenAI memory implementation.

## Why this matters

Codex is direct proof that OpenAI currently uses a production memory pattern containing:

```text
per-thread extraction
      ↓
persistent job/state DB
      ↓
usage + recency signals
      ↓
serialized global consolidation
      ↓
hierarchical memory artifacts
      ↓
selective later retrieval/use
```

It remains **adjacent evidence only**. ChatGPT's Dreaming V3 storage, models, scheduling, and ranking can differ completely.

Primary current sources:

https://github.com/openai/codex/blob/main/codex-rs/memories/README.md

https://github.com/openai/codex/blob/main/codex-rs/state/src/runtime/memories.rs

https://github.com/openai/codex/blob/main/codex-rs/config/src/types.rs

https://github.com/openai/codex/blob/main/codex-rs/memories/write/templates/memories/consolidation.md

https://github.com/openai/codex/blob/main/codex-rs/ext/memories/templates/memories/read_path.md

---

# 73. Historical Group-Chat Memory Boundary

**Class: Historical A**

The 2025–2026 Group Chats pilot provides another explicit example of a memory namespace/access boundary.

During the pilot, OpenAI documented that:

- group chats were separate from private one-to-one chats;
- personal account-level memory was not used in group chats;
- account-level custom instructions were not used in group chats;
- group conversations did not create new personal memories;
- group-specific custom instructions could exist separately;
- ChatGPT's accessible context inside the group was limited to group messages/files/images, participant metadata, and group-specific instructions.

This is important historical evidence that OpenAI treats memory access as a policy/scope decision rather than a universal account-global read.

OpenAI began winding down Group Chats on July 9, 2026; existing history was retained while chats transitioned toward read-only status. Therefore this is **historical product evidence**, not a currently expanding memory surface.

Sources:

https://openai.com/index/group-chats-in-chatgpt/

https://help.openai.com/en/articles/12703475-group-chats-in-chatgpt

https://help.openai.com/en/articles/12703475-

---

# 74. Official Product-History Details Before Dreaming V3

**Class: Historical A**

OpenAI's original Memory announcement page contains a useful dated product evolution that complements the later Dreaming article.

## February 13, 2024

OpenAI began testing Memory with a subset of Free and Plus users. Users could explicitly request memory, inspect what ChatGPT remembered, ask it to forget information, and disable the feature.

## September 5, 2024

OpenAI updated the announcement to say Memory was available to Free, Plus, Team, and Enterprise users and that ChatGPT surfaced when memories were updated.

## April 10, 2025

OpenAI described Memory as becoming more comprehensive by referencing **all past conversations** for eligible users, with two product concepts described to users:

```text
saved memories
+
chat history / insights from past chats
```

This is important historical product evidence that past-chat-derived personalization was already a first-class user-facing concept before the Dreaming V3 announcement.

## June 3, 2025

OpenAI described a lightweight memory improvement for Free users providing shorter-term continuity, while Plus/Pro memory was described as providing longer-term understanding.

The later Dreaming article clarifies that the 2025 generation included Saved Memories + Dreaming V0 and that V0 was not yet the final standalone architecture.

Source:

https://openai.com/index/memory-and-new-controls-for-chatgpt/

---

# 75. Memory Summary and Legacy Saved-Memory Management Details

**Class: A**

Current Memory FAQ details that must remain explicit:

## Memory Summary updates

- The Memory Summary is automatically updated as new context accumulates.
- The UI exposes a “last updated” time.
- Users can type requested changes into the summary-management interface.
- Users can also make corrections through the associated menu controls.

## Sources are intentionally incomplete

OpenAI says Sources are designed to improve transparency/control but **may not show every factor or source** that shaped a response.

Memory sources are also not necessarily included when a chat is shared.

## Improved memory vs legacy saved memories

The current product explicitly exposes an **improved memory** experience and a separate **legacy saved memories** mode. Users can move between them in Memory settings.

This is direct product evidence that “Memory” and “Saved Memories” should not be treated as architecturally identical labels.

## Deleted saved-memory logs

OpenAI says it may retain logs of deleted Saved Memories for up to **30 days** for safety/debugging purposes.

This retention rule applies to the legacy saved-memory system and should not be generalized into a complete Dreaming V3 data-retention architecture.

Primary source:

https://help.openai.com/en/articles/8590148

---

# 76. Project Memory Transition Semantics

**Class: A**

Current project documentation exposes additional scope-transition behavior beyond the basic “project-only” boundary.

## Switching to project-only memory

When a project is switched to project-only memory:

- information from that project is removed from memory used outside the project;
- the project's chats and files remain available inside the project;
- the change may take a few hours to fully take effect.

This is strong evidence that **memory-scope eligibility can change independently of source-data persistence**.

## Shared projects

Current shared projects are automatically set to project-only memory and cannot be switched to default memory.

They do not have access to an individual member's:

- outside-project context;
- account-level custom instructions;
- memories outside the shared project.

## Temporary Chat interaction

Temporary Chats cannot be added to projects.

## Project memory remains non-list-based

OpenAI still says there is no conventional list of project memories; same-project conversations can supply context directly.

Primary source:

https://help.openai.com/en/articles/10169521-projects-in-chatgpt

---

# 77. Expanded OpenAI Personalization-Memory Research Evidence

**Class: A — organizational/research-direction evidence**

Two OpenAI research-role families provide complementary evidence about how the company approaches memory.

## Personal AGI — Memory

The Memory role describes work spanning:

- memory architecture;
- post-training;
- long-horizon tasks for training/evaluation;
- reinforcement learning;
- general-purpose memory across ChatGPT and agentic products.

Known role URL:

https://openai.com/careers/research-engineer-research-scientist-personal-agi-memory-san-francisco/

## Personal AGI — Personalization

The Personalization role describes work including:

- reinforcement learning;
- dataset creation;
- evaluations;
- other post-training methods;
- user signals and human data;
- improving how frontier models use memory/personalization.

Source:

https://openai.com/careers/research-engineer-research-scientist-personal-agi-personalization-san-francisco/

## Interpretation boundary

Job descriptions establish organizational priorities and methods. They do **not** expose the production Dreaming V3 backend schema, model, prompts, or retrieval stack.

---

# 78. Primary Evidence Registry Addendum — Completeness Pass

The following sources were discovered or restored explicitly during the V2.1 completeness pass and must remain in the permanent evidence registry.

## OpenAI current / historical product documentation

**Original Memory announcement and 2024–2025 updates**  
https://openai.com/index/memory-and-new-controls-for-chatgpt/

**Dreaming V3**  
https://openai.com/index/chatgpt-memory-dreaming/

**Memory FAQ**  
https://help.openai.com/en/articles/8590148

**ChatGPT release notes**  
https://help.openai.com/en/articles/6825453-chatgpt-release-notes

**Projects**  
https://help.openai.com/en/articles/10169521-projects-in-chatgpt

**Temporary Chat**  
https://help.openai.com/en/articles/8914046-temporary-chat-faq

**Historical Group Chats announcement**  
https://openai.com/index/group-chats-in-chatgpt/

**Group Chats help / retirement documentation**  
https://help.openai.com/en/articles/12703475-group-chats-in-chatgpt  
https://help.openai.com/en/articles/12703475-

## OpenAI patents

**US20250200361A1**  
https://patents.google.com/patent/US20250200361A1/en

**WO2025128397A1**  
https://patents.google.com/patent/WO2025128397A1/en

**Patent 12,591,766**  
https://patents.justia.com/patent/12591766

**Patent 12,699,964**  
https://patents.justia.com/patent/12699964

**Patent 12,706,918**  
https://patents.justia.com/patent/12706918

**Patent 12,430,518**  
https://patents.google.com/patent/US12430518B2/en

## OpenAI public Codex implementation

**Memory architecture README**  
https://github.com/openai/codex/blob/main/codex-rs/memories/README.md

**Memory state/runtime code**  
https://github.com/openai/codex/blob/main/codex-rs/state/src/runtime/memories.rs

**Memory configuration types**  
https://github.com/openai/codex/blob/main/codex-rs/config/src/types.rs

**Consolidation instructions**  
https://github.com/openai/codex/blob/main/codex-rs/memories/write/templates/memories/consolidation.md

**Memory read path**  
https://github.com/openai/codex/blob/main/codex-rs/ext/memories/templates/memories/read_path.md

## Direct interview / design intent

**Limitless — interview with OpenAI memory/personalization leads, October 6, 2025**  
https://limitless.fm/episodes/we-interviewed-the-team-behind-openais-1-feature/transcript

## Empirical study

**The Algorithmic Self-Portrait: Deconstructing Memory in ChatGPT**  
https://arxiv.org/abs/2602.01450

## Client / reverse-engineering observations

**B4PT0R backend SDK documentation**  
https://github.com/B4PT0R/codex-backend-sdk/blob/main/docs/backend-api.md

**Dreaming frontend analysis**  
https://husain-zaidi.com/chatgpt-dreaming/

**OAI-Reg observed frontend bootstrap memory request**  
https://github.com/WangTianYou537/OAI-Reg/blob/main/chatgpt_login.py

**2025 memory-capacity network observation thread**  
https://linux.do/t/topic/1175093

**2024 community endpoint observation**  
https://www.reddit.com/r/ChatGPT/comments/1f879xy

**GPT2Claude Migration Kit memory export observations**  
https://github.com/Siamsnus/GPT2Claude-Migration-Kit/blob/main/CHANGELOG.md

---

# 79. Version 2.1 Change Log — Completeness Pass

**Date:** September 8, 2026

Version 2.1 does **not** materially change the central architecture conclusion from V2.0. It changes the completeness standard and restores evidence that V2.0 had compressed or omitted.

Added/restored explicitly:

- permanent no-evidence-compression rule;
- October 2025 direct interview with OpenAI memory/personalization leads;
- cognitive-memory design-intent statements from that interview;
- OpenAI's assistant/representative long-term personalization framing;
- Patent 12,706,918 as a third contemporary corroborating personalization-state patent;
- full known metadata for US20250200361A1 / WO2025128397A1;
- historical patent inventor list;
- exact illustrative note/token capacity examples from the historical patent;
- alternative historical memory representations;
- historical natural-language memory-management concepts;
- personalized-vs-non-personalized candidate-response embodiment;
- cross-thread “bad week” inference example;
- explicit Dreaming V3 2× Plus/Pro capacity statement;
- explicit ~5× Free serving/compute-efficiency statement;
- exact 2024 `memory_max_tokens≈8000` client observation;
- exact 2025 `memory_max_tokens≈5000000` client observations;
- 2026 `memory_max_tokens≈12000` reverse-engineered SDK example;
- individual memory metadata fields observed by third-party SDK tooling;
- exact Dreaming `/about_you/summary/regenerate` client path;
- `exclusive_to_gizmo` / `include_memory_entries` client parameters;
- current Codex state DB, retry/backoff, leases, usage-count, last-usage, retention, ranking, and global-lock mechanics;
- historical Group Chat memory boundary and retirement status;
- February 2024 / September 2024 / April 2025 / June 2025 official product-history details;
- current Memory Summary last-updated/correction behavior;
- legacy deleted-memory 30-day safety/debug log detail;
- project memory transition semantics;
- shared-project project-only behavior;
- both Memory and Personalization research-role evidence;
- expanded permanent evidence registry.

The purpose of this revision is explicit:

> **Future summaries may become shorter, but the evidence ledger inside this canonical file must never become less complete.**

---

# 80. Current Final Conclusion — Version 2.1

As of September 8, 2026, the combined evidence supports a highly consistent picture:

**ChatGPT memory is a layered, governed, relevance-routed long-term personalization system rather than a single memory database.**

Current product facts establish:

- Dreaming-based background synthesis;
- continuously updated derived user state;
- separate search/retrieval of specific historical conversations;
- temporal updating;
- preference/constraint memory;
- importance/salience selection;
- partial provenance through Sources;
- explicit corrections and user control;
- runtime paths that can bypass personalization;
- independently governable memory read/write behavior;
- project/workspace/source scope boundaries;
- frontier-model post-training for memory and personalization.

Contemporary OpenAI patents repeatedly describe essentially the same architectural family:

```text
persisted personalization state
        +
synthesized concepts
        +
conversation summaries / searchable history
        +
search of past interactions
        +
typed context management
        ↓
model context
```

Historical OpenAI memory design shows clear architectural ancestry:

```text
selective learned memory writes
        +
compact personalization notepad
        +
asynchronous consolidation
        +
recency/use-based retention
        +
multiple memory identities
        +
deep topic/embedding session memory
```

Current public Codex code independently demonstrates OpenAI using a production memory pattern based on:

```text
raw evidence
→ per-thread extraction
→ persistent memory/job state
→ usage + recency ranking
→ serialized consolidation
→ compact routing memory
→ deeper searchable memory
→ selective runtime use
```

Client observations add implementation clues but do not override official evidence. In particular, the huge variation in observed `memory_max_tokens` values demonstrates that those fields cannot safely be interpreted as the number of memory tokens placed into the model context.

The strongest current abstraction remains:

> **History is evidence. Dreaming compiles history into current state. Retrieval recovers relevant episodes. Governance determines what can flow where. Relevance determines what gets activated. Context management assembles what the model sees. The model decides how to use it.**

And the permanent research rule is now:

> **Preserve every material fact, source, observation, contradiction, historical value, caveat, and unknown. Summaries may compress conclusions; they must not erase evidence.**

That is the current canonical source of truth.

---

# 81. Empirical Runtime Observation — Connected Apps / Tools Can Coincide with Saved-Memory Write Failure

**Class: E — user-side reproducible observation; not an OpenAI-confirmed product rule**

On September 6, 2026, an OpenAI Developer Community user published a controlled same-conversation test suggesting that **after a connected app or tool is used, new legacy Saved Memory writes can become unavailable while reads of existing memories continue to work**.

Reported test sequence:

```text
1. Save unique memory A → succeeds
2. Use one connected app/tool once
3. Save unique memory B → fails or silently does not persist
```

The author reported reproductions involving:

- Google Drive;
- Gmail;
- Google Calendar;
- Notion;
- web search.

An earlier observed error message reportedly stated that the historical `bio` tool had been disabled because another tool incompatible with memory usage had been used in the conversation, and that the information was not saved to model-set context.

Later reproductions sometimes failed silently without that explicit error.

The same author also reported day-to-day variation, including workflows that succeeded on one day and failed the next under otherwise similar device/app/model conditions.

## What this can establish

It is evidence that at least one deployed ChatGPT memory-write path has shown **runtime compatibility or gating behavior associated with tool/connector use**.

It strengthens the architectural distinction:

```text
memory read eligibility
      ≠
memory write eligibility
      ≠
tool/action eligibility
```

## What it cannot establish

It does **not** establish whether the cause is:

- intentional product policy;
- a bug;
- a temporary rollout condition;
- a legacy Saved Memory limitation;
- a specific historical `bio`-tool restriction;
- a security boundary;
- another hidden runtime condition.

It also does not prove that Dreaming V3 background synthesis follows the same restriction.

Source:

https://community.openai.com/t/saved-memory-becomes-unavailable-after-using-a-connected-app-reproducible-within-a-session-and-not-limited-to-google-connectors/1395159

---

# 82. Empirical Failure Observation — Memory Can Exist Yet Fail at Retrieval, Activation, or Model Use

**Class: E — anecdotal/community failure evidence; not a controlled product specification**

A September 6, 2026 OpenAI Developer Community report describes cases where users believed information still existed in memory or project history but ChatGPT failed to retrieve or apply it reliably.

Reported behaviors included:

- failure to recover context from previous chats in the same Project;
- failure to apply instructions believed to be saved in Memory;
- responses that appeared to agree with user-supplied reminders rather than actually retrieving prior evidence;
- inconsistent retrieval across different conversations;
- claims that historical-context tooling returned no result.

The report also says some conversations continued to retrieve context correctly, which argues against treating it as evidence of a total memory-system outage.

## Architectural relevance

This observation supports retaining separate failure categories for:

```text
persistence
retrieval
ranking
activation
model application
provenance / truthful attribution
```

A user-visible “memory failure” does not identify which layer failed.

## Evidence boundary

This is a community bug report, not controlled laboratory evidence and not an OpenAI confirmation of the root cause. It should be used as a failure-mode observation only.

Source:

https://community.openai.com/t/chatgpt-recalls-saved-memory-but-fails-to-apply-it-and-invents-plausible-context-instead-of-verifying-history/1393739/2

---

# 83. Current Codex Adjacent Evidence — Read-Path Scope Bug and 5,000-Token Summary Injection Budget

**Class: D/E — adjacent OpenAI implementation plus public bug analysis; not ChatGPT proof**

OpenAI Codex issue #17496, opened April 11, 2026, documents an architectural mismatch between Codex's memory write/consolidation scoping and its fresh-session read path.

The issue traces current public code showing that:

- memory consolidation preserves `cwd` / project scope metadata;
- `MEMORY.md` blocks can carry `applies_to: cwd=...` style scoping;
- the consolidated summary is organized by project/cwd context;
- but a fresh-session initial-context path can read and inject the **entire global `memory_summary.md`** rather than first filtering it to the current cwd/project;
- the injected summary is truncated to **5,000 tokens** in the cited Codex code path.

The report therefore identifies a concrete adjacent failure mode:

```text
correctly scoped stored memory
        ↓
insufficiently scoped runtime injection
        ↓
irrelevant memory crowding / bias
```

The issue explicitly distinguishes this from every-turn reinjection: the reported behavior concerns the fresh-baseline/new-conversation path.

## Why this matters for the research model

Codex gives us a concrete OpenAI example where **storage scope and runtime retrieval/injection scope are separate engineering problems**. Correct metadata at write/consolidation time does not guarantee correct activation at runtime.

It also provides a concrete adjacent example of a bounded always-available memory-summary budget.

## Strict boundary

None of this proves that ChatGPT Dreaming V3:

- uses `memory_summary.md`;
- has a 5,000-token memory budget;
- uses cwd-style scoping;
- has the same bug.

It belongs only in the adjacent OpenAI implementation evidence class.

Source:

https://github.com/openai/codex/issues/17496

---

# 84. Current Official Documentation Inconsistency — Temporary Chat General vs Feature-Specific Wording

**Class: A — documented product-language inconsistency with resolution by source precedence**

As of September 8, 2026, current OpenAI documentation contains two different levels of specificity about Temporary Chat.

The general Memory FAQ uses blanket wording that Temporary Chats do not use existing memories or create new memories.

The newer, feature-specific Temporary Chat FAQ and August 27, 2026 release notes explicitly distinguish two modes:

```text
non-personalized Temporary Chat
→ does not use memory
→ does not create memory

personalized Temporary Chat
→ can use existing memories
→ can use custom instructions/plugins
→ does not create or update memories while temporary
```

The dedicated Temporary Chat FAQ also says account/workspace restrictions take priority over the per-chat personalization choice and that limited prior-conversation context can still be used for safety/security purposes.

## Resolution

Under this repository's source-precedence rule, the **newer, feature-specific Temporary Chat documentation controls the present-state conclusion**.

The general Memory FAQ wording should be preserved as a documentation inconsistency rather than promoted over the dedicated feature documentation.

Sources:

https://help.openai.com/en/articles/8590148

https://help.openai.com/en/articles/8914046-temporary-chat-faq

https://help.openai.com/en/articles/6825453-chatgpt-release-notes

---

# 85. Improved Memory Is Independently Governed in Regulated Workspaces

**Class: A — current product governance evidence**

The current Memory FAQ explicitly says **improved memory is distinct from Memory generally and from legacy Saved Memories**.

For ChatGPT for Healthcare and Enterprise Regulated Workspace, OpenAI says improved memory is **disabled by default** and workspace owners/admins can make it available to eligible roles.

OpenAI further notes that this improved-memory functionality is not covered under the referenced BAA guidance and directs organizations that want tighter boundaries toward Project-only memory.

## Architectural implication

This adds another confirmed governance dimension:

```text
feature exists at product level
        ↓
workspace policy gate
        ↓
role eligibility
        ↓
user availability
        ↓
project-level scope can further constrain context
```

That is direct current evidence that memory capability, memory scope, and authorization are distinct layers.

Primary source:

https://help.openai.com/en/articles/8590148

---

# 86. Version 2.2 Change Log — Runtime / Failure / Governance Completeness Pass

**Date:** September 8, 2026

Version 2.2 preserves all V2.1 evidence and adds newly surfaced runtime and failure observations without changing the central architecture conclusion.

Added:

- September 2026 empirical connected-app/tool memory-write failure observation;
- historical `bio`-tool incompatibility error wording as an observed runtime clue;
- explicit distinction between read eligibility, write eligibility, and tool compatibility;
- September 2026 empirical retrieval/activation/model-use failure report;
- preservation of truthful-provenance failure as a separate memory failure class;
- Codex issue #17496 as adjacent evidence for write-scope vs runtime-injection-scope mismatch;
- Codex fresh-session whole-summary injection observation;
- explicit **5,000-token** Codex summary injection cap from the cited code path;
- current official Temporary Chat documentation inconsistency;
- source-precedence resolution favoring the newer feature-specific Temporary Chat FAQ;
- limited safety/security prior-context exception for Temporary Chat;
- regulated-workspace improved-memory controls;
- explicit role/workspace authorization as a memory-governance dimension.

No new empirical observation in this revision is promoted to a confirmed Dreaming V3 production fact.

---

# 87. Current Final Conclusion — Version 2.2

As of September 8, 2026, the strongest public evidence still supports the same core architecture:

**ChatGPT memory is a layered, governed, relevance-routed long-term personalization system rather than a single memory database.**

The V2.2 evidence sharpens one additional principle:

> **Memory correctness depends not only on what is stored, but on whether the runtime is allowed to read it, allowed to write it, able to retrieve it, scoped to the right context, and able to apply it truthfully and correctly.**

Current product facts establish Dreaming-based synthesis, historical conversation retrieval, partial provenance, explicit user controls, scope boundaries, relevance gating, and model post-training. Contemporary OpenAI patents repeatedly describe synthesized personalization state plus searchable historical interactions. Historical patents show a clear lineage of selective memory writing, consolidation, deep episodic retrieval, and scoped identities. Current Codex provides adjacent public implementation evidence for hierarchical memory, usage/recency ranking, bounded prompt memory, and the fact that stored scope and runtime activation scope can fail independently.

The strongest overall abstraction remains:

> **History is evidence. Dreaming compiles history into current state. Retrieval recovers relevant episodes. Governance determines what can flow where. Relevance determines what gets activated. Context management assembles what the model sees. The model decides how to use it.**

And the permanent evidence rule remains:

> **Preserve every material fact, source, observation, contradiction, historical value, caveat, and unknown. Summaries may compress conclusions; they must not erase evidence.**

That is the current canonical source of truth.
