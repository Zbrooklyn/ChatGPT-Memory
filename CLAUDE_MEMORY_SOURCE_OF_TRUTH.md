# Claude Memory — Master Source of Truth

**Version:** 1.0  
**Last verified:** September 8, 2026  
**Scope:** Claude consumer Chat memory, Projects, cloud Cowork, past-chat search, memory import/export, sensitive-memory controls, Claude Code memory, `CLAUDE.md`, subagent memory, Claude API Memory Tool, Managed Agent Memory Stores, Dreams, current implementation evidence, architectural inferences, and hard unknowns.  
**Purpose:** Maintain one canonical, evidence-graded account of what is publicly known about Claude memory, what is strongly corroborated by implementation evidence, what can reasonably be inferred, and what remains unknown.

---

# 1. Bottom Line

The strongest evidence supports this conclusion:

**Claude does not have one single memory system. It has a family of external persistence and retrieval mechanisms that differ by product surface. Consumer Claude uses individual memory topics/files plus historical chat retrieval; Projects add isolated memory, summaries, instructions, and knowledge RAG; Claude Code uses local file memory and instructions; the API exposes a developer-owned filesystem-style memory abstraction; Managed Agents add persistent memory stores, versioning, concurrency control, read/write scopes, and offline consolidation through Dreams.**

The most important general architectural principle is:

> **The model is not the durable memory. Durable state lives outside the model and is selectively supplied back into context.**

Current best reconstruction:

```text
                         AUTHORITATIVE SOURCES
                 repos | docs | email | APIs | DBs
                              │
                              ▼
                            TOOLS
                              │

 HISTORICAL EVIDENCE ─── CURRENT CLAUDE ─── PROJECT KNOWLEDGE
 chat history / RAG            │                  RAG
                               │
                               ▼
                            RESPONSE
                               │
                               ▼
                    MEMORY MAINTENANCE
                               │
                    durable-user-fact filter
                               │
                    subject/entity routing
                               │
                    correction/provenance
                               │
                    safe persistent update
                               │
                               ▼
                         DURABLE MEMORY
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
            compact index              detailed files
                 │                           │
                 └──────── JIT retrieval ────┘
                               │
                               ▼
                         FUTURE CLAUDE

                               +
                      periodic consolidation
                               │
                               ▼
                            DREAMS
                  (Managed Agents currently)
```

**The complete diagram is a reconstruction. Its major individual components are independently supported by current Anthropic documentation and contemporary Claude implementations.**

---

# 2. Evidence Standard

Every claim in this document belongs to one of four classes.

## A — Confirmed Anthropic Product Fact

Explicitly documented by Anthropic in current Claude product documentation, release notes, Claude Code documentation, or Claude API documentation.

This is the highest authority.

## B — Strong Implementation Evidence

Observed in current Claude system-prompt captures, exports, client behavior, or other implementation evidence that aligns closely with official functionality but is **not an Anthropic-authenticated public contract**.

Useful for understanding likely internals; not guaranteed.

## C — Architectural Inference

A conclusion strongly suggested by multiple A/B facts but not directly stated by Anthropic.

Must remain labeled as inference.

## U — Unknown

No sufficient public evidence.

Unknowns must not be filled with plausible assumptions.

---

# 3. Source Precedence

When sources conflict, use this order:

1. Newer, feature-specific Anthropic documentation.
2. Current Anthropic general documentation.
3. Current Claude Code / API / Managed Agent documentation.
4. Older Anthropic documentation.
5. Current reproducible implementation evidence.
6. Captured system prompts / exports / client observations.
7. Community speculation.

Freshness matters even among official sources.

---

# 4. Consumer Claude Memory Changed in July 2026

**Class: A**

Anthropic changed Claude consumer memory on **July 10, 2026**.

The previous architecture relied on a synthesized memory summary refreshed periodically.

The new architecture creates **individual categorized memory entries/topics during conversations** rather than depending on one global daily synthesis.

Conceptually:

```text
OLD
many chats
  ↓
periodic global synthesis
  ↓
one memory summary

NEW
conversation
  ↓
individual categorized memory entries
  ↓
read/update later
```

Primary source:

- https://support.claude.com/en/articles/12138966-release-notes

A small number of Team/Enterprise organizations may still temporarily use the legacy system during migration.

Primary source:

- https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

---

# 5. Consumer Memory Is Explicitly File-Oriented in the Product

**Class: A**

Anthropic describes Claude's current remembered information as a **list of files under Topics** in Memory settings.

These are individually inspectable and editable rather than one opaque global profile.

Conceptually:

```text
Memory
 ├── topic/file
 ├── topic/file
 ├── topic/file
 └── ...
```

Primary sources:

- https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

This does **not** prove Anthropic physically stores Markdown files on disk.

The model-facing abstraction may map to database objects or object storage.

**Physical backend: U**

---

# 6. Claude Creates Memory While You Chat

**Class: A**

Claude now updates memory during ordinary conversations rather than waiting for a daily summary cycle.

Anthropic says Claude adds useful information as you chat.

Users can also explicitly request memory operations such as:

```text
Remember this.
Remember that X.
Change my memory about X.
Forget X.
```

Changes apply to subsequent conversations.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 7. What Consumer Claude Tries to Remember

**Class: A**

Anthropic says memory can include useful durable context such as:

- professional role;
- projects and ongoing work;
- important people;
- important places;
- communication preferences;
- working style;
- technical preferences;
- coding preferences;
- project details;
- decisions and constraints useful in future work.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

Memory admission is selective.

Therefore:

```text
mentioned
   ≠
automatically remembered
```

The exact admission scoring/classifier is unknown.

---

# 8. Chat and Cloud Cowork Share Memory

**Class: A**

On **August 25, 2026**, Anthropic expanded consumer memory so **Claude Chat and cloud Cowork share the same memory**.

Conceptually:

```text
             shared memory
            ↗             ↖
       Claude Chat     Cloud Cowork
```

Primary source:

- https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it

Local Cowork sessions do not participate in that same cloud memory behavior.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

There is no current official evidence that Claude Code auto-memory is synchronized with this same consumer store.

Therefore:

```text
Chat ↔ cloud Cowork
        shared

Claude Code auto-memory
        separate
```

**Class: A/C**

---

# 9. Projects Have Isolated Memory

**Class: A**

Each Claude Project has its **own memory space** and a **dedicated project summary**.

That project memory is isolated from:

- normal non-project Claude memory;
- other Projects.

Conceptually:

```text
Claude account
│
├── non-project memory
│
├── Project A
│   ├── isolated memory
│   └── dedicated summary
│
└── Project B
    ├── isolated memory
    └── dedicated summary
```

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

Moving chats into or out of a Project changes which memory domain they belong to.

Primary source:

- https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects

---

# 10. Project Memory, Project Summary, and Project Knowledge Are Different

**Class: A**

A Project can simultaneously contain several distinct context systems:

```text
current conversation
+
project instructions
+
project memory
+
dedicated project summary
+
project conversation history
+
project chat search
+
project knowledge/files
+
project knowledge RAG
```

These should not all be called “memory.”

Project Knowledge is a separate knowledge base that Claude may load directly while it fits and retrieve through RAG as it grows.

Anthropic says enhanced Project RAG can expand practical knowledge capacity by up to approximately **10×**.

Primary source:

- https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects

As of the current documentation, Project RAG is available on:

- Free;
- Pro;
- Max;
- Team;
- Enterprise.

**Class: A**

---

# 11. Past Chat Search Is Separate From Memory

**Class: A**

Claude can retrieve information from old conversations through a separate historical-search capability.

Anthropic explicitly describes past-chat search as a **RAG** mechanism.

Search boundaries follow project boundaries:

```text
non-project conversation
   ↓
search non-project chat history

Project A conversation
   ↓
search Project A chat history only
```

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

Therefore Claude can appear to “remember” in two different ways:

```text
Saved memory
“I know this about the user.”

Historical retrieval
“I found the old conversation where this occurred.”
```

These are not the same mechanism.

---

# 12. Past Chat Search Availability

**Class: A**

Past-chat search is currently documented for:

- Pro;
- Max;
- Team;
- Enterprise.

It is available on web, Desktop, and Mobile.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

Chat Search and saved memory can be controlled independently.

Enterprise organizations using customer-managed encryption keys cannot currently use past-chat search because conversation contents are encrypted.

**Class: A**

---

# 13. Memory Pause and Reset Have Different Semantics

**Class: A**

## Pause Memory

Claude:

- keeps existing memories;
- does not use them;
- does not create new memories;
- does not retroactively learn conversations held while memory was paused.

Thus:

```text
Pause = stop READ + stop WRITE
```

## Reset Memory

Reset:

- deletes all memory;
- includes project memory;
- is irreversible.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 14. Deleting a Chat Does Not Delete Derived Memories

**Class: A**

In the modern memory system, deleting the source conversation does **not automatically delete memory entries already generated from it**.

The memory must be removed separately.

Conceptually:

```text
conversation
   ↓
memory created
   ↓
conversation deleted
   ↓
memory may remain
```

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

This demonstrates that conversation storage and memory storage are distinct persistent objects.

---

# 15. Memory Import and Export

**Class: A**

Claude supports first-class memory portability.

Users can import memory generated by another AI provider.

Claude parses the imported material and extracts useful information into individual memory entries.

Imports are currently documented for:

- Free;
- Pro;
- Max;
- Team;

through:

- Web;
- Claude Desktop.

Primary source:

- https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

Anthropic describes import as experimental and warns that not every item will necessarily be retained.

Users can also export memory and can ask Claude to output the memory it currently sees.

**Class: A**

Architectural implication:

> Consumer AI memory is becoming portable user state rather than a completely opaque vendor-only artifact.

**Class: C**

---

# 16. Legacy-Memory Migration

**Class: A**

The old memory architecture used a synthesized summary rather than individual memory files/topics.

Anthropic replaced that model on July 10, 2026.

As of **September 8, 2026**, users who suspect migration loss can still export legacy memory until **September 9, 2026**.

Primary sources:

- https://support.claude.com/en/articles/12138966-release-notes
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

A small number of Team/Enterprise organizations may still temporarily use the legacy system.

---

# 17. Sensitive Memory Controls

**Class: A**

By default Claude avoids automatically saving many sensitive categories, including areas such as:

- health;
- race;
- ethnicity;
- religion;
- political beliefs;
- gender identity.

Users can enable:

**Include sensitive topics in memory.**

When enabled:

- only future material becomes eligible;
- it is not retroactive;
- Claude notifies the user when sensitive memory is saved;
- disabling the setting removes sensitive items previously stored under that mechanism.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

Anthropic also documents categories that Claude will not store even if requested, including:

- government ID numbers;
- financial account numbers;
- criminal history;
- immigration status.

**Class: A**

---

# 18. Incognito Creates a Memory/History Boundary

**Class: A**

Incognito chats:

- do not use normal Claude memory;
- do not create normal Claude memory;
- do not appear in normal chat history;
- are not available to historical chat search;
- are excluded from Monthly Recap.

Primary source:

- https://support.claude.com/en/articles/12260368-use-incognito-chats

Incognito chats may still receive other personalization such as profile/custom style.

Therefore:

```text
MEMORY ≠ ALL PERSONALIZATION
```

Incognito conversations are retained for a limited period, 30 days by default unless organizational retention settings differ.

**Class: A**

---

# 19. Team and Enterprise Governance

**Class: A**

Memory is currently:

### Enabled by default
- Free
- Pro
- Max

### Organizationally disabled by default
- Team
- Enterprise

Owners can enable memory availability, after which individual users manage their own memories.

Owners cannot inspect or edit an individual's memory through the memory controls.

If an organization owner disables memory at the organization level, existing organizational memories are immediately and permanently deleted.

Memory is currently unavailable for some organizations using:

- HIPAA configurations;
- public-sector arrangements;
- custom data-retention agreements.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 20. Monthly Recap Is Memory-Adjacent, Not Memory

**Class: A**

Claude includes a reflective feature under Settings → Reflect → Monthly Recap.

It can analyze recent history to surface:

- topic distribution;
- usage/activity patterns;
- peak usage times;
- observations about working patterns;
- AI-fluency suggestions.

It requires memory to be enabled but is not itself the persistent memory store.

Primary source:

- https://support.claude.com/en/articles/15672559-see-your-monthly-recap

Important exclusions include:

- Incognito;
- Health integration chats;
- Cowork;
- Claude Code.

**Class: A**

---

# 21. Strong Implementation Evidence: Consumer Memory Filesystem

**Class: B**

A publicly captured Fable 5.1 claude.ai system prompt describes a model-facing component named approximately:

```text
<memory_filesystem>
```

and a persistent cross-session working-memory filesystem.

Implementation evidence describes operations corresponding to:

```text
memory_read
memory_write
memory_str_replace
memory_append
memory_list
memory_delete
```

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

This capture is **not an Anthropic-authenticated product specification**.

It is nevertheless strongly consistent with:

- Anthropic's official “files under Topics” language;
- Claude Code's file memory;
- the Claude API Memory Tool;
- Managed Agent memory stores.

Therefore it is useful implementation evidence, but not contractual fact.

---

# 22. Strong Implementation Evidence: Apparent Consumer File Structure

**Class: B**

The captured Fable 5.1 instructions describe an apparent organization resembling:

```text
/profile.md
/preferences.md
/topics/...
/areas/...
/people/...
```

Approximate semantics:

### `/profile.md`
Stable identity/context expected to remain true for a long horizon.

### `/preferences.md`
How Claude should interact/respond.

### `/topics/`
Recurring interests, routines, habits, or general subject areas.

### `/areas/`
Ongoing projects, responsibilities, decisions, and work domains.

### `/people/`
Persistent context about recurring people and relationships.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

Exact names and organization remain **B-level**, not officially guaranteed.

---

# 23. Strong Implementation Evidence: Sparse Retrieval

**Class: B/C**

The captured Fable 5.1 prompt indicates an architecture in which:

- profile information is directly available;
- preferences are directly available;
- Claude receives a compact memory listing/index;
- detailed topic/area/person files are read selectively when relevant.

Conceptually:

```text
always available
  profile
  preferences
  memory index/listing

on demand
  detailed memory files
```

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

This strongly resembles Claude Code's officially documented **index + lazy retrieval** architecture.

---

# 24. Strong Implementation Evidence: Background Memory Pass

**Class: B**

The captured Fable 5.1 system instructions describe two memory-writing modes:

1. a **background memory pass** after each completed assistant turn; and
2. foreground mutation when the user explicitly requests remember/change/forget behavior.

Probable flow:

```text
USER MESSAGE
   ↓
MAIN CLAUDE
   ↓
NORMAL RESPONSE
   ↓
turn completes
   ↓
BACKGROUND MEMORY PASS
   ├── review exchange
   ├── decide what is durable
   ├── classify destination
   ├── inspect existing memory
   ├── reconcile old/new state
   └── update persistent memory
```

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

Anthropic has not publicly documented which model/process performs that background pass.

**Background writer model: U**

---

# 25. Explicit Remember/Forget Appears to Use a Foreground Path

**Class: B**

When the user explicitly says:

```text
remember X
update X
forget X
```

the captured instructions indicate the foreground Claude should perform the memory mutation immediately.

The automatic background writer then avoids reprocessing that turn in a way that could duplicate or reverse the explicit action.

This is especially important for forgetting semantics.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

---

# 26. Strong Evidence for Provenance / Epistemic Typing

**Class: B**

The captured memory representation includes evidence/provenance concepts resembling:

```text
[stated]
[observed]
[inferred]
```

The captured Chat behavior appears particularly strict about writing user-established information rather than automatically converting Claude-generated conclusions into autobiographical user memory.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

This implies a distinction such as:

```text
STATED
user explicitly established it

OBSERVED
another surface/process observed it

INFERRED
another surface/process inferred it
```

The exact implementation and whether all surfaces use these tags are not officially documented.

---

# 27. Claude-Generated Advice Is Apparently Not Automatically User Memory

**Class: B**

Under the captured rules:

```text
Claude: “Stripe looks like the best choice.”
```

is not itself a durable user fact.

If the user later says:

```text
“Yes, we're going with Stripe.”
```

that confirmation becomes user-established and may become memory.

Likewise, search results and connector data are generally not supposed to become autobiographical memory merely because Claude retrieved them.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

Architectural implication:

> **Do not silently convert model outputs or re-queryable external facts into durable user beliefs.**

**Class: C**

---

# 28. “Remember the Pointer, Not the Stale Copy” Is a Cross-Anthropic Principle

**Class: A/C**

Claude Code independently confirms this design principle.

Its auto-memory system includes a `reference` type for remembering **where authoritative information lives**, rather than duplicating dynamic values from that external source.

Claude Code also avoids storing technical information it can recover from the repository itself.

Primary source:

- https://code.claude.com/docs/en/memory

Conceptually:

```text
CAN REALITY BE RELIABLY RECONSTRUCTED?

YES
→ don't duplicate it into adaptive memory

NO / costly / user-specific
→ candidate for durable memory
```

This is one of the strongest reusable design lessons in Anthropic's memory architecture.

---

# 29. Memory Admission Appears to Use Durability and Repetition

**Class: B**

The captured Fable 5.1 prompt suggests memory admission is calibrated by durability.

Stable identity information has a long expected lifespan.

Passing execution state is generally not worth storing.

Casual one-off tastes may require repetition or meaningful engagement before becoming durable memory.

The apparent profile rule uses a longer-horizon stability test, approximately whether something remains true months later.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

Exact scoring formula: **U**

---

# 30. Apparent Entity Resolution Through Aliases

**Class: B**

Captured consumer memory files appear to support aliases, particularly for people and ongoing areas.

Example:

```text
David
Dave
David from Crystal Tile
```

may resolve to one canonical subject rather than three duplicated memories.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

Architectural implication:

> Long-term memory needs entity resolution or it fragments into contradictory duplicates.

**Class: C**

---

# 31. Apparent Cross-Memory Links

**Class: B/C**

Captured memory files also appear capable of linking to related memories.

Semantically the memory layer therefore resembles:

```text
ENTITY
 ├── canonical identity/path
 ├── aliases
 ├── facts
 ├── provenance
 └── relationships
```

A useful description is:

> **Markdown-shaped lightweight knowledge graph.**

This is our architectural terminology, not Anthropic's.

---

# 32. Apparent Cross-Surface Source Metadata

**Class: B**

The captured memory format includes source metadata indicating which Claude surfaces have written a memory.

A conceptual example is:

```yaml
sources:
  - chat
  - cowork
```

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

This aligns strongly with Anthropic's official statement that Chat and cloud Cowork share one memory.

Exact metadata: **B**  
Shared Chat/Cowork memory: **A**

---

# 33. Apparent Optimistic Concurrency in Consumer Memory

**Class: B**

The captured memory tools accept version-like preconditions (`if_version`).

Probable flow:

```text
read file → version A

another surface writes → version B

Claude tries update(version A)
        ↓
conflict
        ↓
re-read/merge/retry
```

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

This pattern is independently confirmed in Anthropic's official Managed Agent memory design, which uses content SHA-256 preconditions for concurrency-safe writes.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

Thus concurrency-safe memory is clearly a broader Anthropic design pattern.

---

# 34. Apparent Consumer Memory File-Size Management

**Class: B**

The captured prompt says individual consumer memory files are size-capped and Claude is given information about remaining capacity.

When a file becomes crowded, Claude is instructed to:

- consolidate overlapping facts;
- remove stale details;
- split broad topics;
- summarize repetitive history;
- preserve links/pointers to external canonical records instead of copying them.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

Exact consumer file-size limit: **U**

---

# 35. Correction and Forgetting Are Semantically Different

**Class: B**

Captured instructions distinguish a normal correction from explicit forgetting.

Example correction:

```text
“I work on Infrastructure now instead of Search.”
```

may preserve useful chronology:

```text
works on Infrastructure; previously Search
```

But:

```text
“Forget that I ever worked on Search.”
```

should remove the old fact rather than preserve it as history.

Captured instructions also suggest facts derived solely from the forgotten information should be removed.

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

This approaches provenance-aware cascading deletion, though no formal dependency graph is publicly documented.

---

# 36. Memory Is a Security Boundary

**Class: A/B/C**

Captured consumer instructions treat persistent memory as potentially unsafe context that must not silently override core behavior or the user's current explicit request.

Separately, Anthropic officially warns Managed Agent developers that untrusted content can prompt an agent to write poisoned data into persistent memory, causing later sessions to ingest malicious instructions.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

Recommended defense includes read-only memory mounts where write access is unnecessary.

Architectural conclusion:

> **Persistent memory expands prompt-injection risk across future sessions and therefore must be treated as a security boundary.**

**Class: C**

---

# 37. Consumer Memory Appears “Best Effort,” Not Authoritative

**Class: B/C**

The captured consumer prompt describes memory as effectively best-effort rather than load-bearing.

If memory maintenance fails, the user's main task should continue.

This is an important architectural distinction:

```text
AUTHORITATIVE STATE
repo
DB
calendar
email
documents

ADAPTIVE MEMORY
helpful context
not canonical truth
```

Observed source:

- https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

---

# 38. Claude Code Memory Is a Separate System

**Class: A**

Claude Code has its own documented persistence architecture and should not be conflated with consumer Claude memory.

Claude Code distinguishes:

```text
CLAUDE.md
human-authored instructions

Auto Memory
Claude-authored learned context
```

Primary source:

- https://code.claude.com/docs/en/memory

They solve different problems.

---

# 39. Claude Code Auto-Memory Types

**Class: A**

Current Claude Code auto-memory supports four official types:

### `user`
Role, expertise, working preferences.

### `feedback`
Corrections and confirmed approaches.

### `project`
Ongoing work, deadlines, decisions not recoverable from repository/git state.

### `reference`
Where authoritative external information can be found.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 40. Claude Code Avoids Reconstructible Memory

**Class: A**

Claude Code's documentation explicitly discourages auto-memory from duplicating information it can recover from the repository or existing instructions.

Examples include:

- architecture visible in the codebase;
- file paths visible in the codebase;
- debugging conclusions that can be reproduced;
- information already present in `CLAUDE.md`.

Primary source:

- https://code.claude.com/docs/en/memory

This is one of the clearest official statements of Anthropic's memory philosophy:

> **Persist information that would otherwise be lost or expensive to reconstruct.**

---

# 41. Claude Code Auto-Memory Storage

**Class: A**

Claude Code stores auto-memory locally under an approximate path:

```text
~/.claude/projects/<project>/memory/

├── MEMORY.md
├── user_role.md
├── feedback_testing.md
├── project_deadline.md
└── ...
```

Primary source:

- https://code.claude.com/docs/en/memory

Properties:

- repository-scoped;
- shared across worktrees of the same repository;
- machine-local;
- not automatically synchronized between machines/cloud environments.

**Class: A**

---

# 42. Claude Code Uses Index + Lazy Retrieval

**Class: A**

`MEMORY.md` functions as the lightweight memory index.

At conversation startup Claude automatically receives approximately:

```text
first 200 lines
OR
first 25 KB
whichever comes first
```

Detailed topic files are not all loaded automatically.

Claude reads them on demand when needed.

Primary source:

- https://code.claude.com/docs/en/memory

Architecture:

```text
MEMORY.md
small index
    │
    ├── user_role.md
    ├── feedback_testing.md
    ├── project_x.md
    └── reference_y.md

Claude loads details only when useful.
```

This is explicit hierarchical sparse retrieval.

---

# 43. Claude Code Memory Survives Transcript Cleanup

**Class: A**

Claude Code may clean up older session transcripts according to retention configuration.

Auto-memory files are excluded from that transcript cleanup and remain until Claude or the user edits/removes them.

Primary source:

- https://code.claude.com/docs/en/memory

Thus:

```text
conversation history
≠
persistent memory
```

---

# 44. Claude Code Auto-Memory Can Be Relocated Safely

**Class: A**

Claude Code supports an `autoMemoryDirectory` setting.

Anthropic intentionally refuses unsafe project-local control of this setting because a malicious repository could redirect memory writes into sensitive filesystem paths.

Trusted user/policy settings can control it instead.

Primary source:

- https://code.claude.com/docs/en/memory

This is another concrete example of memory being treated as a privileged filesystem capability.

---

# 45. CLAUDE.md Is Guidance, Not Deterministic Enforcement

**Class: A/C**

`CLAUDE.md` contains durable human-authored instructions.

But it is still model context, not a hard constraint system.

For reliable agent architecture, the layers should be distinguished:

```text
Claude should KNOW X
→ memory

Claude should generally DO X
→ CLAUDE.md / instruction / skill

Claude MUST DO or MUST NOT DO X
→ permissions / hooks / deterministic policy

Did the result actually satisfy X?
→ tests / verification
```

Primary source:

- https://code.claude.com/docs/en/memory

---

# 46. Claude Code Memory and Compaction

**Class: A**

Claude Code documents what survives or reloads after context compaction.

Persistent sources such as root `CLAUDE.md`, unscoped rules, and auto-memory can be reintroduced from disk.

Path-specific/nested instructions reload when relevant files/subtrees are accessed.

Primary sources:

- https://code.claude.com/docs/en/memory
- https://code.claude.com/docs/en/context-window

This establishes a fundamental rule:

> **Context compaction is not persistent memory deletion.**

Memory survives because it exists outside the active context window.

---

# 47. Claude Code Subagents Can Have Separate Persistent Memory

**Class: A**

Claude Code subagents can declare persistent memory scopes such as:

```text
memory: user
memory: project
memory: local
```

Primary source:

- https://code.claude.com/docs/en/sub-agents

This enables role-specialized long-term memory:

```text
Main agent
 ├── reviewer memory
 ├── security memory
 ├── design memory
 └── debugging memory
```

A multi-agent system therefore does not need one universal memory pool.

---

# 48. Claude API Memory Tool

**Class: A**

Anthropic exposes a developer-facing Memory Tool for the Claude API.

Claude sees a directory resembling:

```text
/memories/
```

and can request operations such as:

```text
view
create
str_replace
insert
delete
rename
```

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

Crucially, this is a **client-implemented** storage abstraction.

The developer decides whether `/memories/foo.md` maps to:

- filesystem;
- Postgres;
- S3/object storage;
- encrypted storage;
- another system.

Therefore:

> A model-facing filesystem does not prove a physical filesystem backend.

**Class: A/C**

---

# 49. API Memory Is Designed for Just-in-Time Retrieval

**Class: A**

Anthropic explicitly presents persistent memory as a way to keep durable state outside the active model context and retrieve only what is needed.

Primary sources:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool
- https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools

Architecture:

```text
store externally
      ↓
retrieve selectively
      ↓
keep active context focused
```

This is the same broad pattern seen in Claude Code and strongly suggested by consumer Claude.

---

# 50. API Memory and Compaction Solve Different Problems

**Class: A/C**

Anthropic recommends combining persistent memory with context editing/compaction for long-running agents.

### Active Context
Temporary working cognition.

### Compaction Summary
Compressed current-session continuity.

### Persistent Memory
Cross-session durable context.

### Authoritative External Systems
Ground truth.

Conceptually:

```text
ACTIVE CONTEXT
      ↓
COMPACTION SUMMARY
      ↓
PERSISTENT MEMORY
      ↓
AUTHORITATIVE SOURCES
```

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

---

# 51. Anthropic Explicitly Recommends Memory Expiration

**Class: A**

For developer implementations Anthropic recommends controls such as:

- file-size caps;
- paging;
- sensitive-data validation;
- path-traversal protection;
- removal of old/unused memory.

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

Architectural conclusion:

> **Forgetting/garbage collection is a design feature, not only a privacy action.**

**Class: C**

---

# 52. Managed Agent Memory Stores

**Class: A**

Managed Agents provide persistent **Memory Stores**.

A store is a workspace-scoped collection of text documents that can be mounted into agent sessions.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

When attached, the memory store appears in the sandbox under a mounted directory resembling:

```text
/mnt/memory/<store-name>/
```

Writes inside a read/write mount persist across sessions.

**Class: A**

---

# 53. Managed Memory Limits

**Class: A**

Current documented limits include:

### Per memory
Approximately **100 KB**, roughly **25K tokens**.

### Per memory store
Up to **2,000 memories**.

### Mounted stores per session
Up to **8 memory stores**.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

Anthropic recommends many small focused memories instead of a small number of huge documents.

---

# 54. Managed Memory Read/Write Scopes

**Class: A**

Memory stores can be mounted as:

```text
read_write
```

or:

```text
read_only
```

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

This supports architectures such as:

```text
Agent
 ├── company standards      READ ONLY
 ├── team knowledge         READ ONLY
 ├── user preferences       READ/WRITE
 └── project memory         READ/WRITE
```

This is both a correctness and security mechanism.

---

# 55. Managed Memory Supports Optimistic Concurrency

**Class: A**

Managed Agent Memory supports concurrency-safe updates using a content SHA-256 precondition.

Conceptually:

```text
read content + hash A
        ↓
another writer changes it
        ↓
write expecting hash A fails
        ↓
re-read
merge
retry
```

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

This is strong official evidence for Anthropic treating durable memory as shared mutable state that needs normal concurrency controls.

---

# 56. Managed Memory Has Immutable Version History

**Class: A**

Every Managed Agent memory mutation creates a version record.

Version history supports:

- what changed;
- who changed it;
- when it changed;
- restoring older content;
- redacting historical sensitive data.

Actors can include:

- agent sessions;
- API keys;
- human Console users;
- service accounts.

Primary sources:

- https://platform.claude.com/docs/en/managed-agents/memory
- https://platform.claude.com/docs/en/api/http/beta/memory_stores/memory_versions

Historical versions are generally retained for approximately 30 days, with recent versions guaranteed and some infrequently changed history potentially surviving longer.

**Class: A**

---

# 57. Prompt Injection Can Persist Through Managed Memory

**Class: A**

Anthropic explicitly warns that an agent processing untrusted content while holding writable memory access may be induced to persist malicious instructions.

A future session could then read the poisoned memory.

Threat model:

```text
untrusted input
    ↓
prompt injection
    ↓
writable memory
    ↓
persistent malicious state
    ↓
later session reads it
```

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

Recommended mitigation includes using `read_only` for shared/reference memory whenever writes are unnecessary.

---

# 58. Dreams: Anthropic's Memory-Consolidation Layer

**Class: A**

Managed Agents include a research-preview feature called **Dreams**.

Anthropic explicitly identifies a long-term memory problem:

incremental memory accumulates:

- duplicates;
- contradictions;
- stale information;
- fragmented organization.

A Dream receives:

```text
existing memory store
+
1–100 historical sessions
```

and produces:

```text
NEW memory store
```

with the material reorganized and consolidated.

The original store remains unchanged.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/dreams

---

# 59. Dreams Create a Two-Speed Memory Architecture

**Class: A/C**

Anthropic now has a documented architecture that supports two memory loops.

## Fast Loop

```text
conversation
   ↓
incremental memory update
```

## Slow Loop

```text
many sessions
+
existing memory
   ↓
Dream
   ↓
deduplicate
reconcile
compress
reorganize
generalize
   ↓
clean new memory store
```

Architectural lesson:

> **Persistent memory needs consolidation, not merely accumulation.**

---

# 60. Core Anthropic Memory Design Principles

Across consumer Claude, Claude Code, API Memory, Managed Agents, and Dreams, at least ten strong design principles recur.

## 1. Externalize durable state

Do not make the model context window itself the persistence mechanism.

## 2. Keep active context sparse

Use indexes and just-in-time retrieval rather than stuffing everything into every prompt.

## 3. Separate truth from memory

Repositories, databases, documents, and connected systems remain canonical.

## 4. Store what cannot cheaply be reconstructed

Do not turn memory into a stale duplicate database.

## 5. Preserve provenance

A user statement is not equivalent to a model inference or search result.

## 6. Scope memory

Projects, agents, people, and roles should not necessarily share all memory.

## 7. Make memory editable/auditable

Persistent model beliefs need inspection, correction, and—in enterprise systems—version history.

## 8. Protect concurrent writes

Shared durable state needs version/conflict handling.

## 9. Treat memory as a security boundary

Writable long-term memory converts transient prompt injection into persistent compromise.

## 10. Consolidate and forget

Memory accumulation without deduplication, reconciliation, pruning, and expiration eventually becomes garbage.

---

# 61. Memory Is Not Enough for Reliability

**Class: A/C**

Anthropic's own ecosystem makes the role separation clear:

```text
SOURCE OF TRUTH
repo / DB / docs / external systems

        ↓

MEMORY
things worth remembering that are hard to reconstruct

        ↓

INSTRUCTIONS
how Claude should behave

        ↓

SKILLS / PROCEDURES
how Claude should perform recurring work

        ↓

ENFORCEMENT
permissions / hooks / deterministic policy

        ↓

VERIFICATION
proof the real outcome is correct
```

Using memory as a substitute for source-of-truth systems or deterministic enforcement is an architectural mistake.

---

# 62. Best Overall Mental Model

The cleanest complete abstraction is:

```text
                         EXTERNAL TRUTH
              repos / email / calendar / DB / docs
                             │
                             ▼
                           TOOLS
                             │

HISTORICAL EVIDENCE ──── CURRENT CLAUDE ──── PROJECT KNOWLEDGE
 chat search / RAG             │                   RAG
                               │
                               ▼
                            RESPONSE
                               │
                               ▼
                    MEMORY MAINTENANCE
                               │
                     epistemic admission
                               │
                    durability calibration
                               │
                      entity resolution
                               │
                   provenance + correction
                               │
                   concurrency-safe write
                               │
                               ▼
                       DURABLE MEMORY
                               │
                ┌──────────────┴─────────────┐
                │                            │
           compact index               detailed files
                │                            │
                └────── JIT retrieval ──────┘
                               │
                               ▼
                         future Claude

                               +
                       periodic cleanup
                    / consolidation / Dreams
```

---

# 63. Confidence Summary

| Architectural claim | Confidence |
|---|---|
| Consumer memory uses individual categorized entries/topics | A — Confirmed |
| Memory is represented to users as short files under Topics | A — Confirmed |
| Claude updates memory during conversations | A — Confirmed |
| Users can explicitly remember/change/forget | A — Confirmed |
| Chat and cloud Cowork share memory | A — Confirmed |
| Projects have isolated memory | A — Confirmed |
| Projects have dedicated summaries | A — Confirmed |
| Historical chat retrieval uses RAG | A — Confirmed |
| Project Knowledge RAG is separate | A — Confirmed |
| Memory import/export exists | A — Confirmed |
| Sensitive-memory controls exist | A — Confirmed |
| Deleting chats does not automatically delete derived memory | A — Confirmed |
| Consumer memory is exposed internally as a filesystem abstraction | B — Strong evidence |
| `/profile.md`, `/preferences.md`, `/topics`, `/areas`, `/people` exist | B — Strong evidence |
| Profile/preferences are supplied directly while detailed memories are read selectively | B/C |
| Background post-turn memory pass exists | B — Strong evidence |
| Explicit remember/forget uses a foreground write path | B — Strong evidence |
| Provenance concepts like stated/observed/inferred exist | B — Strong evidence |
| Consumer memory uses version-safe writes | B — Strong evidence |
| Consumer memory supports aliases/cross-links | B — Strong evidence |
| Claude Code uses index + lazy retrieval | A — Confirmed |
| Claude Code memory is machine-local | A — Confirmed |
| API Memory Tool is developer-owned and filesystem-like | A — Confirmed |
| Managed Agent memory supports scopes/versioning/concurrency | A — Confirmed |
| Dreams consolidate memory into a new store | A — Confirmed |
| Exact consumer backend storage | U — Unknown |
| Exact consumer retrieval/ranking algorithm | U — Unknown |

---

# 64. What We Still Do Not Know

These remain **U — UNKNOWN**.

## Consumer retrieval

- whether embeddings are used;
- which embedding model, if any;
- whether vector search is used at all;
- lexical vs semantic routing;
- recency weighting;
- frequency weighting;
- salience weighting;
- retrieval thresholds;
- maximum number of memory files read per turn;
- token budget for detailed memory retrieval;
- whether the main Claude selects files;
- whether a separate router selects files;
- whether server-side pre-ranking occurs.

## Consumer memory writing

- exact model used for the background pass;
- whether it is the same Claude model with a different prompt;
- whether smaller classifiers participate;
- exact admission score;
- exact conflict-resolution algorithm;
- exact cadence if there are additional maintenance passes beyond the post-turn pass.

## Consumer storage

- physical database/object-store technology;
- exact record schema;
- whether model-facing files map one-to-one with backend objects;
- exact per-file size cap;
- total per-account memory capacity;
- internal deletion/retention implementation.

## Consolidation

- whether ordinary consumer Claude runs a hidden Dreams-like consolidation pass;
- whether old consumer files are automatically merged;
- whether unused memories decay automatically;
- whether retrieval/use reinforces retention.

## Cross-surface behavior

Officially confirmed:

```text
Chat ↔ cloud Cowork
```

Unknown:

- full list of surfaces that can write the shared consumer memory store;
- whether future/local surfaces share the same internal provenance model.

---

# 65. Things We Must Not Claim Without New Evidence

Do not say:

### “Claude consumer memory definitely uses a vector database.”

Unknown.

### “Every memory is inserted into every prompt.”

Evidence indicates selective loading.

### “Anthropic physically stores consumer Markdown files on disk.”

Unproven; the filesystem may be only a model-facing abstraction.

### “Claude Code and Claude Chat share the same memory store.”

Not officially documented.

### “Projects just search all past chats.”

False. Project memory/search is scoped.

### “Deleting the original chat deletes its memory.”

False in the modern consumer memory system.

### “Memory guarantees Claude will follow a preference.”

Memory is context, not deterministic enforcement.

### “Dreams currently run on ordinary consumer Claude memory.”

Unknown.

### “The captured Fable 5.1 system prompt is an official Anthropic product specification.”

False. It is strong implementation evidence only.

---

# 66. Failure Modes a Complete Claude-Like Memory Architecture Must Handle

A serious architecture must account for at least these categories.

### Capture failure
Useful information is never written.

### Over-capture
Temporary or irrelevant details become durable memory.

### Provenance failure
Model-generated or external facts are silently treated as user-established truth.

### Entity duplication
The same person/project appears under multiple memory identities.

### Staleness
Old state survives after circumstances change.

### Contradiction
Incompatible claims coexist.

### Retrieval failure
The right memory exists but is not found.

### Ranking failure
It is found but ranked below less useful material.

### Scope failure
Correct information is unavailable because of Project/privacy boundaries.

### Leakage
Information crosses a scope where it should not.

### Over-personalization
Memory influences a response where it should not.

### Under-personalization
Relevant memory is ignored.

### Correction failure
New information does not supersede old state correctly.

### Forgetting failure
Explicitly removed material survives or is recreated.

### Concurrency failure
One surface overwrites another surface's newer memory.

### Memory poisoning
Untrusted prompt injection becomes persistent state.

### Capacity/entropy failure
The store grows until retrieval quality degrades.

### Consolidation failure
Duplicates/stale material accumulate because cleanup is absent or wrong.

### Model-use failure
Correct memory reaches Claude but is interpreted badly.

---

# 67. Experimental Questions That Could Reduce the Unknowns

Document research is approaching diminishing returns in several areas. Controlled tests are the next major evidence source.

## Consumer write latency

Plant a novel durable fact and measure when it appears in Topics and becomes available across new chats.

## Foreground vs background memory

Compare explicit “remember X” with naturally stated facts and measure timing/visibility differences.

## Memory admission

Introduce equally durable facts with different repetition frequencies and compare which become saved topics.

## Provenance

Compare:

1. user directly states fact;
2. Claude infers fact;
3. web/tool reports fact;
4. user confirms tool-reported fact.

Observe which becomes durable memory.

## Entity resolution

Refer repeatedly to one person/project using aliases and inspect whether Claude creates one topic/file or duplicates.

## Correction handling

Establish A, then later establish B replacing A. Inspect whether chronology or only current state survives.

## Explicit forgetting

Establish a fact, create derived context from it, then request forgetting and test whether dependent material also disappears.

## Concurrency

If multiple Claude surfaces can be exercised in parallel, attempt near-simultaneous writes to the same memory and inspect conflict behavior.

## Sparse retrieval

Build many memory topics and ask queries targeting one topic. Measure which files/topics appear to activate.

## Project boundaries

Repeat similar facts inside and outside Projects and test leakage in both directions.

## Chat vs Cowork

Create a durable memory in Chat and test cloud Cowork; then reverse direction.

## Incognito

Verify read/write boundaries against normal memory.

## Import/export fidelity

Export memory, re-import to a clean test state, and compare semantic fidelity, omissions, and restructuring.

## Scale

Build hundreds/thousands of distinct memories and test retrieval degradation, salience, pruning, and consolidation behavior.

---

# 68. Architectural Lessons for Reproducing Claude-Like Memory

If the goal is to reproduce the **principles** rather than undocumented internals, a serious architecture should include:

```text
AUTHORITATIVE SOURCES
       ↓
DURABLE USER EVIDENCE
       ↓
MEMORY ADMISSION FILTER
       ↓
PROVENANCE / EPISTEMIC TYPE
       ↓
ENTITY RESOLUTION
       ↓
SCOPED MEMORY OBJECTS
       ↓
INDEX / ROUTING SUMMARY
       ↓
JIT DETAIL RETRIEVAL
       ↓
CURRENT CONTEXT
       ↓
MODEL
       ↓
USER CORRECTION / FORGETTING
       ↓
VERSION-SAFE UPDATE
       ↓
PERIODIC CONSOLIDATION / GC
```

Required properties:

- preserve authoritative sources outside memory;
- store only durable context that is expensive to reconstruct;
- distinguish user statements from model inference/tool output;
- use one canonical entity per subject;
- keep memory scoped by project/user/agent as appropriate;
- retrieve details only when relevant;
- support explicit correction and forgetting;
- use version-safe writes;
- make shared/reference memory read-only when possible;
- keep audit history where consequences matter;
- prune/expire low-value material;
- periodically consolidate duplicates and contradictions;
- never treat memory as deterministic policy enforcement.

---

# 69. Core Design Principle

The central insight from all available evidence is:

> **External systems hold truth. Memory preserves the durable residue that is hard to reconstruct. Historical search recovers exact episodes. Scoping controls where information may flow. Sparse retrieval controls what reaches context. The model decides how to use it. Consolidation prevents long-term memory from decaying into noise.**

This is the current highest-level architectural source of truth for Claude memory.

---

# 70. Primary Evidence Registry

## Current Claude Consumer Memory

### Anthropic — Use Claude's chat search and memory to build on previous context

Primary current source for consumer memory, project memory, chat search, pause/reset, sensitive-memory controls, deletion semantics, plan availability, and organizational controls.

https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

### Anthropic — Release Notes

Primary source for the July 10, 2026 move from a synthesized summary to individual categorized memory entries and later 2026 memory changes.

https://support.claude.com/en/articles/12138966-release-notes

### Anthropic — Claude's memory works everywhere and you decide what's in it

Primary source for short memory files under Topics and shared Chat/cloud-Cowork memory.

https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it

### Anthropic — Import and export your memory from Claude

Primary source for memory portability, import behavior, export behavior, and legacy migration status.

https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

### Anthropic — RAG for Projects

Primary source for Project Knowledge RAG and current all-plan availability.

https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects

### Anthropic — Incognito chats

Primary source for Incognito memory/history boundaries and retention.

https://support.claude.com/en/articles/12260368-use-incognito-chats

### Anthropic — Monthly Recap

Primary source for memory-dependent but separate reflective recap behavior.

https://support.claude.com/en/articles/15672559-see-your-monthly-recap

## Claude Code

### Claude Code — Memory

Primary source for `CLAUDE.md`, auto-memory types, storage, repository scope, local persistence, index loading, lazy topic retrieval, and memory-location security.

https://code.claude.com/docs/en/memory

### Claude Code — Context Window

Primary source for compaction/reinjection behavior.

https://code.claude.com/docs/en/context-window

### Claude Code — Subagents

Primary source for user/project/local subagent memory scopes.

https://code.claude.com/docs/en/sub-agents

## Claude API

### Anthropic — Memory Tool

Primary source for `/memories`, client-owned persistent storage, filesystem abstraction, JIT retrieval, compaction integration, and implementation guidance.

https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

### Anthropic — Context engineering tools

Additional source for just-in-time context/memory retrieval patterns.

https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools

## Managed Agents

### Anthropic — Managed Agent Memory

Primary source for Memory Stores, mounting, limits, read/write modes, security warnings, optimistic concurrency, and versioning.

https://platform.claude.com/docs/en/managed-agents/memory

### Anthropic — Memory Versions

Primary source for Managed Agent memory version/audit records.

https://platform.claude.com/docs/en/api/http/beta/memory_stores/memory_versions

### Anthropic — Dreams

Primary source for offline memory consolidation from existing memory plus 1–100 prior sessions.

https://platform.claude.com/docs/en/managed-agents/dreams

## Strong Implementation Evidence

### Captured Claude Fable 5.1 system prompt

Evidence for model-facing `memory_filesystem`, file operations, apparent `/profile.md` / `/preferences.md` / `/topics` / `/areas` / `/people` structure, background memory pass, versioned writes, provenance concepts, aliases, and size management.

https://github.com/elder-plinius/CL4R1T4S/blob/main/ANTHROPIC/Claude-Fable-5.1.md

**This source is not an Anthropic-authenticated product specification. Claims depending solely on it remain Class B.**

---

# 71. Change Log

## Version 1.0 — September 8, 2026

Initial canonical Claude memory source of truth.

Includes:

- consumer memory migration from legacy summary to individual topics/files;
- shared Chat/cloud-Cowork memory;
- Project memory isolation and project summaries;
- Project Knowledge RAG;
- historical chat search;
- memory pause/reset semantics;
- deletion separation between chats and memories;
- memory import/export and migration window;
- sensitive-memory and Incognito controls;
- Team/Enterprise governance;
- Monthly Recap distinction;
- Fable 5.1 implementation evidence;
- apparent consumer memory filesystem and file taxonomy;
- background memory-write pass;
- foreground explicit remember/forget path;
- provenance/epistemic typing evidence;
- entity aliases/cross-links;
- optimistic concurrency evidence;
- file-size/consolidation behavior;
- correction and forgetting semantics;
- memory-poisoning/security boundary;
- Claude Code auto-memory architecture;
- `CLAUDE.md` vs memory vs enforcement;
- compaction/reinjection behavior;
- subagent memory scopes;
- API Memory Tool;
- JIT retrieval;
- Managed Agent Memory Stores;
- memory limits and access modes;
- version history and concurrency control;
- Dreams memory consolidation;
- failure taxonomy;
- experimental research agenda;
- architectural replication guidance;
- explicit hard unknowns.

---

# 72. Current Final Conclusion

As of September 8, 2026:

**The strongest evidence indicates that Claude memory is an external, persistent, selectively retrieved context architecture rather than a property stored inside the model itself.**

Consumer Claude now uses individual categorized memory files/topics, with Project-scoped memory and separate historical chat search. Chat and cloud Cowork share consumer memory. Claude Code independently implements a transparent local **index + topic files + lazy retrieval** architecture. The Claude API exposes a developer-owned filesystem-like memory primitive. Managed Agents extend the model with scoped persistent stores, read/write controls, version history, optimistic concurrency, and offline memory consolidation through Dreams.

The strongest cross-product principle is:

> **Store the irrecoverable residue; re-query authoritative reality.**

The best architectural abstraction is:

> **Source systems hold truth. Memory stores durable user/project context that is hard to reconstruct. Historical retrieval recovers exact episodes. Scope controls what can flow where. Sparse retrieval controls what reaches the model. Explicit corrections update state. Versioning prevents clobbering. Security boundaries prevent poisoned memory. Consolidation prevents long-lived memory from decaying into duplicates, contradictions, and stale assumptions.**

That is the current canonical source of truth.