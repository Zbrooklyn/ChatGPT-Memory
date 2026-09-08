# Claude Memory — Master Source of Truth

**Version:** 1.1 — Exhaustive Evidence Ledger  
**Last verified:** September 8, 2026  
**Supersedes:** Version 1.0  
**Scope:** Claude consumer Chat memory, Projects, cloud Cowork, past-chat search, memory import/export, Incognito, sensitive-memory controls, organization governance, Monthly Recap, Claude Code `CLAUDE.md`/rules/auto-memory/compaction/subagents, Claude API Memory Tool, Managed Agent Memory Stores, Dreams, current implementation evidence from captured Claude prompts, architectural inferences, failure modes, experiments, and hard unknowns.  
**Purpose:** Maintain one canonical, evidence-graded and deliberately exhaustive account of everything currently established about Claude memory within the evidence corpus listed here.

> **Completeness boundary:** “Exhaustive” means exhaustive with respect to current public Anthropic documentation plus the specifically identified implementation evidence reviewed as of the verification date. It does not mean undocumented production internals are known. Those remain explicitly listed as unknown.

---

# 1. Bottom Line

The strongest evidence supports this conclusion:

**Claude does not have one single memory system. It has a family of external persistence, retrieval, instruction, knowledge, and consolidation mechanisms whose behavior differs by product surface.**

Consumer Claude now uses individual categorized memory topics/files plus a separate historical chat-search mechanism. Projects create isolated memory/search domains and dedicated project summaries while also providing Project Knowledge and RAG. Claude Chat and cloud Cowork share consumer memory; local Cowork does not. Claude Code independently uses human-authored instruction files plus local Claude-authored auto-memory with an index-and-topic-file design. The Claude API exposes a developer-owned filesystem-like Memory Tool. Managed Agents add durable Memory Stores with read/write access modes, version history, optimistic concurrency, sandbox mounts, self-hosted synchronization, and a research-preview consolidation process called Dreams.

The most important cross-product principle is:

> **The model is not the durable memory. Durable state lives outside the model and relevant state is selectively supplied back into model context.**

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
                    admission / durability
                               │
                    subject/entity routing
                               │
                    provenance / correction
                               │
                    safe persistent update
                               │
                               ▼
                         DURABLE MEMORY
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
            routing/index              detailed state
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

The full diagram is a reconstruction. Its major pieces are independently supported by current Anthropic documentation or clearly marked implementation evidence.

---

# 2. Evidence Standard

Every material claim in this document belongs to one of four classes.

## A — Confirmed Anthropic Product Fact

Explicitly documented in current Anthropic product, support, Claude Code, Claude Platform/API, or Managed Agent documentation.

## B — Strong Implementation Evidence

Observed in current captured Claude system instructions, exports, or client behavior that closely aligns with official functionality but is **not** an Anthropic-authenticated product contract.

## C — Architectural Inference

A conclusion strongly suggested by multiple A/B facts but not directly stated by Anthropic.

## U — Unknown

No sufficient current evidence. Unknowns must not be silently filled with plausible assumptions.

---

# 3. Source Precedence

When evidence conflicts, use this order:

1. Newer, feature-specific current Anthropic documentation.
2. Current Anthropic general documentation.
3. Current Claude Code / Claude Platform / Managed Agent documentation.
4. Older Anthropic documentation retained for historical lineage.
5. Reproducible implementation observations.
6. Captured system prompts and client/export observations.
7. Community speculation.

Freshness matters. A newer Anthropic page overrides stale wording in this document or in older Anthropic pages.

---

# 4. Terminology: “Memory” Is Not One Thing

**Class: A/C**

Within the Claude ecosystem, at least these distinct mechanisms exist:

| Mechanism | What it does | Persistent? | Scope |
|---|---|---:|---|
| Consumer Claude Memory | Durable user/project context as individual topics/files | Yes | non-project or per Project |
| Past Chat Search | Retrieves specific historical conversations with RAG | History-dependent | non-project or per Project |
| Project Summary | Dedicated compressed project context | Yes/current-state | one Project |
| Project Knowledge | Uploaded/project knowledge | Yes | one Project |
| Project Knowledge RAG | Retrieves relevant project knowledge when needed | Derived retrieval | one Project |
| User preferences/styles | Personalization instructions, distinct from memory | Yes | account/product scope |
| Claude Code `CLAUDE.md` | Human-authored persistent guidance | Yes | managed/user/project/local/path scope |
| Claude Code auto-memory | Claude-authored learned context | Yes, local | repo/project scope |
| Claude Code subagent memory | Agent-specific learned context | Yes | user/project/local agent scope |
| API Memory Tool | Developer-owned persistent memory abstraction | Developer-defined | application-defined |
| Managed Agent Memory Stores | Durable text-document stores mounted to agents | Yes | workspace/store/session attachment |
| Dreams | Offline consolidation into a new Memory Store | Produces durable output | Managed Agents |
| Context compaction | Compresses active conversational context | Workflow continuity | current run/session |
| Monthly Recap | Reflective analysis of recent usage/history | Derived output | consumer account |

The terms should not be collapsed into one generic “memory store.”

---

# 5. Consumer Memory Product Evolution

## 5.1 Legacy consumer memory

**Class: A — historical/current for a small migration remainder**

The legacy architecture synthesized a memory summary periodically from conversation history. For remaining legacy Team/Enterprise organizations, the synthesis is described as refreshing approximately every 24 hours. Project memory remains separate from standalone/non-project memory.

## 5.2 July 10, 2026 architecture change

**Class: A**

On **July 10, 2026**, Anthropic replaced the normal consumer daily-summary architecture with **individual categorized memory entries/topics that Claude can read and update during conversations**.

```text
LEGACY
many chats
  ↓
periodic global synthesis
  ↓
memory summary

CURRENT
conversation
  ↓
individual durable memory topics/files
  ↓
read/update during later conversations
```

## 5.3 August 25, 2026 expansion

**Class: A**

Anthropic expanded the system so Chat and **cloud Cowork** share memory, Topics are directly editable, and sensitive-topic controls are exposed.

## 5.4 Remaining legacy migration

**Class: A**

A small number of Team/Enterprise organizations may still temporarily use legacy memory. Anthropic documents a legacy-memory export window ending **September 9, 2026** for migrated users who need to recover content that did not carry forward as expected.

Primary sources:

- https://support.claude.com/en/articles/12138966-release-notes
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

---

# 6. Current Consumer Memory Is File/Topic-Oriented

**Class: A**

Anthropic describes everything Claude remembers as a **list of short files under Topics** in Memory settings. Users can inspect, edit, or delete individual topics instead of manipulating one opaque global summary.

```text
Memory
 ├── topic/file
 ├── topic/file
 ├── topic/file
 └── ...
```

This officially establishes the product-facing file/topic model.

It does **not** establish that Anthropic physically stores Markdown files on disk. A file abstraction may map to database/object-store records.

**Physical storage backend: U**

Primary sources:

- https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 7. Consumer Memory Is Written While You Chat

**Class: A**

Claude adds useful memory during ordinary conversations rather than waiting for a daily synthesis pass. An updated deadline or durable project detail can therefore become available to future conversations as the user chats.

Users can also explicitly request operations such as:

```text
Remember this.
Remember that X.
Update what you remember about X.
Forget X.
```

Changes to Topics apply to future conversations.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 8. What Consumer Claude Is Documented to Remember

**Class: A**

Anthropic lists durable collaboration-oriented information including:

- professional role and professional context;
- projects and ongoing work;
- important people;
- important places;
- communication preferences;
- working style;
- technical preferences;
- coding preferences;
- project details;
- ongoing decisions and constraints useful in future work.

Memory admission is selective:

```text
mentioned
   ≠
automatically remembered
```

The exact production admission algorithm is unknown.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 9. Current Consumer Memory Availability

**Class: A**

Current new memory is documented on web, Desktop, and supported mobile apps.

At the account/product level:

- **Free:** memory enabled by default.
- **Pro:** enabled by default.
- **Max:** enabled by default.
- **Team:** organizationally off until an owner enables it.
- **Enterprise:** organizationally off until an owner enables it.

The precise UI may vary by platform/app version.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 10. Chat and Cloud Cowork Share Consumer Memory

**Class: A**

Claude Chat and **cloud Cowork** use the same memory in both directions.

```text
              shared consumer memory
             ↗                      ↖
        Claude Chat              cloud Cowork
```

A memory learned in Chat may help cloud Cowork, and context learned through cloud Cowork may later help Chat.

**Local Cowork does not participate in this cloud shared-memory behavior.**

There is no current official evidence that Claude Code auto-memory is synchronized with the same store.

```text
Chat ↔ cloud Cowork
        shared

Claude Code auto-memory
        separate local system
```

Primary sources:

- https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 11. Projects Are Separate Memory Namespaces

**Class: A**

Each Claude Project has:

1. its own **memory space**; and
2. a **dedicated project summary**.

Project memory is isolated from ordinary non-project memory and from other Projects.

```text
Claude account
│
├── non-project memory
│
├── Project A
│   ├── isolated project memory
│   └── dedicated project summary
│
└── Project B
    ├── isolated project memory
    └── dedicated project summary
```

Moving a conversation into or out of a Project changes the memory/search domain to which it belongs.

Primary sources:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects

---

# 12. Project Memory, Summary, Instructions, Knowledge, and RAG Are Distinct

**Class: A**

A Project can simultaneously expose:

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
past-chat search inside the project
+
project uploaded knowledge/files
+
project-knowledge RAG
```

These are separate context mechanisms.

Project Knowledge can be provided directly while small enough. As it approaches/exceeds context limits, Claude can use RAG to retrieve relevant material rather than loading all project knowledge every time.

Anthropic says this can expand practical project knowledge capacity by up to approximately **10×**.

Current Project RAG documentation lists availability on:

- Free;
- Pro;
- Max;
- Team;
- Enterprise.

The RAG behavior is automatic; users do not have to manually build an index.

Primary source:

- https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects

---

# 13. Past Chat Search Is Separate From Saved Memory

**Class: A**

Claude has a separate historical retrieval capability for prior conversations. Anthropic explicitly describes past-chat search as **RAG** and exposes it as a tool call when used.

Search boundaries follow memory/project boundaries:

```text
non-project conversation
   ↓
search non-project chat history

Project A conversation
   ↓
search Project A chat history only
```

Therefore Claude can appear to remember through two very different paths:

```text
SAVED MEMORY
“I know this durable thing about the user/project.”

HISTORICAL RETRIEVAL
“I found the old conversation where this occurred.”
```

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 14. Past Chat Search Availability and Controls

**Class: A**

Past-chat search is currently documented for:

- Pro;
- Max;
- Team;
- Enterprise.

It is available on web, Desktop, and Mobile.

Once rolled out, Anthropic documents it as enabled by default, with a separate **Search and reference chats** control. It is independently configurable from generated memory.

Incognito conversations are excluded.

Enterprise organizations using customer-managed encryption keys currently cannot use past-chat search because conversation contents are encrypted in a way that prevents this search feature.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 15. Pause Memory = Stop Reading and Stop Writing

**Class: A**

Pausing memory:

- keeps existing memories stored;
- stops Claude from using them;
- stops Claude from generating new memories;
- keeps sensitive memories stored but inactive;
- does **not** retroactively learn from conversations held while memory was paused.

```text
Pause Memory
   =
stop READ
+
stop WRITE
```

Unpausing reactivates ordinary memory behavior and any retained eligible memory.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 16. Reset Memory Is Global and Irreversible

**Class: A**

Reset Memory deletes all generated memory, including project memory, and Anthropic describes the action as irreversible.

Re-enabling after reset starts with a new memory state.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 17. Chat Deletion and Memory Deletion Are Separate

**Class: A**

In the **current** memory architecture, deleting/expiring the source conversation does **not automatically delete an already-created memory entry**.

```text
conversation
   ↓
memory entry produced
   ↓
conversation later deleted
   ↓
memory entry may remain
```

To remove the generated memory, the relevant Topic must also be removed/edited.

This establishes that conversation storage and generated memory are distinct persisted objects.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 18. Current Memory Data Lifecycle and Exports

**Class: A**

Anthropic documents these additional data-handling facts:

- current generated memory follows applicable chat/account retention policies;
- memory can reflect changes to chats over time;
- deleting/expiring a source chat does not itself delete the derived current-memory topic;
- memory data is included in account data exports;
- Team/Enterprise organizational retention rules apply as documented;
- Enterprise memory entries are encrypted at rest.

For organization accounts, memory and Incognito/export behavior are governed by the organization's data policies in addition to user controls.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 19. Memory Topics Are Directly Editable

**Class: A**

Under Settings → Memory → Topics, users can inspect individual memory topics/files, edit them, or delete them. Changes affect future conversations.

Users can also ask Claude in chat to remember, change, or forget relevant information.

Past-chat search results can surface links/citations to original conversations, which are separate from generated Topics.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 20. Memory Import Is a First-Class Product Feature

**Class: A**

Claude can import memory material from another AI provider.

The documented flow allows a user to paste exported memory/profile information and have Claude extract useful material into individual memory entries.

Current documented import availability:

- Free;
- Pro;
- Max;
- Team;
- web;
- Claude Desktop.

Anthropic describes import as **experimental**. It may not retain every item.

The importer is specifically work-oriented; Claude may discard imported personal material that is unrelated to its collaboration/work memory focus.

Primary source:

- https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

---

# 21. Memory Export and Portability

**Class: A**

Claude supports exporting memory. Anthropic explicitly describes the ability to view/export memory in the form Claude sees it, including asking Claude to write out its memories verbatim.

Architectural implication:

> Consumer AI memory is becoming portable user state rather than an entirely opaque vendor-owned profile.

**Class for implication: C**

Primary source:

- https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

---

# 22. Legacy Memory Export Window

**Class: A**

As of September 8, 2026, migrated users can still export legacy memory through **September 9, 2026** if they believe information failed to migrate correctly.

This is a migration detail, not the design of the modern memory system.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 23. Sensitive Topics Are Excluded by Default

**Class: A**

By default Claude avoids automatically saving sensitive categories including areas such as:

- health;
- race;
- ethnicity;
- religious beliefs;
- political beliefs;
- gender identity;
- similar sensitive attributes.

Users can enable **Include sensitive topics in memory** where available.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 24. Sensitive-Memory Opt-In Semantics

**Class: A**

When sensitive memory is enabled:

- the change is prospective, not retroactive;
- Claude can save eligible sensitive material from future conversations;
- the user receives a review notice when a sensitive memory is saved;
- the first decline related to sensitive-memory settings can trigger a one-time explanatory notice;
- current mobile sensitive-save notices require a sufficiently current app version; older unsupported app versions do not silently save the sensitive item;
- disabling sensitive memory removes sensitive items stored under that mechanism.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 25. Categories Anthropic Says Consumer Memory Will Not Store

**Class: A**

Anthropic publicly identifies categories Claude will not save even if asked, including:

- government identification numbers;
- financial account numbers;
- criminal history;
- immigration status.

Anthropic also excludes content that violates applicable policies from memory behavior as described in its product materials.

Primary sources:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it

---

# 26. Incognito Is a Hard Memory/History Boundary

**Class: A**

Incognito chats:

- do not read normal Claude memory;
- do not create normal Claude memory;
- are not saved in ordinary chat history;
- are excluded from past-chat search;
- are excluded from Monthly Recap;
- are not used for model training under Anthropic's documented Incognito behavior.

However, Incognito may still receive other personalization such as profile/custom styles/preferences.

Therefore:

```text
MEMORY
   ≠
ALL PERSONALIZATION
```

Primary source:

- https://support.claude.com/en/articles/12260368-use-incognito-chats

---

# 27. Incognito Retention and Organizational Visibility

**Class: A**

Incognito is not zero-retention.

- Default retention is approximately **30 days** for safety purposes.
- Enterprise/custom organizational policies may require longer retention.
- Team/Enterprise Incognito conversations can appear in organization data exports as documented.
- Enterprise Compliance API/retention controls may apply.
- Incognito currently operates outside Projects and is not simply convertible into an ordinary saved conversation after closure.

Primary source:

- https://support.claude.com/en/articles/12260368-use-incognito-chats

---

# 28. Team and Enterprise Governance

**Class: A**

For Team/Enterprise:

- organization-level generated memory is off by default in the new experience;
- an Owner/Primary Owner enables memory availability;
- sensitive-memory permission is a separate organization-level control;
- individual users manage their own generated memories after the organization enables the feature;
- organization owners cannot inspect/edit an individual's memory through the memory UI;
- disabling organization memory immediately and permanently deletes generated memory entries for users in that organization;
- some organizations using HIPAA configurations, public-sector arrangements, or custom data-retention agreements do not currently have this memory feature.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 29. Team/Enterprise Security, Export, and Audit Details

**Class: A**

Additional documented enterprise behavior includes:

- memory entries encrypted at rest;
- ordinary organization retention/export rules apply;
- Incognito may be included in organization exports despite being hidden from the user's ordinary history;
- organization-level memory setting changes are available in audit logging;
- normal conversation access logging applies;
- individual member edits to personal memory topics are not documented as separate organization audit-log events.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 30. Legacy Team/Enterprise Behavior Must Not Be Confused With Current Memory

**Class: A — historical/migration behavior**

For the small remaining legacy cohort, Anthropic documents differences such as:

- memory synthesis approximately every 24 hours;
- standalone/non-project synthesis and separate project memory;
- deleting conversations changes the material available to the next synthesis;
- direct edits to the legacy memory summary can apply immediately rather than waiting for the next daily cycle;
- organization control defaults differ from the new memory experience.

These are retained for historical completeness only. They must not be generalized to the current individual-topic architecture.

Primary source:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

---

# 31. Monthly Recap Is Memory-Adjacent, Not the Memory Store

**Class: A**

Claude provides **Settings → Reflect → Monthly Recap**, a reflective analysis of recent Claude usage.

It requires generated memory to be enabled but is a separate derived feature.

Current documented availability includes Free, Pro, and Max on web/Desktop for viewing; supported mobile activity can still contribute to recap statistics even when the recap page itself is not available on mobile. Team/Enterprise are not currently part of the consumer recap experience.

Primary source:

- https://support.claude.com/en/articles/15672559-see-your-monthly-recap

---

# 32. Monthly Recap Inputs and Exclusions

**Class: A**

Recap can surface:

- opening summary;
- total conversation counts;
- most active day;
- peak hour;
- daily activity chart;
- topic distribution;
- AI-fluency observations/skills such as Delegation, Description, Discernment, and Diligence.

Recap excludes:

- Incognito;
- Health integration chats;
- Cowork;
- Claude Code.

Raw connected Gmail/Google Drive content is not directly included in the recap dataset; Claude-authored summaries or comments that appeared in conversations can be reflected.

Sensitive/distress-related topics are not used as recap-leading categories/count breakdowns as documented.

Recap is generated when the user visits/refreshes Reflect rather than acting as a constantly visible memory object.

Primary source:

- https://support.claude.com/en/articles/15672559-see-your-monthly-recap

---

# 33. Consumer Memory and Historical Search Form a Semantic/Episodic Pair

**Class: C — terminology**

Anthropic does not require these cognitive-science labels, but the architecture is usefully described as:

### Semantic/adaptive memory

Compressed durable context such as role, preferences, projects, recurring people, and decisions.

### Episodic retrieval

Search for the original prior conversation/episode when exact historical evidence is needed.

This explains why Claude can sometimes recover highly specific old details without those details appearing as durable Topics.

---

# 34. Implementation Evidence Boundary

Everything from this section through the captured-consumer internals section is **Class B unless explicitly stated otherwise**.

Primary implementation source:

- `elder-plinius/CL4R1T4S`, captured `ANTHROPIC/Claude-Fable-5.1.md`, commit `93b0ae6fb503db6642e58f9d6352db973a900cdc`.

A highly similar memory block also appears in that repository's `ANTHROPIC/OPUS-5.md`. Because both captures come through the same external repository/capture pipeline, this is **same-source corroboration**, not an independent Anthropic confirmation.

The captures are valuable evidence but are not Anthropic-authenticated product specifications. They can change without notice.

---

# 35. Captured Consumer `memory_filesystem`

**Class: B**

The captured Fable 5.1 instructions describe a persistent model-facing component approximately named:

```text
<memory_filesystem>
```

with operations corresponding to:

```text
memory_read
memory_write
memory_str_replace
memory_append
memory_list
memory_delete
```

This strongly supports the conclusion that current consumer Claude is exposed to a **path-addressed memory abstraction**.

It does not prove the physical backend is a filesystem.

---

# 36. Captured Consumer File Taxonomy

**Class: B**

The capture describes an organization resembling:

```text
/profile.md
/preferences.md
/topics/...
/areas/...
/people/...
```

Approximate semantics:

- `/profile.md`: stable identity/context;
- `/preferences.md`: how Claude should interact/respond;
- `/topics/`: recurring interests, routines, habits, broad recurring domains;
- `/areas/`: ongoing projects, responsibilities, decisions, work domains;
- `/people/`: persistent relationship/context for recurring people.

Exact paths are implementation evidence, not public contract.

---

# 37. Captured `/profile.md` Admission Rule

**Class: B**

The Fable/Opus capture gives a concrete profile-stability test roughly equivalent to:

> Would this still be true in three months?

It separates durable identity from temporary state. A role/team relationship can belong in profile; something tied to “this sprint,” a deadline, or “currently” normally belongs in `/areas/` or `/topics/` instead.

The capture also instructs Claude to keep `/profile.md` **under 300 words**.

This 300-word limit is observed implementation evidence, not an Anthropic public product guarantee.

---

# 38. Captured Sparse Retrieval / `memory_listing`

**Class: B**

The capture describes a `<memory_listing>` block that exposes the current memory directory at a routing level.

The listing includes information such as:

- file path;
- one-line description;
- aliases where applicable;
- source metadata.

The description is explicitly a **routing hint**, not a substitute for reading the detailed memory. When the listing suggests a relevant file, Claude is instructed to open/read it before concluding that it does not know the information.

The apparent architecture is:

```text
always/cheaply available
  profile
  preferences
  compact memory listing

on demand
  detailed topic/area/person memory files
```

This is highly consistent with Claude Code's officially documented index + lazy retrieval design.

---

# 39. Captured Memory Frontmatter / Metadata

**Class: B**

The capture shows memory documents using metadata concepts resembling:

```yaml
name: <canonical-slug>
description: <one-line routing description>
sources:
  - chat
aliases:
  - <alternate subject name>
```

Observed semantics include:

- `name` corresponds to a canonical subject identity/path stem;
- `name` is expected to be unique across the memory set;
- `description` is what the compact memory listing uses to decide whether to open the file;
- `sources` records surfaces that have contributed to the memory;
- `aliases` are particularly associated with `/areas/` and `/people/` to resolve alternate names.

Exact schema is B-level evidence.

---

# 40. Captured Cross-Memory Links

**Class: B/C**

The capture supports `[[name]]`-style references between memory subjects. Canonical unique names provide link targets.

This makes the semantic structure resemble a lightweight graph:

```text
MEMORY SUBJECT
 ├── canonical path/name
 ├── aliases
 ├── description
 ├── facts
 ├── provenance/source surfaces
 └── links to related memory subjects
```

“Markdown-shaped lightweight knowledge graph” is our architectural terminology, not Anthropic's.

---

# 41. Captured Entity Resolution

**Class: B**

Aliases appear specifically intended to prevent fragmentation such as:

```text
David
Dave
David from Crystal Tile
```

becoming three conflicting memories when they refer to one subject.

The captured rules emphasize routing a fact by the **fact's semantic domain**, not by whichever file happened to be open or already existed.

Architectural lesson:

> Long-term memory requires canonical entity resolution or the memory graph slowly fragments into duplicates.

**Class for lesson: C**

---

# 42. Captured Background Memory Pass

**Class: B**

The Fable 5.1 capture explicitly describes durable filing as occurring **automatically after each completed assistant turn** through a background memory pass that re-reads the finished exchange.

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
   ├── reread exchange
   ├── determine durable facts
   ├── choose semantic file
   ├── read current memory state
   ├── reconcile/correct
   └── persist update
```

Anthropic has not publicly identified the exact model, prompt, or service performing this pass.

**Background writer model: U**

---

# 43. Explicit Remember/Update/Forget Uses an Apparent Foreground Path

**Class: B**

When a user explicitly requests a memory mutation, the captured instructions tell foreground Claude to perform it directly rather than waiting for ordinary post-turn filing.

The background mechanism is then intended not to reprocess the same exchange in a way that duplicates or reverses the explicit request.

This prevents the obvious failure:

```text
User: Forget X.
foreground: delete X
background: sees X in transcript and recreates X
```

---

# 44. Captured Memory Is Best-Effort, Not Load-Bearing

**Class: B/C**

The consumer prompt treats automatic memory maintenance as best-effort. A memory I/O failure should not derail the user's primary task.

```text
AUTHORITATIVE STATE
repo / database / calendar / email / documents

ADAPTIVE MEMORY
helpful durable context
not the canonical operational state
```

This distinction is central to reliable agent design.

---

# 45. Captured Epistemic/Provenance Typing

**Class: B**

The capture contains provenance concepts resembling:

```text
[stated]
[observed]
[inferred]
```

The Chat-side captured rules are especially conservative about adding user memory: direct user-established facts are eligible; Claude's own guesses/recommendations are not silently converted into user truth.

Other surfaces may apparently contribute observed/inferred material, but the exact cross-surface semantics are not an official contract.

---

# 46. Claude-Generated Advice Is Not Automatically User Memory

**Class: B**

Under the captured rules:

```text
Claude: Stripe looks like the best option.
```

is not itself a user memory.

If the user later establishes:

```text
Yes, we're going with Stripe.
```

that confirmation may become durable because it is now user-established.

Likewise, web results, connector results, and hearsay are not meant to become autobiographical facts merely because Claude encountered them.

Architectural implication:

> Do not silently convert model outputs or re-queryable external facts into durable user beliefs.

**Class for implication: C**

---

# 47. Captured Admission Uses Durability and Repetition

**Class: B**

The prompt indicates that stable facts can be admitted quickly, while fleeting execution state should remain outside long-term memory.

A casual one-off taste/hobby does not necessarily deserve storage on first mention; repetition or meaningful engagement can make it more durable/relevant.

This is more nuanced than “every statement becomes memory.”

The exact scoring/admission model is unknown.

---

# 48. “Remember the Pointer, Not the Stale Copy”

**Class: A/C**

Claude Code independently confirms a powerful Anthropic-wide pattern: its `reference` auto-memory type records **where authoritative information can be found** rather than copying dynamic values that can be re-queried.

```text
CAN THIS FACT BE RELIABLY RECONSTRUCTED?

YES
→ prefer authoritative source / pointer

NO / expensive / user-specific
→ candidate for durable memory
```

This principle also matches the captured consumer instruction not to turn external search/tool results into durable autobiographical truth.

Primary official source:

- https://code.claude.com/docs/en/memory

---

# 49. Captured Cross-Surface Provenance

**Class: B**

The consumer capture includes `sources` metadata. When Chat updates a memory created by another participating surface, the rules indicate preserving existing source metadata and adding `chat` rather than destroying provenance.

This aligns with the confirmed Chat ↔ cloud Cowork shared-memory product behavior.

Exact metadata = B.  
Shared Chat/cloud-Cowork behavior = A.

---

# 50. Captured Optimistic Concurrency

**Class: B**

The captured memory-write tools use an `if_version`-like precondition.

Observed intended protocol:

```text
read memory → receive current version
       ↓
perform mutation with if_version
       ↓
if another writer changed it:
       conflict
       ↓
re-read latest state
merge/reconcile
retry
```

The capture also specifically instructs reading a file before delete/update so the current version is available.

This is strongly corroborated by official Managed Agent Memory, which implements comparable optimistic concurrency with `content_sha256`.

---

# 51. Captured Fine-Grained Deletion Semantics

**Class: B**

The capture distinguishes:

- deleting a whole subject/file;
- deleting/replacing a specific line/fact;
- normal correction;
- explicit forgetting.

For a whole subject, the captured flow reads the file for version then uses delete. For a line, it can replace the exact line with an empty/new value.

If a second fact existed **only because** of the forgotten fact, captured instructions say that dependent fact should be removed too.

This approximates provenance-aware cascading deletion even though no formal dependency graph is publicly documented.

---

# 52. Correction and Forgetting Are Semantically Different

**Class: B**

A correction can preserve useful chronology:

```text
Works on Infrastructure; previously Search.
```

An explicit privacy-style request such as “forget that I ever worked on Search” should remove the old fact rather than preserve it as historical detail.

This is an important distinction between:

```text
supersession / chronology
```

and:

```text
true forgetting / erasure semantics
```

---

# 53. Captured Memory File Capacity Management

**Class: B**

The captured consumer tool provides capacity/size information for memory files. When a file becomes crowded, Claude is instructed to:

- consolidate overlapping facts;
- remove stale low-value detail;
- split overly broad topics;
- summarize repetitive history;
- preserve references to external canonical systems rather than copying dynamic state.

**Exact consumer per-file limit: U**

---

# 54. Captured Sensitive-Data Rules Are Broader Than the Public Contract

**Class: B**

The Fable/Opus capture contains a broader internal exclusion/protection scheme than the public consumer-memory help page. It includes or discusses categories such as:

- race/color/ethnicity/caste;
- religion;
- sexual orientation;
- gender identity;
- immigration status;
- disability/serious illness;
- union membership;
- socioeconomic/financial details;
- medical conditions, diagnoses, labs/genetics, mental-health/therapy/addiction information;
- criminal/victimization history;
- sexual-history information;
- restrictions on inferring health information.

The captured rules contain nuanced exceptions and distinctions. Because this comes from an unauthenticated prompt capture, it must **not** be treated as a stable public product guarantee or used to override the official help center.

---

# 55. Consumer Memory Is Treated as Untrusted Context

**Class: A/B/C**

The captured consumer prompt warns against allowing stored memory to override higher-priority behavior or the user's current explicit request.

Official Managed Agent documentation independently warns that prompt injection can poison writable memory and persist into future sessions.

Therefore:

> **Persistent memory is a security boundary.**

Stored state should be treated as potentially stale, mistaken, or malicious—not equivalent to system policy.

---

# 56. Claude Code Has a Separate Memory Architecture

**Class: A**

Claude Code treats each new session as fresh context and supports two major persistence mechanisms:

```text
CLAUDE.md / rules
human-authored instructions

Auto Memory
Claude-authored learned context
```

Both are context for the model, not deterministic enforcement.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 57. `CLAUDE.md` Instruction Scopes

**Class: A**

Current Claude Code supports persistent instruction files at multiple scopes, including:

### Managed policy

Examples include OS-level managed `CLAUDE.md` locations such as:

- macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`
- Linux/WSL: `/etc/claude-code/CLAUDE.md`
- Windows: `C:\Program Files\ClaudeCode\CLAUDE.md`

### User

`~/.claude/CLAUDE.md`

### Project

`./CLAUDE.md` or `./.claude/CLAUDE.md`

### Local project

`./CLAUDE.local.md`

Nested/path-specific instructions can also be loaded when relevant files/subtrees are accessed.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 58. `CLAUDE.md` Loading and Precedence

**Class: A**

Claude Code loads applicable ancestor instruction files from the project hierarchy. Relevant files are concatenated rather than behaving as a simple “last file wins” configuration override.

Nested instruction files/path-scoped rules can be discovered/reloaded on demand when Claude accesses matching files.

A local instruction file in a directory is applied after the ordinary project instruction file at the same level.

Claude strips ordinary HTML comments from instruction text before injection except where comments occur inside code blocks, according to current docs.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 59. `CLAUDE.md` Size Guidance and Verification

**Class: A**

Anthropic recommends keeping `CLAUDE.md` concise, with approximately **under 200 lines** as a practical target.

Claude Code skips a `CLAUDE.md` file larger than approximately **4 MiB**.

`/context` can be used to verify which instruction/memory sources are actually loaded into the current context.

`/init` can create an initial project instruction file and can suggest improvements if one already exists; newer Claude Code versions also have evolving initialization/import flows.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 60. `CLAUDE.md` Imports

**Class: A**

Claude Code supports `@path` imports in instruction files.

Current documented behavior includes:

- relative and absolute imports;
- relative paths resolved from the containing instruction file;
- recursive imports with a bounded depth (currently up to four hops);
- import syntax ignored when it appears inside code spans/fenced code;
- external-project imports can require trust/approval;
- user-scope imports are trusted according to product/security rules;
- `CLAUDE.local.md` can serve as a worktree-local/private instruction file when excluded from version control.

Current Claude Code also documents migration/import support for other agent-rule files; exact version requirements can change.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 61. Claude Code Does Not Natively Treat `AGENTS.md` as `CLAUDE.md`

**Class: A**

Current docs state Claude Code reads `CLAUDE.md`, not `AGENTS.md`, as its native persistent instruction filename.

Users can explicitly import/symlink/copy other rule files where appropriate, and newer `/init`/import workflows can help migrate existing agent instructions.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 62. `.claude/rules/` Provides Modular and Path-Scoped Instructions

**Class: A**

Claude Code supports modular rule files under `.claude/rules/` and corresponding user-level rules.

Rules can be:

- unscoped and loaded at project startup;
- path-scoped through frontmatter/glob patterns and loaded when relevant files are touched;
- organized recursively.

User rules and project rules can coexist. The exact pattern-expansion and total file-size safety limits are implementation details documented in the current Claude Code memory page and may evolve.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 63. Extra Directories Do Not Automatically Import Their Instructions

**Class: A**

Adding a directory with `--add-dir` does not automatically mean Claude should load that directory's `CLAUDE.md`/rules. Current Claude Code provides an explicit environment setting to opt into additional-directory instruction loading.

This prevents “filesystem access” from being silently equated with “instruction authority.”

Primary source:

- https://code.claude.com/docs/en/memory

---

# 64. Managed Instructions and Exclusion Controls

**Class: A**

Claude Code supports managed organization policy/instruction mechanisms in addition to ordinary user/project files.

Current settings can exclude selected `CLAUDE.md` paths/globs from normal loading, while managed policy cannot simply be removed by a lower-trust project file.

This reinforces a hierarchy between organization policy and repository-authored context.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 65. `CLAUDE.md` Is Context, Not Enforcement

**Class: A/C**

Anthropic explicitly distinguishes behavioral guidance from deterministic controls.

```text
Claude should KNOW X
→ auto-memory / reference context

Claude should generally DO X
→ CLAUDE.md / rules / skills

Claude MUST NOT perform operation X
→ permissions / deterministic settings / hooks

Did the result actually satisfy X?
→ tests / verification
```

For deterministic blocking around tool calls, Anthropic points to hooks such as `PreToolUse` rather than relying only on prose instructions.

---

# 66. `CLAUDE.md` Is Delivered as Model Context, Not a Hard System Law

**Class: A**

Current Claude Code documentation explains that project instructions are delivered as contextual user-level content after the core system prompt rather than gaining absolute system-prompt enforcement semantics.

If multiple instructions conflict, the model may not deterministically resolve them the way a configuration engine would.

For stronger system-level prompting, Claude Code exposes separate mechanisms such as system-prompt append/managed policy, and for hard enforcement, hooks/settings remain the preferred layer.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 67. Auto Memory: Official Types

**Class: A**

Claude Code auto-memory uses four current types:

### `user`

Role, expertise, and working preferences.

### `feedback`

Corrections and approaches the user has confirmed.

### `project`

Ongoing work, deadlines, and decisions that are not reliably recoverable from repository/git state.

### `reference`

Where authoritative external information can be found.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 68. Auto Memory Avoids Reconstructible Technical Facts

**Class: A**

Claude Code is explicitly told not to waste auto-memory on information it can recover from:

- the repository architecture;
- file paths;
- reproducible debugging state;
- information already present in `CLAUDE.md`.

This is the clearest official Anthropic statement of the principle:

> **Store what would otherwise be lost or expensive to reconstruct.**

Primary source:

- https://code.claude.com/docs/en/memory

---

# 69. Auto Memory Is On by Default and Can Be Disabled

**Class: A**

Current Claude Code auto-memory is on by default in supported versions.

Documented controls include:

- `/memory` UI/toggle;
- `autoMemoryEnabled` settings;
- project/user settings as supported;
- environment variable `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

The modern auto-memory feature requires a sufficiently recent Claude Code version; Anthropic's current docs identify the initial support line as **2.1.59+**, with later subfeatures requiring newer versions.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 70. Claude Code Auto-Memory Storage and Scope

**Class: A**

Default local storage resembles:

```text
~/.claude/projects/<project>/memory/

├── MEMORY.md
├── user_role.md
├── feedback_testing.md
├── project_deadline.md
└── ...
```

Scope behavior:

- a Git repository defines the main auto-memory project identity;
- worktrees/subdirectories of the same repo share the project memory directory;
- outside Git, the project root defines the scope;
- auto-memory is machine-local by default;
- it does not automatically synchronize between different computers/cloud environments.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 71. Auto-Memory Project Directory Overrides

**Class: A**

Current Claude Code supports advanced ways to control project-memory directory identity, including environment/config directory mechanisms in newer versions.

`CLAUDE_CODE_PROJECT_DIR_NAME` combined with Claude config location can be used in supported releases to intentionally share/identify a memory directory across environments.

This is advanced configuration and version-sensitive.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 72. Correction: `autoMemoryDirectory` Is Settings-Scope Aware

**Class: A**

Version 1.0 of this document incorrectly stated that project/local configuration of `autoMemoryDirectory` was categorically refused.

Current Claude Code documentation says `autoMemoryDirectory` can be resolved through supported settings scopes—including user/project/local/policy/explicit settings—with workspace-trust rules controlling whether project/local configuration is honored safely.

The configured path must satisfy the current path/trust requirements (for example, absolute or home-relative forms as documented).

This correction replaces the stale v1.0 statement.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 73. `MEMORY.md` Is the Auto-Memory Index

**Class: A**

Claude Code uses `MEMORY.md` as a lightweight index/routing file and keeps detail in additional topic files.

```text
MEMORY.md
small routing index
    │
    ├── user_role.md
    ├── feedback_testing.md
    ├── project_x.md
    └── reference_y.md
```

At conversation startup Claude automatically loads only the first:

```text
200 lines
OR
25 KB
whichever comes first
```

Detailed topic files are read lazily when useful.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 74. Auto-Memory Index Overflow Behavior

**Class: A**

The 200-line/25-KB limit applies to the startup `MEMORY.md` index, not to the whole memory directory.

Current docs describe behavior such as:

- near the limit, Claude receives reminders to reorganize/trim the index;
- writes can still occur even if the index is over the startup limit;
- content beyond the startup limit is not loaded in the next session;
- detailed topic files can remain larger because they are selectively opened rather than all injected at startup.

This is deliberate hierarchical context management, not merely storage limitation.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 75. Auto-Memory UI Shows Saving and Recall Activity

**Class: A**

Claude Code exposes visible indicators such as saved/recalled memory counts or memory-writing/retrieval activity. This gives the user evidence that auto-memory is being written or read rather than making all memory behavior invisible.

The exact wording can vary by version/UI.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 76. Auto-Memory Frontmatter Can Track Modification Time

**Class: A**

In current Claude Code releases, when a memory file already uses YAML frontmatter, Claude Code can add/update a `modified` timestamp in ISO-8601 form.

A file without frontmatter is not automatically forced to gain frontmatter solely for this timestamp.

This behavior requires a newer Claude Code build (documented around the 2.1.214+ line).

Primary source:

- https://code.claude.com/docs/en/memory

---

# 77. Transcript Cleanup Does Not Delete Auto Memory

**Class: A**

Claude Code can prune older session transcripts according to transcript-cleanup settings such as `cleanupPeriodDays`.

Auto-memory files are excluded from ordinary transcript cleanup and remain until Claude or the user changes/removes them.

```text
session transcript lifecycle
        ≠
auto-memory lifecycle
```

Primary source:

- https://code.claude.com/docs/en/memory

---

# 78. `/memory` and `/context` Serve Different Inspection Purposes

**Class: A**

`/memory` surfaces memory/instruction locations and auto-memory controls. It can open/create the relevant memory locations where supported.

`/context` reports what is actually loaded into the active context.

This distinction matters:

```text
file exists / configured
      ≠
file is currently loaded
```

Primary source:

- https://code.claude.com/docs/en/memory

---

# 79. Explicit User Wording Routes to Different Persistence Layers

**Class: A/C**

Claude Code documentation distinguishes intents such as:

```text
“Remember X”
→ auto-memory candidate

“Add X to CLAUDE.md”
→ explicit human-authored project instructions
```

This is a useful product boundary between adaptive learned state and explicit durable policy/guidance.

---

# 80. Claude Code Compaction Does Not Delete Persistent Memory

**Class: A**

After context compaction, persistent disk-backed instruction/memory sources can be re-read or re-injected.

Current documented behavior includes:

- root/project `CLAUDE.md` context being reintroduced;
- unscoped rules being available again;
- auto-memory being available again;
- path-scoped/nested instructions reloading when matching files/subtrees are accessed.

Therefore:

> **Context compaction is not memory deletion.**

Primary sources:

- https://code.claude.com/docs/en/memory
- https://code.claude.com/docs/en/context-window

---

# 81. `InstructionsLoaded` and Other Diagnostics Improve Auditability

**Class: A**

Current Claude Code documentation exposes diagnostic mechanisms—including instruction-loading hooks/events and `/context`—that help determine which persistent instruction sources were loaded, when, and why.

This is important because model behavior should not be inferred solely from the existence of a file on disk.

Primary source:

- https://code.claude.com/docs/en/memory

---

# 82. Claude Code Subagents Can Have Independent Persistent Memory

**Class: A**

A subagent definition can request memory scope:

```text
memory: user
memory: project
memory: local
```

Typical locations are documented as patterns such as:

```text
user:    ~/.claude/agent-memory/<agent-name>/
project: .claude/agent-memory/<agent-name>/
local:   .claude/agent-memory-local/<agent-name>/
```

Each subagent can build specialized persistent knowledge rather than sharing one universal memory pool.

Primary source:

- https://code.claude.com/docs/en/sub-agents

---

# 83. Subagent Memory Obeys Global Auto-Memory Enablement

**Class: A**

If Claude Code auto-memory is globally disabled by setting/environment, a subagent `memory` declaration does not magically re-enable memory. The agent does not receive the normal persistent-memory read/write behavior in that disabled state.

Primary source:

- https://code.claude.com/docs/en/sub-agents

---

# 84. Subagent Startup Uses the Same Index Principle

**Class: A**

When subagent memory is enabled, its system/context setup includes memory instructions plus the initial portion of its own `MEMORY.md`, subject to the same general first-200-lines/25-KB startup pattern.

Read/Write/Edit tools needed for its memory workflow can be made available automatically as documented.

Primary source:

- https://code.claude.com/docs/en/sub-agents

---

# 85. Main-Agent Auto Memory Is Not Automatically Inherited by Ordinary Subagents

**Class: A**

A non-fork subagent does not automatically receive the parent conversation's auto-memory directory/content merely because the parent has memory.

A **forked** subagent inherits parent context as part of the fork semantics, which is different from sharing the same persistent auto-memory store.

Agent-specific persistent memory remains separately scoped.

Primary source:

- https://code.claude.com/docs/en/sub-agents

---

# 86. Subagent Resumption and Persistent Memory Are Different

**Class: A/C**

Claude Code can resume some subagent conversations by agent/session identity, preserving conversational history. That is distinct from the subagent's explicit persistent memory directory.

```text
resumed conversation history
        ≠
persistent agent memory files
```

This is the same general distinction seen throughout Claude: history and durable memory are different layers.

---

# 87. Claude API Memory Tool Is Client-Side

**Class: A**

Anthropic provides a developer-facing Memory Tool that gives Claude a filesystem-style interface under a logical prefix such as:

```text
/memories/
```

The tool is **client-side**: Anthropic does not require one specific storage backend. The developer implements persistence.

Possible mappings include:

- local filesystem;
- Postgres;
- S3/object storage;
- encrypted store;
- custom database/service.

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

---

# 88. API Memory Tool Configuration

**Class: A**

Current documentation uses a tool declaration with type/version approximately:

```text
memory_20250818
```

and tool name:

```text
memory
```

The memory tool is available to supported Claude 4+ models according to current docs.

SDK helper abstractions exist in several Anthropic SDKs, while some languages require implementing the tool loop more directly.

Exact SDK helper names and beta namespaces can evolve and should be checked against current SDK docs when implementing.

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

---

# 89. API Memory Tool Commands

**Class: A**

The model-facing command set includes operations such as:

```text
view
create
str_replace
insert
delete
rename
```

`view` can inspect files/directories and selected ranges. Other commands create or mutate the developer-owned store.

The developer is responsible for enforcing the documented contract and safely mapping paths to the underlying backend.

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

---

# 90. API Memory Uses Just-in-Time Retrieval

**Class: A**

Anthropic presents persistent memory as a context-engineering primitive: store durable information externally and retrieve only what is useful for the present request.

```text
large durable store
      ↓
selective read
      ↓
small relevant active context
```

This reduces pressure on the context window and lets state survive session boundaries.

Primary sources:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool
- https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools

---

# 91. API Memory and Compaction Solve Different Problems

**Class: A/C**

Anthropic recommends combining persistent memory with context editing/server-side compaction for long-running agents.

```text
ACTIVE CONTEXT
temporary working cognition

COMPACTION SUMMARY
compressed current-run continuity

PERSISTENT MEMORY
cross-session durable context

AUTHORITATIVE EXTERNAL SYSTEMS
canonical ground truth
```

Persistent memory is specifically useful because it survives compaction and session boundaries.

---

# 92. API Memory Security Requirements

**Class: A**

Anthropic recommends defensive implementation controls including:

- strictly confining paths to the memory prefix;
- preventing path traversal and encoded traversal variants;
- file-size caps;
- paging large reads/listings;
- sensitive-data validation;
- expiration/removal of stale unused memory;
- appropriate encryption/storage controls for the application's risk profile.

These recommendations reinforce that a memory filesystem is a privileged durable-state interface.

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

---

# 93. API Memory Expiration Is an Explicit Design Recommendation

**Class: A/C**

Anthropic explicitly recommends periodically removing memories that have not been accessed for a long time in suitable implementations.

Architectural lesson:

> **Forgetting/garbage collection is a system-health feature, not only a privacy action.**

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

---

# 94. Managed Agents Use Persistent Memory Stores

**Class: A**

Managed Agents provide workspace-scoped **Memory Stores**, collections of text documents that can be attached to agent sessions.

The agent receives memory-store metadata/instructions and accesses attached stores through its filesystem/tool environment.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 95. Managed Memory Limits — Current Correct Values

**Class: A**

Current documented limits are:

### Per memory document

**100 KB**, approximately **25K tokens**.

### Per Memory Store

Up to **10,000 memories**.

### Memory Stores attached to one session

Up to **8 stores**.

Version 1.0 of this document incorrectly listed **2,000 memories per store**. The current official limit is **10,000**, and v1.1 corrects that stale value.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 96. Store Attachments Are Chosen at Session Creation

**Class: A**

Memory Stores are attached when a Managed Agent session is created. Current docs do not treat store attachment as a freely mutable property of an already-running session.

Each attachment can carry:

- access mode;
- store description/name context;
- optional instructions (currently bounded to about 4096 characters).

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 97. Managed Memory Access Modes

**Class: A**

A Memory Store can be attached as:

```text
read_write
```

or:

```text
read_only
```

`read_write` is the normal default where not otherwise constrained.

This supports architectures such as:

```text
Agent
 ├── company standards      READ ONLY
 ├── shared references      READ ONLY
 ├── user preferences       READ/WRITE
 └── project memory         READ/WRITE
```

Read-only attachments are a major security/correctness control.

---

# 98. Managed Memory Mount Paths

**Class: A**

Memory Stores appear under the agent memory mount hierarchy, commonly under:

```text
/mnt/memory/...
```

The store API/session response returns the actual `mount_path`. Anthropic recommends using the returned mount path rather than constructing one from the display name yourself.

The `/mnt/memory` parent is controlled; writes must target an actual attached writable store rather than arbitrary sibling locations.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 99. Managed vs Self-Hosted Memory Mounting

**Class: A**

For Anthropic-managed sandboxes, attached Memory Stores behave like durable mounted storage within the agent environment.

For **self-hosted** sandboxes, the implementation is not a live remote mount. The worker downloads/synchronizes a local copy of Memory Store content and periodically synchronizes mutations back to the store.

This is important when reasoning about consistency and concurrency.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 100. Self-Hosted Memory Synchronization Semantics

**Class: A**

Current Managed Agent documentation describes self-hosted synchronization behavior approximately as:

- synchronize after relevant tool activity;
- rate-limit/background synchronization to roughly once per **15 seconds** by default;
- perform a final synchronization when the session ends normally;
- another self-hosted worker may not see changes until both sides have synchronized;
- writes made outside recognized Memory Store mounts do not become durable store state;
- read-only mounts refuse mutation through supported write/edit paths.

This means self-hosted Memory Stores are **eventually synchronized local copies**, not instantly coherent shared filesystems.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 101. Managed Memory Listing Semantics

**Class: A**

The Memory Store API supports listing with path-prefix/depth controls. Current docs define segment-aware path-prefix behavior and depth semantics for retrieving whole subtrees versus immediate children.

Ordering is server-defined/stable for the API rather than something the model should infer from filesystem naming alone.

Exact API parameters should be checked against current platform docs during implementation.

---

# 102. Managed Memory Create/Update/Delete Semantics

**Class: A**

Memory Store operations support:

- creating a new memory path;
- retrieving a memory;
- updating content;
- renaming/moving a path;
- updating path and content together;
- deleting a memory.

Create does not silently overwrite an existing path.

At the store limit, creating new memory paths fails while reads/updates to existing memories remain possible under current docs.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 103. Managed Memory Optimistic Concurrency

**Class: A**

Managed Agent Memory supports optimistic concurrency with a `content_sha256` precondition.

```text
read memory + hash A
       ↓
another writer updates it
       ↓
write expecting hash A
       ↓
precondition conflict
       ↓
re-read
merge
retry
```

This prevents silent last-writer-wins clobbering when multiple agents/tools share durable state.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 104. Every Managed Memory Mutation Creates Version History

**Class: A**

Managed Memory mutations create immutable version records (`memver_...`-style identifiers).

Version history can record:

- operation/change;
- actor;
- timestamp;
- historical content state.

Actors can include agent sessions, API keys, human Console users, and service accounts according to the Managed Agent version/audit model.

Primary sources:

- https://platform.claude.com/docs/en/managed-agents/memory
- https://platform.claude.com/docs/en/api/http/beta/memory_stores/memory_versions

---

# 105. Version History Survives Live Memory Deletion

**Class: A**

Deleting the current memory object does **not automatically remove its immutable version records**.

This creates an audit/history layer separate from the live memory object.

Recent versions are retained under Anthropic's documented retention guarantees, commonly around a 30-day historical window, while infrequently changed memory may retain older live-relevant history longer under the documented model.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 106. Redaction Preserves the Audit Event

**Class: A**

Managed Agent version history supports redacting sensitive historical content while preserving the version/audit record that a change occurred.

Redaction is therefore different from deleting the audit trail itself.

The current live/head content has separate handling: historical-version redaction is not a substitute for changing/deleting the current memory state.

Primary sources:

- https://platform.claude.com/docs/en/managed-agents/memory
- https://platform.claude.com/docs/en/api/http/beta/memory_stores/memory_versions

---

# 107. There Is No Magical One-Click Version Restore Contract

**Class: A**

Managed Agent APIs expose historical content/version records, but recovery is conceptually performed by reading the desired historical content and writing it back as new current state rather than rewinding the entire store through an opaque hidden rollback operation.

For long-term audit requirements beyond Anthropic's retention window, export/version archiving should be handled explicitly.

---

# 108. Memory Store Lifecycle: Archive and Delete

**Class: A**

Managed Memory Stores can be managed independently of their memories.

Current docs distinguish archiving from deleting:

- archived stores are excluded from normal active listings/attachments as documented and are effectively frozen for new session use;
- archiving is not the same as deleting the store's data;
- deleting a store permanently removes the store and its contained memories/version history according to current API semantics.

Archive behavior is deliberately not equivalent to a reversible active-state toggle in every current workflow; check the current API before assuming unarchive support.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

---

# 109. Managed Agent Memory Is a Prompt-Injection Persistence Boundary

**Class: A**

Anthropic explicitly warns:

```text
untrusted input
   ↓
prompt injection
   ↓
agent with writable memory
   ↓
malicious persistent state
   ↓
future session reads it
```

Recommended architecture:

- writable stores only where learning is necessary;
- read-only stores for reference/standards where possible;
- strong source trust boundaries;
- avoid letting arbitrary untrusted content become durable instructions.

Persistent memory extends the blast radius of a one-turn prompt injection into future sessions.

---

# 110. Managed Agent Memory Store API Uses Separate Beta/Feature Headers

**Class: A**

Current Managed Agent and Memory Store APIs use versioned feature headers. The Managed Agents session API and the Memory Store endpoints can require different beta identifiers; they should not be blindly combined on every request.

Examples in current docs include Managed Agents and Agent Memory feature-date identifiers such as:

```text
managed-agents-2026-04-01
agent-memory-2026-07-22
```

These identifiers are API-version details and may change. Always verify against current docs before implementation.

---

# 111. Dreams Are a Separate Memory-Consolidation System

**Class: A**

Managed Agents provide a research-preview capability called **Dreams**.

Anthropic explicitly frames Dreams as a response to long-lived memory stores accumulating:

- duplicates;
- contradictions;
- stale facts;
- fragmented organization.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/dreams

---

# 112. Dream Inputs and Outputs

**Class: A**

A Dream accepts:

```text
one existing Memory Store
+
1–100 historical Managed Agent sessions
```

and produces:

```text
A NEW Memory Store
```

The input Memory Store is **not modified**.

The output can then be reviewed, used for future sessions, or discarded.

---

# 113. Dreams Are Asynchronous

**Class: A**

Dream processing is asynchronous and has lifecycle states such as:

```text
pending
running
completed
failed
canceled
```

An output Memory Store may exist before the dream has finished populating it.

Failed/canceled dreams can leave a partial output store rather than automatically deleting every produced artifact.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/dreams

---

# 114. Dreams Preserve Inputs and Historical Sessions

**Class: A**

Dreaming does not mutate or delete the input Memory Store or source sessions.

The underlying processing session can remain archived rather than being treated as disposable invisible state.

This makes Dreaming suitable for reviewable consolidation rather than destructive in-place rewriting.

---

# 115. Dream Instructions Are High-Level Consolidation Guidance

**Class: A**

Dream creation can include optional instructions (currently bounded around 4096 characters) that guide consolidation.

These are meant for high-level synthesis/organization goals, not as a precision line-editor for an existing store. For targeted deterministic edits, use Memory Store APIs directly.

---

# 116. Dreams Have Model Constraints

**Class: A**

Dreams support a defined set of Claude models rather than arbitrary models. Current documentation includes supported models from the Fable/Opus/Sonnet families, and that set can change.

Because model support is a fast-moving implementation detail, the current Dreams page should be treated as authoritative at execution time.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/dreams

---

# 117. Dream Input Availability Is Required Throughout the Run

**Class: A**

If an input Memory Store or required source session becomes unavailable (for example through deletion/archive in a way the Dream cannot access) while Dream processing is running, the Dream can fail with an input-unavailable error.

This means consolidation has real dependency/lifecycle constraints.

---

# 118. Dream Cancellation and Archiving

**Class: A**

Dreams can be canceled according to their lifecycle state. Completed/failed/canceled records can be archived under the documented API lifecycle.

Archiving the Dream record is distinct from deleting its output Memory Store.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/dreams

---

# 119. Dream Billing and Scale

**Class: A**

Dreams are billed using ordinary model token pricing for the selected model. Cost grows roughly with the amount/length of input memory and source sessions.

Documented hard inputs include:

- maximum **100 sessions** per Dream;
- optional instruction-size limit;
- supported model list;
- organization/store capacity constraints.

Primary source:

- https://platform.claude.com/docs/en/managed-agents/dreams

---

# 120. Dreams Create a Two-Speed Memory Architecture

**Class: A/C**

Anthropic now publicly exposes both:

```text
FAST LOOP
conversation/session
   ↓
incremental memory update
```

and:

```text
SLOW LOOP
many sessions + accumulated memory
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

> **Persistent memory requires consolidation, not merely accumulation.**

---

# 121. Cross-Product Design Principle: Externalize Durable State

**Class: A/C**

Consumer Claude, Claude Code, API Memory, and Managed Agents all separate model cognition from persistent storage.

```text
context window = temporary cognition
external store = durable state
```

This is the most fundamental commonality across Anthropic memory systems.

---

# 122. Cross-Product Design Principle: Keep Active Context Sparse

**Class: A/C**

Claude Code explicitly uses a small startup index plus on-demand detail. API Memory is explicitly designed for JIT retrieval. Consumer implementation evidence strongly suggests a similar memory listing + file read pattern.

Thus Anthropic repeatedly favors:

```text
small routing layer
+
selective detail retrieval
```

over loading the entire durable store into every prompt.

---

# 123. Cross-Product Design Principle: Truth and Memory Are Different

**Class: A/C**

Repositories, databases, documents, email, calendars, APIs, and other connected systems should remain authoritative for their own state.

Memory should generally preserve:

- user-specific preferences;
- corrections;
- decisions;
- context difficult to reconstruct;
- pointers to canonical systems.

It should not become a stale shadow database of re-queryable reality.

---

# 124. Cross-Product Design Principle: Preserve Provenance

**Class: A/B/C**

Consumer captured memory distinguishes user-stated vs observed/inferred provenance. Managed Memory has actors/version history. Claude Code distinguishes user feedback from project/reference memories.

Therefore a mature memory system should know **why it believes something**, not only store the proposition.

---

# 125. Cross-Product Design Principle: Scope Memory

**Class: A/C**

Anthropic uses multiple independent scope boundaries:

- non-project vs Project consumer memory;
- project-specific chat search;
- user/project/local Claude Code/subagent memory;
- workspace Memory Stores;
- per-session Memory Store attachments;
- read-only vs read-write access.

Long-term memory is not assumed to be globally visible to every agent/surface.

---

# 126. Cross-Product Design Principle: Human Editability and Auditability

**Class: A/C**

Consumer Topics are editable. Claude Code memory is plain files. API Memory is developer-controlled. Managed Memory adds immutable versions and redaction.

Persistent model beliefs need inspection and correction proportional to their consequence.

---

# 127. Cross-Product Design Principle: Concurrency Matters

**Class: A/B/C**

Consumer captured writes use `if_version` evidence; Managed Memory officially uses `content_sha256` preconditions.

Once multiple surfaces/agents can write a shared durable store, memory requires ordinary distributed-state correctness mechanisms.

---

# 128. Cross-Product Design Principle: Memory Is a Security Boundary

**Class: A/C**

Writable memory can turn transient prompt injection into persistent compromise.

Therefore robust systems need:

- source trust;
- read-only reference stores;
- path validation;
- constrained write capability;
- provenance;
- review/version history where important;
- explicit instruction-authority separation.

---

# 129. Cross-Product Design Principle: Forget and Consolidate

**Class: A/C**

Anthropic recommends memory expiration in the API Memory Tool and provides Dreams for consolidation. Consumer captured memory also includes size-based consolidation/pruning behavior.

A good memory system is not append-only by default.

It must manage:

- redundancy;
- staleness;
- contradiction;
- low-value information;
- privacy erasure;
- capacity pressure.

---

# 130. Memory Is Not Instructions, and Instructions Are Not Enforcement

**Class: A/C**

A reliable Claude harness should distinguish:

```text
SOURCE OF TRUTH
what is actually true

MEMORY
what durable context Claude should know

INSTRUCTIONS / CLAUDE.md / RULES
how Claude should generally behave

SKILLS / PROCEDURES
how Claude should perform recurring work

HOOKS / PERMISSIONS / POLICY
what must/must not happen deterministically

TESTS / OBSERVATION
whether the real outcome is correct
```

Using memory as deterministic policy enforcement is an architectural mistake.

---

# 131. Complete Failure Taxonomy

A serious Claude-like memory system must handle at least these failure categories.

### Capture failure
Useful durable information is never admitted.

### Over-capture
Temporary or irrelevant details become persistent.

### Provenance failure
Model/tool/inference content is silently promoted to user-established truth.

### Entity duplication
One person/project becomes several inconsistent identities.

### Misrouting
A fact is stored under the wrong semantic subject/domain.

### Staleness
Old state remains active after circumstances change.

### Contradiction
Incompatible states coexist without resolution.

### Retrieval failure
The right memory exists but is not found/read.

### Ranking/routing failure
The right memory is available but loses to less relevant context.

### Scope failure
Correct state is inaccessible due to intended or accidental namespace boundaries.

### Scope leakage
State crosses a Project/agent/org/privacy boundary where it should not.

### Over-personalization
Memory affects a response where it should not.

### Under-personalization
Relevant durable state is ignored.

### Correction failure
New user-established state fails to supersede/reconcile old state.

### Forgetting failure
Explicitly removed state persists or is recreated.

### Dependency-erasure failure
Derived facts survive after their only supporting fact was erased.

### Concurrency failure
One writer overwrites another writer's newer change.

### Memory poisoning
Untrusted instructions/data become durable malicious state.

### Instruction escalation
Persisted data incorrectly gains policy/system authority.

### Capacity failure
Memory/index grows beyond useful context/retrieval limits.

### Entropy failure
Duplicates and stale fragments degrade retrieval over time.

### Consolidation failure
Cleanup merges distinct facts incorrectly or preserves stale ones.

### Audit failure
No reliable explanation exists for who/what changed durable state.

### Retention mismatch
User expectations do not match actual chat/memory/version retention.

### Model-use failure
Correct context reaches Claude but Claude misinterprets or ignores it.

### Source-of-truth divergence
Memory becomes a stale duplicate of a live external system.

---

# 132. Consumer Experiments That Could Reduce Unknowns

Document research has diminishing returns on undocumented internals. Controlled experiments should test:

## Write latency

Compare natural durable facts vs explicit “remember X” and measure Topic visibility/new-chat availability.

## Admission threshold

Introduce equally durable facts with different repetition/importance levels.

## Profile threshold

Test stable identity vs dated/temporary project state and inspect file routing where possible.

## Provenance

Compare direct user statement, Claude inference, web/tool result, and user-confirmed tool result.

## Entity resolution

Refer to the same person/project through aliases and inspect whether memory remains canonical.

## Correction

Establish A, later B, and inspect chronology/current-state behavior.

## Explicit forgetting

Create a fact plus a dependent fact, forget the source, and inspect both.

## Concurrency

Perform near-simultaneous Chat/cloud-Cowork updates to one topic where practical.

## Sparse retrieval

Create many memory topics and observe which detailed files activate for targeted questions.

## Project boundaries

Duplicate facts inside/outside Projects and test leakage in both directions.

## Incognito

Verify no memory read/write while other personalization remains available.

## Import/export fidelity

Export, re-import into a clean controlled state, compare loss/restructuring.

## Scale

Build hundreds/thousands of facts/topics and measure retrieval degradation/pruning behavior.

---

# 133. Claude Code Experiments

Useful black-box tests include:

- verify `MEMORY.md` 200-line vs 25-KB startup cutoff independently;
- place relevant data beyond the cutoff and confirm it is not injected until explicitly read elsewhere;
- measure index-overflow reminders;
- compare worktrees and distinct repos/machines;
- test `autoMemoryDirectory` across user/project/local scopes with workspace trust on/off;
- verify `modified` frontmatter behavior;
- verify transcript cleanup does not remove auto-memory;
- compare normal vs forked subagent inheritance;
- compare disabled auto-memory with subagent `memory:` declarations;
- inspect `/context` before/after compaction.

---

# 134. API / Managed Agent Experiments

Useful implementation tests include:

- `content_sha256` conflict injection and merge/retry;
- simultaneous self-hosted workers and 15-second synchronization visibility;
- writes outside actual returned mount paths;
- read-only mutation attempts;
- store-limit behavior at 10,000 paths;
- deletion followed by version-history retrieval;
- redaction while preserving audit event;
- Dream completion vs partial output on cancellation/failure;
- input deletion/archive mid-Dream;
- Dream consolidation quality across duplicates/contradictions;
- read-only old store + fresh read/write consolidated store architecture.

---

# 135. What We Know With Very High Confidence

1. Current consumer memory uses individual categorized topics/files rather than one daily global summary for normal users.
2. Chat and cloud Cowork share current consumer memory.
3. Local Cowork does not share that cloud memory.
4. Projects have isolated memory spaces and dedicated summaries.
5. Project Knowledge/RAG is separate from Project memory.
6. Past Chat Search is a separate RAG system with project boundaries.
7. Memory and chat search have separate controls/availability.
8. Pause stops both current-memory read and write without deleting stored memory.
9. Reset deletes generated memory including Project memory.
10. Deleting a source chat does not automatically delete already-generated current memory.
11. Memory Topics are individually editable.
12. Consumer memory supports import/export.
13. Sensitive memory is opt-in and separately governed.
14. Incognito neither reads nor writes ordinary memory/history.
15. Incognito can still receive non-memory personalization.
16. Team/Enterprise have separate organization controls and retention/export behavior.
17. Consumer memory data is included in data exports.
18. Enterprise memory entries are encrypted at rest.
19. Claude Code separates human-authored instructions from auto-memory.
20. Claude Code auto-memory has user/feedback/project/reference types.
21. Claude Code avoids storing repository-reconstructible facts.
22. Claude Code uses `MEMORY.md` as a startup index and lazy topic files.
23. Startup `MEMORY.md` is bounded to first 200 lines or 25 KB.
24. Claude Code auto-memory is machine-local by default.
25. Auto-memory can be toggled/disabled.
26. Auto-memory survives ordinary transcript cleanup.
27. Claude Code subagents can have user/project/local memories.
28. Ordinary subagents do not automatically inherit main-agent auto-memory.
29. API Memory is client-owned persistent storage exposed through a filesystem-like tool.
30. API Memory is designed for JIT retrieval and combination with compaction.
31. Managed Agent Memory Stores support up to 10,000 memories per store, 100 KB per memory, and up to 8 stores/session under current docs.
32. Managed Memory supports read-only/read-write scopes.
33. Managed Memory supports `content_sha256` optimistic concurrency.
34. Managed Memory maintains immutable version history.
35. Historical versions can survive deletion of the live memory.
36. Historical content can be redacted while preserving an audit record.
37. Self-hosted Managed Agent memory uses synchronized local copies rather than a live network mount.
38. Writable persistent memory creates a prompt-injection persistence risk.
39. Dreams take one input store plus 1–100 sessions and create a separate output store.
40. Dreams do not modify the input store.
41. Dreams are asynchronous and can leave partial output on failure/cancel.
42. Anthropic explicitly treats consolidation/forgetting as necessary for long-lived memory systems.

---

# 136. Hard Unknowns — Consumer Retrieval

These remain **U**:

- whether consumer memory uses embeddings;
- which embedding model, if any;
- whether a vector database/search service is used;
- lexical vs semantic routing details;
- recency weighting;
- repetition/frequency weighting;
- salience/importance weighting;
- retrieval thresholds;
- maximum detailed files read per turn;
- detailed-memory token budget;
- whether the main Claude model selects files itself;
- whether a separate router/classifier preselects files;
- whether server-side pre-ranking/reranking occurs;
- exact fallback behavior when relevant files exceed context budget.

---

# 137. Hard Unknowns — Consumer Memory Writing

**U:**

- exact model used for the post-turn/background memory pass;
- whether it is the same Claude model with a special prompt;
- whether smaller classifiers participate;
- exact admission score/threshold;
- exact durability/repetition formula;
- precise correction-merging algorithm;
- exact consumer conflict-resolution implementation;
- whether there are additional periodic maintenance passes beyond turn-level filing;
- whether consumer memory uses an unpublished Dreams-like consolidator.

---

# 138. Hard Unknowns — Consumer Storage

**U:**

- physical database/object-store technology;
- exact backend record schema;
- whether each model-facing file maps one-to-one to one backend object;
- exact total memory capacity per user/account;
- exact per-topic/file capacity;
- cache architecture;
- tenant/shard model;
- consistency guarantees between Chat and cloud Cowork;
- exact deletion propagation mechanics;
- exact internal provenance schema beyond captured evidence.

---

# 139. Hard Unknowns — Consumer Consolidation and Decay

**U:**

- whether old consumer files automatically merge outside capacity pressure;
- whether unused memories decay automatically;
- whether successful retrieval reinforces retention;
- whether salience changes with frequency/use;
- whether there is a hidden periodic “Dreaming” equivalent for consumer memory;
- whether legacy migration and new-memory cleanup share infrastructure.

---

# 140. Hard Unknowns — Cross-Surface Consumer Writers

Officially confirmed:

```text
Chat ↔ cloud Cowork
```

Unknown:

- complete list of Claude surfaces able to write the same consumer store;
- whether Claude in Chrome/Excel/PowerPoint/other product surfaces can directly mutate it or only through Cowork/Chat mediation;
- whether every surface uses the same provenance tags/file schema;
- whether consumer-memory concurrency handling is exactly the captured `if_version` implementation in production for all users.

---

# 141. Things We Must Not Claim Without New Evidence

Do not say:

### “Claude consumer memory definitely uses a vector database.”

Unknown.

### “Every memory is injected into every prompt.”

Strong evidence indicates sparse/selective loading.

### “Anthropic physically stores Markdown files on disk for consumer memory.”

Unproven; the filesystem may be a model-facing abstraction.

### “Claude Code and Claude Chat use one shared store.”

No official evidence.

### “Projects search all chats everywhere.”

False; project boundaries apply.

### “Deleting a chat deletes current generated memory.”

False in the current topic-based architecture.

### “Memory guarantees Claude follows a preference.”

Memory is context, not enforcement.

### “Dreams run on ordinary consumer memory.”

Unknown.

### “The Fable 5.1 prompt capture is an official Anthropic contract.”

False; it is Class B evidence.

### “Managed Memory Store capacity is 2,000.”

Stale. Current docs say **10,000 memories per store**.

### “`autoMemoryDirectory` can never come from project/local settings.”

Stale. Current docs describe settings-scope behavior governed by workspace trust.

### “Incognito means zero retention.”

False; documented retention still applies.

### “Monthly Recap directly ingests raw Gmail/Drive content.”

False under current documentation; raw connector content is excluded, though Claude-authored conversation summaries may contribute.

---

# 142. Canonical Architecture

```text
                              SOURCES OF TRUTH
          ┌───────────────┬───────────────┬───────────────┐
          │               │               │               │
        repos           docs            apps            DB/APIs
          │               │               │               │
          └───────────────┴───────┬───────┴───────────────┘
                                  │
                                  ▼
                                TOOLS
                                  │
                  ┌───────────────┴────────────────┐
                  │                                │
                  ▼                                ▼
          HISTORICAL EVIDENCE                DURABLE MEMORY
            chat search/RAG              topics/files/stores
                  │                                │
                  │                        compact routing state
                  │                                │
                  │                         JIT detail retrieval
                  │                                │
                  └───────────────┬────────────────┘
                                  │
                         SCOPE / PERMISSIONS
                                  │
                         RELEVANCE / ROUTING
                                  │
                                  ▼
                           ACTIVE CONTEXT
                                  │
                     instructions / rules / tools
                                  │
                                  ▼
                                MODEL
                                  │
                                  ▼
                              RESPONSE
                                  │
                     explicit correction/forget
                                  │
                    background/agent memory writes
                                  │
                         version-safe persistence
                                  │
                                  ▼
                              FUTURE USE
                                  │
                    periodic pruning/consolidation
                                  │
                                  ▼
                                DREAMS
                       (Managed Agents today)
```

---

# 143. Core Design Principle

The strongest abstraction across everything currently known is:

> **External systems hold truth. Memory preserves durable context that is hard or expensive to reconstruct. Historical search recovers exact episodes. Scope controls where information may flow. Sparse retrieval controls what reaches context. Provenance controls what should be trusted. Versioning protects shared writes. Explicit correction and forgetting repair state. Security boundaries prevent poisoned memory from gaining authority. Consolidation prevents long-lived memory from decaying into duplicates, contradictions, and stale assumptions.**

---

# 144. Primary Evidence Registry

## Current Consumer Claude

### Use Claude's chat search and memory to build on previous context

Canonical current source for generated memory, past-chat search, Projects, pause/reset, sensitive controls, deletion semantics, plan availability, organization governance, data exports, retention, encryption, and legacy migration notes.

https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

### Claude Release Notes

Timeline source for the 2026 memory rollout/evolution including July 10 and August 25 changes.

https://support.claude.com/en/articles/12138966-release-notes

### Claude's memory works everywhere and you decide what's in it

Official source for short files under Topics and shared Chat/cloud-Cowork memory.

https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it

### Import and export your memory from Claude

Memory portability, work-focused import extraction, export behavior, and legacy transition.

https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

### RAG for Projects

Project Knowledge RAG, all-plan current availability, automatic retrieval behavior, and approximate 10× practical capacity claim.

https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects

### Incognito Chats

No ordinary memory/history/search, 30-day default retention, organization export behavior, and non-memory personalization distinction.

https://support.claude.com/en/articles/12260368-use-incognito-chats

### Monthly Recap

Reflect behavior, exclusions, connected-source handling, availability, and recap metrics.

https://support.claude.com/en/articles/15672559-see-your-monthly-recap

---

# 145. Claude Code Evidence Registry

### Claude Code — Memory

Canonical source for `CLAUDE.md`, rules, imports, scopes, auto-memory types, storage, startup index limits, configuration, `modified` timestamps, transcript cleanup, diagnostics, compaction interactions, and memory philosophy.

https://code.claude.com/docs/en/memory

### Claude Code — Context Window

Canonical companion source for context compaction/reinjection behavior.

https://code.claude.com/docs/en/context-window

### Claude Code — Subagents

Canonical source for user/project/local subagent memory scopes, directories, enablement, startup behavior, and non-inheritance distinctions.

https://code.claude.com/docs/en/sub-agents

---

# 146. Claude Platform Evidence Registry

### Memory Tool

Canonical source for client-owned `/memories` abstraction, tool commands, JIT retrieval, implementation security, memory expiration, and compaction integration.

https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

### Context engineering tools

Additional Anthropic context-engineering examples for selective/JIT memory retrieval.

https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools

### Managed Agent Memory

Canonical source for Memory Stores, current **10,000-memory** limit, 100-KB documents, eight-store session attachments, access modes, mount paths, self-hosted sync, security warnings, optimistic concurrency, versions, archive/delete lifecycle, and store operations.

https://platform.claude.com/docs/en/managed-agents/memory

### Memory Versions

Canonical API source for immutable version records, actors, historical content, and redaction/audit behavior.

https://platform.claude.com/docs/en/api/http/beta/memory_stores/memory_versions

### Dreams

Canonical source for offline consolidation from one Memory Store + 1–100 sessions into a new store, asynchronous lifecycle, instructions, model restrictions, cancellation/failure semantics, and billing/limits.

https://platform.claude.com/docs/en/managed-agents/dreams

---

# 147. Implementation-Evidence Registry

### Captured Claude Fable 5.1 system prompt

Primary Class B evidence for:

- consumer `memory_filesystem` abstraction;
- memory read/write/list/delete operations;
- `/profile.md`, `/preferences.md`, `/topics/`, `/areas/`, `/people/` taxonomy;
- `<memory_listing>` routing view;
- `name`, `description`, `sources`, aliases, `[[links]]`;
- three-month profile stability test;
- under-300-word profile target;
- post-turn background memory pass;
- foreground explicit remember/forget path;
- stated/observed/inferred provenance concepts;
- user-confirmation admission semantics;
- semantic file routing;
- `if_version` concurrency protocol;
- read-before-mutation behavior;
- correction vs forgetting semantics;
- dependent-fact deletion;
- file-capacity consolidation;
- broader internal sensitive-data protections;
- memory-as-untrusted-context rules.

https://github.com/elder-plinius/CL4R1T4S/blob/93b0ae6fb503db6642e58f9d6352db973a900cdc/ANTHROPIC/Claude-Fable-5.1.md

### Captured Claude Opus 5 system prompt

Contains highly similar memory instructions in the same external capture repository. Treat as same-source corroboration, not independent Anthropic confirmation.

https://github.com/elder-plinius/CL4R1T4S/blob/93b0ae6fb503db6642e58f9d6352db973a900cdc/ANTHROPIC/OPUS-5.md

---

# 148. Version 1.1 Corrections From Version 1.0

Version 1.1 explicitly corrects these v1.0 problems:

1. **Managed Memory Store capacity:** v1.0 said 2,000 memories/store. Current official documentation says **10,000 memories/store**.
2. **`autoMemoryDirectory`:** v1.0 described project/local configuration as categorically refused. Current docs describe settings-scope resolution governed by workspace trust.
3. Added current consumer memory export, encryption, organization audit, and retention details.
4. Added work-focused import filtering and experimental import semantics.
5. Added Monthly Recap connector/input/exclusion details.
6. Added exact captured Fable profile stability test and **under-300-word** target.
7. Added exact `<memory_listing>` routing behavior and observed metadata schema.
8. Added read-before-versioned-mutation and fine-grained forgetting behavior.
9. Added broader captured sensitive-data rules while clearly keeping them Class B.
10. Added Claude Code startup/load limits, `/context`, `/memory`, imports, rules, auto-memory toggles, index-overflow, `modified` timestamps, transcript cleanup, and subagent non-inheritance.
11. Added Managed Agent mount-path and self-hosted synchronization semantics, including approximately 15-second sync cadence.
12. Added current Managed Memory version survival/redaction/audit behavior.
13. Added deeper Dreams lifecycle, error/dependency, model, instruction, billing, and partial-output behavior.

---

# 149. Change Log

## Version 1.1 — September 8, 2026

Rebuilt v1.0 from a comprehensive architecture overview into an exhaustive evidence ledger.

Added or expanded:

- plan/platform availability;
- exact current/legacy boundaries;
- data export, retention, encryption, and audit behavior;
- full sensitive-memory and Incognito semantics;
- Monthly Recap inputs/exclusions;
- exact Fable memory-listing/profile/frontmatter/concurrency/deletion evidence;
- Claude Code instruction scopes/imports/rules/load mechanics;
- auto-memory toggles, versions, storage overrides, index overflow, timestamps, diagnostics;
- subagent memory directories/enablement/non-inheritance;
- API Memory Tool configuration/commands/security/expiration;
- Managed Agent current limits and attachment semantics;
- managed vs self-hosted memory synchronization;
- Memory Store concurrency/version/redaction/archive lifecycle;
- Dreams async lifecycle, dependencies, partial outputs, models, instructions, limits and billing;
- expanded failure taxonomy;
- expanded experimental agenda;
- hard-unknown registry;
- claims that must not be made;
- explicit completeness boundary.

Corrected:

- Managed Store capacity from stale 2,000 to current **10,000** memories per store.
- stale `autoMemoryDirectory` scope characterization.

## Version 1.0 — September 8, 2026

Initial canonical Claude memory source-of-truth document.

---

# 150. Current Final Conclusion

As of September 8, 2026:

**The strongest available evidence indicates that Claude memory is an external, persistent, selectively retrieved family of context systems rather than a property stored inside the model itself.**

Consumer Claude now uses individual categorized memory topics/files, with Project-scoped memory and separate historical-chat RAG. Chat and cloud Cowork share consumer memory. Claude Code independently exposes a transparent local **index + topic files + lazy retrieval** architecture alongside human-authored instructions. The Claude API exposes a developer-owned filesystem-like memory primitive. Managed Agents extend the pattern with scoped persistent stores, read/write controls, current 10,000-memory store capacity, version history, optimistic concurrency, sandbox/self-hosted synchronization, and offline consolidation through Dreams.

The strongest cross-product principle is:

> **Store the irrecoverable residue; re-query authoritative reality.**

The full design principle is:

> **Source systems hold truth. Memory stores durable user/project/agent context that is hard or expensive to reconstruct. Historical retrieval recovers exact episodes. Scope controls what can flow where. Sparse retrieval controls what reaches the model. Provenance controls what should be trusted. Explicit corrections repair state. Versioning prevents clobbering. Security boundaries prevent poisoned memory from becoming policy. Garbage collection and Dreams prevent long-lived memory from decaying into duplicates, contradictions, and stale assumptions.**

Within the evidence corpus listed above, this Version 1.1 document is the canonical exhaustive source of truth. Anything not established here belongs in the Unknowns registry until new evidence appears.
