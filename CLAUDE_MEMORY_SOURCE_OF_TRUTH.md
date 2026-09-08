# Claude Memory — Master Source of Truth

**Version:** 1.2 — Exhaustive Current-State Reconciliation  
**Last verified:** September 8, 2026  
**Supersedes:** Version 1.1  
**Scope:** Claude consumer Chat memory, Projects, cloud Cowork, past-chat search, import/export, Incognito, sensitive-memory controls, organization governance, Monthly Recap, Claude Code `CLAUDE.md`/rules/auto-memory/compaction/subagents, Claude API Memory Tool, Managed Agent Memory Stores, Dreams, captured consumer implementation evidence, failure modes, experiments, and hard unknowns.  
**Purpose:** Maintain one canonical, evidence-graded, deliberately exhaustive account of what is currently established about Claude memory across the evidence corpus listed here.

> **Completeness boundary:** “Exhaustive” means exhaustive with respect to the current public Anthropic documentation and the specifically identified implementation evidence reviewed as of the verification date. It does **not** mean undocumented production internals are known. Anything not supported by that corpus remains explicitly unknown.

---

# 1. Executive conclusion

The strongest evidence supports one high-level conclusion:

> **Claude does not have one memory system. It has a family of external persistence, retrieval, instruction, knowledge, and consolidation mechanisms whose behavior differs by product surface.**

The common pattern is:

```text
model context      = temporary working cognition
external memory    = durable adaptive state
historical search  = recovery of specific prior episodes
source systems     = authoritative reality
instructions       = behavioral guidance
hooks/settings     = deterministic enforcement
```

Current Claude consumer memory uses individual categorized memory topics/files plus a separate historical chat-search system. Projects create isolated memory/search domains and dedicated project summaries while also providing Project Knowledge and RAG. Claude Chat and **cloud Cowork** share consumer memory; local Cowork does not. Claude Code independently uses human-authored instruction files plus local Claude-authored auto-memory organized as an index plus topic files. The Claude API exposes a developer-owned filesystem-style Memory Tool. Managed Agents add durable Memory Stores with access modes, version history, optimistic concurrency, sandbox mounts, self-hosted synchronization, and memory consolidation through Dreams.

The strongest reusable design rule is:

> **Store the irrecoverable residue; re-query authoritative reality.**

---

# 2. Evidence classes and precedence

## A — Confirmed Anthropic product fact

Explicitly documented in current Anthropic support, product, Claude Code, Claude Platform/API, or Managed Agent documentation.

## B — Strong implementation evidence

Observed in current captured Claude system instructions or client/export behavior that closely matches official functionality but is **not** an Anthropic-authenticated product contract.

## C — Architectural inference

A conclusion strongly supported by multiple A/B facts but not directly stated by Anthropic.

## U — Unknown

No sufficient current evidence.

When sources conflict, use this order:

1. newer feature-specific Anthropic API/product documentation;
2. current Anthropic general documentation;
3. current Claude Code / Platform / Managed Agent documentation;
4. older Anthropic documentation retained for history;
5. reproducible implementation evidence;
6. captured system prompts / client observations;
7. community speculation.

**Freshness controls.** The current API reference can expose a newer capability that an overview page has not yet incorporated.

---

# 3. The Claude ecosystem has multiple distinct “memory” layers

| Mechanism | Purpose | Persistence | Scope |
|---|---|---:|---|
| Consumer Claude Memory | durable user/project context as Topics/files | Yes | non-project or one Project |
| Past Chat Search | retrieve specific old conversations via RAG | history-dependent | non-project or one Project |
| Project Summary | compressed project continuity | persistent/derived | one Project |
| Project Knowledge | user-provided project files/knowledge | Yes | one Project |
| Project Knowledge RAG | retrieve relevant project knowledge | derived retrieval | one Project |
| User preferences/styles | personalization instructions distinct from memory | Yes | account/product scope |
| Claude Code `CLAUDE.md` | human-authored instructions | Yes | managed/user/project/local/path scope |
| Claude Code `.claude/rules/` | modular/path-scoped instructions | Yes | user/project/path scope |
| Claude Code auto-memory | Claude-authored learned context | Yes, local | repository/project identity |
| Claude Code subagent memory | agent-specific learned context | Yes | user/project/local agent scope |
| API Memory Tool | developer-owned persistent memory abstraction | developer-defined | application-defined |
| Managed Agent Memory Stores | persistent text documents mounted to agents | Yes | workspace/store/session attachment |
| Managed Memory Versions | immutable audit/history of mutations | Yes, retention-bounded | Memory Store |
| Dreams | offline/asynchronous memory consolidation | produces/writes durable memory | Managed Agents |
| Context compaction | compress active conversational state | workflow/session | active run |
| Monthly Recap | reflective analysis of recent usage/history | derived | consumer account |

These mechanisms should not be conflated.

---

# 4. Consumer Claude Memory — current product source of truth

## 4.1 Product evolution

**A. Legacy design.** The older consumer system periodically synthesized a memory summary from conversation history. Remaining legacy Team/Enterprise organizations may still temporarily use that behavior, with synthesis documented at roughly a 24-hour cadence.

**A. July 10, 2026.** Anthropic changed normal consumer memory from a synthesized summary to **individual categorized entries/topics that Claude reads and updates during conversations**.

**A. August 25, 2026.** Anthropic expanded the current system so Claude Chat and **cloud Cowork** share memory, Topics are directly editable, and sensitive-memory controls are exposed.

**A. Migration.** A small number of Team/Enterprise organizations may still be on legacy memory. Migrated users can export legacy memory through **September 9, 2026** if something was lost in migration.

Sources:

- https://support.claude.com/en/articles/12138966-release-notes
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

## 4.2 Modern consumer memory is topic/file-oriented

**A.** Anthropic describes everything Claude remembers as a **list of short files under Topics** in Memory settings. Users can inspect, edit, and delete these individually.

```text
Memory
 ├── Topic/file
 ├── Topic/file
 ├── Topic/file
 └── ...
```

This establishes a product-facing file/topic abstraction. It does **not** prove that Anthropic physically stores Markdown files on a filesystem.

**Physical storage backend: U.**

Sources:

- https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it
- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context

## 4.3 Memory is written while you chat

**A.** The current system adds useful memory during ordinary conversation rather than waiting for a daily synthesis. Users can also explicitly say things equivalent to:

```text
Remember X.
Update what you remember about X.
Forget X.
```

Topic edits apply to future conversations.

## 4.4 What Claude is documented to remember

**A.** Anthropic lists durable collaboration-oriented context including:

- professional role and context;
- projects and ongoing work;
- important people;
- important places;
- communication preferences;
- working style;
- technical preferences;
- coding preferences;
- project details;
- durable decisions/constraints useful later.

**A/C.** Mentioning something does not guarantee admission to memory. The exact production admission/ranking model is unknown.

## 4.5 Current availability

**A.** Current generated memory is documented across web, Desktop, and supported mobile apps.

- Free: enabled by default.
- Pro: enabled by default.
- Max: enabled by default.
- Team: organization-level availability off until an Owner enables it.
- Enterprise: organization-level availability off until an Owner enables it.

UI details can vary by app version.

## 4.6 Chat and cloud Cowork share consumer memory

**A.** Chat and **cloud Cowork** share current consumer memory bidirectionally.

```text
                shared consumer memory
               ↗                      ↖
          Claude Chat              cloud Cowork
```

**A.** Local Cowork does not participate in the same cloud memory system.

**U.** There is no official evidence that Claude Code auto-memory is synchronized with this consumer store.

## 4.7 Projects are isolated memory namespaces

**A.** Each Project has:

- its own memory space;
- a dedicated project summary.

Project memory is isolated from ordinary non-project memory and from other Projects. Moving a conversation into or out of a Project changes the relevant memory/search domain.

Sources:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects

## 4.8 Project memory is not Project Knowledge

**A.** A Project can simultaneously expose:

```text
current conversation
+ project instructions
+ project memory
+ dedicated project summary
+ project conversation history
+ project-scoped past-chat search
+ Project Knowledge/files
+ Project Knowledge RAG
```

**A.** Project Knowledge RAG is separate from generated memory. It activates automatically as project knowledge approaches/exceeds direct-context capacity and can fall back to direct context if the knowledge set shrinks. Anthropic says it can expand practical project knowledge capacity by up to approximately **10×**.

**A. Current Project RAG plans:** Free, Pro, Max, Team, Enterprise.

Source:

- https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects

## 4.9 Past Chat Search is a separate RAG system

**A.** Historical chat search is separate from generated memory. Anthropic explicitly describes it as RAG and surfaces it as a tool call when used.

Scope:

```text
non-project conversation
  → searches non-project history

Project A conversation
  → searches Project A history only
```

This creates two different “remembering” paths:

```text
Generated memory
“I know this durable thing.”

Historical retrieval
“I found the old conversation where this happened.”
```

## 4.10 Past Chat Search availability and controls

**A.** Current documented plans: Pro, Max, Team, Enterprise. Available on web, Desktop, and Mobile.

**A.** Once rolled out it is enabled by default but has a separate **Search and reference chats** control, independently configurable from generated memory.

**A.** Incognito conversations are excluded.

**A.** Enterprise organizations using customer-managed encryption keys currently cannot use Past Chat Search because the feature cannot search encrypted conversation contents in that configuration.

## 4.11 Pause Memory semantics

**A.** Pausing memory:

- retains existing memory;
- stops Claude from reading it;
- stops Claude from creating new memory;
- leaves stored sensitive memory present but inactive;
- does not retroactively turn pause-period conversations into memory after unpausing.

```text
Pause = stop READ + stop WRITE
```

## 4.12 Reset Memory semantics

**A.** Reset Memory deletes generated memory including Project memory and is documented as irreversible. Re-enabling memory starts from a new memory state.

## 4.13 Chat deletion and memory deletion are separate

**A.** In the modern topic-based system, deleting/expiring the source conversation does **not** automatically delete a generated memory Topic already derived from it. The Topic must be removed separately if the user wants it gone.

```text
source chat
  ↓
memory Topic created
  ↓
source chat deleted
  ↓
Topic may remain
```

## 4.14 Consumer memory data lifecycle

**A.** Current documentation establishes:

- memory follows applicable account/organization retention rules;
- generated memory data is included in account data exports;
- Team/Enterprise organization retention rules apply;
- Enterprise memory entries are encrypted at rest;
- deletion of the source chat does not itself delete an already-generated Topic;
- organization data policy can affect Incognito/export visibility.

## 4.15 Topics are human-editable state

**A.** Settings → Memory → Topics lets a user inspect, edit, and delete generated memories. Users can also request remember/change/forget operations directly in chat.

**A.** Historical chat search can surface links/citations to original conversations; those are separate from Topics.

## 4.16 Importing memory

**A.** Claude can import memory/profile material exported from another AI provider and extract useful information into individual memory entries.

Current documented import availability:

- Free;
- Pro;
- Max;
- Team;
- web;
- Claude Desktop.

**A.** Import is experimental and may omit material.

**A.** The importer is work-oriented; Claude may discard personal information that does not fit its collaboration/work memory focus.

Source:

- https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude

## 4.17 Exporting memory

**A.** Users can export memory and can ask Claude to write out the memories it sees verbatim/exactly as stored in the product-facing representation.

**C.** This makes memory increasingly portable user state rather than a completely opaque internal profile.

## 4.18 Sensitive topics are excluded by default

**A.** Default auto-memory excludes many sensitive categories including areas such as:

- health;
- race;
- ethnicity;
- religious beliefs;
- political beliefs;
- gender identity;
- similar sensitive attributes.

Users can opt into **Include sensitive topics in memory** where available.

## 4.19 Sensitive-memory opt-in semantics

**A.** Enabling sensitive memory is prospective, not retroactive.

Current documented behavior includes:

- eligible future sensitive material can be saved;
- users receive a review notice when a sensitive memory is saved;
- the first relevant decline can trigger a one-time explanatory notice;
- current mobile sensitive-save notices require a sufficiently current app version; an older unsupported app version does not silently save the sensitive item;
- disabling sensitive memory removes sensitive items stored through that mechanism.

## 4.20 Categories Anthropic publicly says memory will not save

**A.** Public never-save examples include:

- government identification numbers;
- financial account numbers;
- criminal history;
- immigration status.

Applicable policy restrictions also constrain memory behavior.

Sources:

- https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it

## 4.21 Incognito is a hard normal-memory/history boundary

**A.** Incognito chats:

- do not read ordinary Claude memory;
- do not create ordinary Claude memory;
- are not saved in ordinary chat history;
- are excluded from Past Chat Search;
- are excluded from Monthly Recap;
- are not used for model training under Anthropic's documented Incognito behavior.

**A.** Incognito can still receive other personalization such as profile/custom styles/preferences.

Therefore:

```text
memory ≠ all personalization
```

Source:

- https://support.claude.com/en/articles/12260368-use-incognito-chats

## 4.22 Incognito retention and organization visibility

**A.** Incognito is not zero-retention:

- default retention is approximately 30 days for safety;
- Enterprise/custom organization policies can retain longer;
- Team/Enterprise Incognito can appear in organization exports/Compliance API flows as documented;
- Incognito runs outside Projects;
- once closed, it is not simply saved/converted into a regular conversation under the current product model.

## 4.23 Team/Enterprise governance

**A.** In the new memory experience:

- organization-level generated memory is off by default;
- an Owner/Primary Owner enables availability;
- sensitive-memory permission is a separate organization-level control;
- individual users manage their own generated memories after enablement;
- owners cannot inspect/edit an individual's memory through the memory UI;
- disabling organization memory immediately and permanently deletes generated memory entries for users in that organization;
- the feature is unavailable for some HIPAA configurations, public-sector arrangements, and custom-retention organizations.

## 4.24 Team/Enterprise security/export/audit details

**A.** Current documented behavior includes:

- memory entries encrypted at rest;
- organization retention/export rules apply;
- Incognito may be included in organization exports despite not appearing in user-visible ordinary history;
- organization-level memory setting changes can appear in audit logs;
- ordinary conversation access logging applies;
- individual member edits to personal Topics are not documented as separate organization audit events.

## 4.25 Remaining legacy Team/Enterprise behavior

**A — historical/migration only.** For the small legacy cohort, old behavior differs:

- synthesis approximately every 24 hours;
- standalone/non-project synthesis separate from Project memory;
- no current cloud-Cowork shared-memory behavior;
- deleting conversations changes what can enter the next synthesis;
- direct legacy-summary edits can apply immediately;
- organization-control defaults differ from the new experience.

These facts must not be generalized to modern topic-based memory.

## 4.26 Monthly Recap is memory-adjacent, not memory

**A.** Settings → Reflect → Monthly Recap is a derived reflection feature that requires memory to be enabled but is not the persistent memory store.

Current documented availability is Free/Pro/Max on web/Desktop for viewing; supported mobile activity can contribute even when the recap page itself is not available on mobile. Team/Enterprise are not currently part of the consumer recap experience.

Source:

- https://support.claude.com/en/articles/15672559-see-your-monthly-recap

## 4.27 Monthly Recap inputs/exclusions

**A.** Recap can report:

- opening summary;
- conversation counts;
- most active day;
- peak hour;
- daily activity chart;
- topic distribution;
- AI-fluency observations such as Delegation, Description, Discernment, and Diligence.

**A. Excluded:** Incognito, Health integration chats, Cowork, Claude Code.

**A.** Raw connected Gmail/Google Drive content is not directly included in the recap dataset; Claude-authored summaries/comments that appeared in conversations can be reflected.

**A.** Sensitive/distress-related subjects are not used as recap-leading categories/count breakdowns as documented.

**A.** Recap is generated/refreshed when the user visits Reflect rather than existing as a continuously visible memory record.

## 4.28 Consumer semantic memory + episodic retrieval

**C terminology.** The current architecture is usefully modeled as:

- **semantic/adaptive memory:** durable compressed context (role, preferences, projects, people, decisions);
- **episodic retrieval:** exact prior conversations recovered through Past Chat Search.

Anthropic does not require these cognitive-science terms; they are analytical labels.

---

# 5. Consumer implementation evidence — captured Fable 5.1 / Opus 5

Everything in this section is **Class B unless stated otherwise**.

Primary capture source:

- https://github.com/elder-plinius/CL4R1T4S/blob/93b0ae6fb503db6642e58f9d6352db973a900cdc/ANTHROPIC/Claude-Fable-5.1.md

Same-source corroboration:

- https://github.com/elder-plinius/CL4R1T4S/blob/93b0ae6fb503db6642e58f9d6352db973a900cdc/ANTHROPIC/OPUS-5.md

Because both files come through the same external capture repository/pipeline, the second is **not** independent Anthropic confirmation.

## 5.1 Model-facing memory filesystem

The capture describes a persistent component approximately named:

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

This strongly suggests a path-addressed memory abstraction. It does not prove physical filesystem storage.

## 5.2 Apparent file taxonomy

```text
/profile.md
/preferences.md
/topics/...
/areas/...
/people/...
```

Observed semantics:

- `/profile.md`: stable identity/context;
- `/preferences.md`: how Claude should interact/respond;
- `/topics/`: recurring interests, habits, routines, broad recurring domains;
- `/areas/`: ongoing projects, responsibilities, decisions, work domains;
- `/people/`: persistent context about recurring people/relationships.

## 5.3 Exact captured profile-admission test

The capture uses a stability test roughly equivalent to:

> Would this still be true in three months?

Temporary/deadline/“currently” material is routed to `/areas/` or `/topics/` rather than stable profile identity.

The capture also instructs Claude to keep `/profile.md` **under 300 words**.

That numeric target is B-level implementation evidence, not a public product contract.

## 5.4 Sparse retrieval through `<memory_listing>`

The capture describes a compact `<memory_listing>` containing routing-level information such as:

- file path;
- one-line description;
- aliases where applicable;
- sources/provenance metadata.

The one-line description is a **routing hint, not a substitute for reading the file**. When the listing suggests relevance, Claude is instructed to read the detailed file before concluding it lacks the information.

Apparent design:

```text
cheap/always context
  profile
  preferences
  memory listing/index

on demand
  detailed topic/area/person files
```

## 5.5 Captured frontmatter/schema

Observed concepts include:

```yaml
name: canonical-slug
description: one-line routing description
sources:
  - chat
aliases:
  - alternate subject name
```

Observed semantics:

- canonical `name` corresponds to the path/subject identity;
- `name` should be unique across the memory set;
- `description` supports routing;
- `sources` preserves contributing surfaces;
- `aliases` are especially associated with `/areas/` and `/people/`.

## 5.6 Cross-memory links

The capture uses `[[name]]`-style links between memory subjects. Canonical unique names act as link targets.

**C analytical description:** the result resembles a **Markdown-shaped lightweight knowledge graph**.

## 5.7 Entity resolution

Aliases are intended to prevent one subject from fragmenting across several memories, e.g. `David`, `Dave`, `David from Crystal Tile`.

The capture says the **fact's domain determines the file**, not whichever file already exists/is open.

## 5.8 Post-turn background memory pass

The Fable 5.1 capture explicitly says durable filing occurs **automatically after each completed assistant turn** through a background pass that re-reads the finished exchange.

Probable flow:

```text
user message
  ↓
foreground Claude answer
  ↓
turn completes
  ↓
background memory pass
  ├─ reread exchange
  ├─ decide durability
  ├─ choose semantic subject/file
  ├─ read existing state
  ├─ reconcile/correct
  └─ persist update
```

**U:** exact background model/service/prompt.

## 5.9 Explicit remember/update/forget uses a foreground path

The capture instructs foreground Claude to directly perform explicit memory mutations rather than waiting for ordinary background filing.

The background process then avoids reprocessing the same turn in a way that would recreate/undo an explicit deletion.

## 5.10 Best-effort, not load-bearing

The capture treats automatic memory I/O as best-effort: a memory maintenance failure should not derail the user's primary task.

```text
canonical operational state = source systems
adaptive memory             = helpful context
```

## 5.11 Provenance/epistemic typing

The capture includes concepts resembling:

```text
[stated]
[observed]
[inferred]
```

Chat-side rules are conservative about user memory: direct user-established facts are eligible; Claude's own guesses/recommendations are not silently promoted to autobiographical truth.

## 5.12 Model advice, web/tool output, and hearsay are not automatically user memory

A Claude recommendation is not a user fact merely because Claude generated it. A web/connector result is not automatically autobiographical memory. If the user explicitly confirms a decision/fact, that confirmation can become user-established state.

## 5.13 Admission uses durability/repetition

Stable facts can be admitted quickly. Fleeting execution state should stay out. Casual one-off tastes/hobbies may require recurrence or meaningful engagement before becoming worth retaining.

**U:** exact scoring formula.

## 5.14 Cross-surface provenance preservation

The capture includes `sources` metadata. Chat should preserve existing source values and add `chat` when it contributes rather than overwriting provenance.

This aligns with official Chat ↔ cloud Cowork shared memory, but the exact metadata is still B-level.

## 5.15 Optimistic concurrency

Captured mutation operations use an `if_version`-style precondition.

Protocol:

```text
read current file/version
  ↓
mutate using if_version
  ↓
conflict if another writer changed it
  ↓
re-read latest
merge/reconcile
retry
```

The capture explicitly instructs **read before update/delete** to obtain the current version.

## 5.16 Fine-grained deletion

The capture distinguishes:

- deleting a whole subject/file;
- removing/replacing one exact fact/line;
- correction/supersession;
- explicit forgetting.

If a dependent fact existed solely because of a forgotten source fact, the captured rules say the dependent fact should also be removed.

**C:** this approximates provenance-aware cascading deletion even though no formal dependency graph is publicly documented.

## 5.17 Correction is different from forgetting

A correction may preserve chronology (`now Infrastructure; previously Search`). An explicit “forget that I ever worked on Search” should remove the historical fact instead of preserving it as chronology.

## 5.18 Capacity management

The captured memory tool provides capacity/size information. Under pressure Claude is told to:

- consolidate overlapping facts;
- prune stale low-value detail;
- split overly broad subjects;
- summarize repetitive history;
- remember pointers to authoritative external systems instead of copying changing data.

**U:** exact consumer per-file/account limits.

## 5.19 Broader captured sensitive-data policy

The captured internal rules are broader/more nuanced than the public help page and include or discuss categories such as:

- race/color/ethnicity/caste;
- religion;
- sexual orientation;
- gender identity;
- immigration status;
- disability/serious illness;
- union membership;
- socioeconomic/financial details;
- medical conditions/diagnoses/labs/genetics/mental-health/therapy/addiction details;
- criminal/victimization history;
- sexual-history information;
- restrictions on inferring health information.

Because this is prompt-capture evidence, it must not be treated as a stable public product guarantee or used to override current official documentation.

## 5.20 Stored memory is untrusted context

The capture warns against allowing stored memory to override higher-priority/current explicit instructions. Official Managed Agent docs independently warn that writable memory can persist prompt injection.

**C conclusion:** persistent memory is a security boundary.

---

# 6. Claude Code — explicit instructions (`CLAUDE.md` and rules)

Primary source:

- https://code.claude.com/docs/en/memory

## 6.1 Two complementary persistence mechanisms

**A.** Each Claude Code session starts with fresh context. Persistent continuity comes from:

- `CLAUDE.md` / rules: written by the human;
- auto-memory: written by Claude.

Both are model context, **not enforced configuration**. Anthropic explicitly recommends `PreToolUse` hooks for deterministic blocking.

## 6.2 `CLAUDE.md` scopes

**A. Managed policy examples:**

- macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`
- Linux/WSL: `/etc/claude-code/CLAUDE.md`
- Windows: `C:\Program Files\ClaudeCode\CLAUDE.md`

**A. User:** `~/.claude/CLAUDE.md`

**A. Project:** `./CLAUDE.md` or `./.claude/CLAUDE.md`

**A. Local project:** `./CLAUDE.local.md`

Nested/path-specific instructions can load when relevant files/subtrees are accessed.

## 6.3 Loading behavior and precedence

**A.** Applicable instruction files are combined/concatenated rather than acting as a simple “last value wins” config system. Ancestor files load at startup; nested/path-scoped sources can lazy-load later. At the same directory level, local project guidance applies after ordinary project guidance.

**A.** Ordinary HTML comments are stripped before context injection except comments inside code blocks; reading the file explicitly with tools still shows comments.

## 6.4 Size guidance

**A.** Anthropic recommends targeting **under 200 lines** per `CLAUDE.md` for adherence/context economy.

**A.** Claude Code loads a `CLAUDE.md` file up to approximately **4 MiB** and skips a larger file.

## 6.5 `/init`

**A.** `/init` can create a starting project `CLAUDE.md`; if one exists it suggests improvements rather than blindly overwriting it.

**A.** With `CLAUDE_CODE_NEW_INIT=1`, supported versions use a more interactive setup flow that can propose `CLAUDE.md`, skills, and hooks, explore with a subagent, ask follow-up questions, and present a reviewable proposal.

## 6.6 Imports with `@path`

**A.** `CLAUDE.md` supports relative and absolute `@path` imports.

Current documented details include:

- relative paths resolve from the containing instruction file;
- recursive imports are bounded to **four hops**;
- import syntax inside code spans/fenced code is ignored;
- external-project imports can require first-use trust approval;
- `CLAUDE.local.md` can serve as private/worktree-local guidance when excluded from version control;
- user-scope trust/security rules differ from untrusted project imports, with additional Cowork/desktop restrictions documented for external/symlinked content.

## 6.7 `AGENTS.md` is not the native filename

**A.** Claude Code natively reads `CLAUDE.md`, not `AGENTS.md`. Users can import/symlink/copy other rule files, and newer initialization/import workflows can migrate supported agent-rule formats.

## 6.8 `.claude/rules/`

**A.** Modular rules can live under `.claude/rules/` (and corresponding user-level rules).

Rules can be:

- unscoped/startup-loaded;
- path-scoped with YAML frontmatter/globs;
- recursive;
- symlinked, with circular symlinks handled.

**A. Path-pattern safety limits:** a rule's `paths` list shares a brace-expansion budget of **1,000 expanded patterns and 4 MiB**. Patterns without braces do not count. A pattern that would exceed the budget remains unexpanded/literal.

**A historical bug notes:** before v2.1.217 excessive brace groups could stall/crash startup; before v2.1.207 one invalid bracket expression could break Read evaluation rather than simply matching nothing.

## 6.9 Additional directories

**A.** `--add-dir` gives filesystem access but does **not** automatically load that directory's instructions.

To opt in:

```text
CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1
```

This can load `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/*.md`, and `CLAUDE.local.md` from added directories, with setting-source rules affecting local files.

## 6.10 Managed organization instructions and exclusions

**A.** Managed policy instructions exist separately from ordinary project/user files. Current settings can exclude selected ordinary `CLAUDE.md` paths/globs, while managed-policy instruction sources cannot simply be removed by lower-trust project settings.

## 6.11 Context, not deterministic law

**A.** Claude Code documentation explicitly says instruction files are context, not enforced configuration. For deterministic behavior:

```text
behavioral guidance      → CLAUDE.md / rules
hard tool/action policy  → settings / permissions / hooks
system-level additions   → --append-system-prompt
verification             → tests / inspection
```

## 6.12 Diagnostics

**A.** `/context` shows which instruction/memory sources actually loaded into the active context.

**A.** The `InstructionsLoaded` hook can log exactly which instruction files loaded, when, and why.

## 6.13 `/doctor` trimming

**A.** Current `/doctor` can propose trimming checked-in `CLAUDE.md` by removing code-derivable directory/dependency/architecture material while preserving pitfalls, rationale, and conventions that differ from defaults. Current docs identify this trim check as requiring **v2.1.206+**.

## 6.14 Compaction

**A.** Project-root `CLAUDE.md` survives `/compact` by being reread/reinjected. Nested/path-scoped rules reload when matching files are subsequently accessed.

```text
compaction ≠ persistent instruction deletion
```

---

# 7. Claude Code auto-memory

Primary source:

- https://code.claude.com/docs/en/memory

## 7.1 Official memory types

**A.** Current types:

- `user` — role, expertise, working preferences;
- `feedback` — corrections and user-confirmed approaches;
- `project` — ongoing work/decisions/deadlines not reliably recoverable from repo/git;
- `reference` — where authoritative external information can be found.

## 7.2 Auto-memory avoids reconstructible facts

**A.** Claude Code explicitly avoids wasting auto-memory on architecture/file paths/debug conclusions recoverable from the codebase or information already present in `CLAUDE.md`.

> **Persist what would otherwise be lost or expensive to reconstruct.**

## 7.3 Enabled by default; disable controls

**A.** Auto-memory is on by default in supported versions.

Documented controls include:

- `/memory` toggle;
- `autoMemoryEnabled` in settings;
- project/user settings as supported;
- `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`.

The modern feature was introduced in the **2.1.59+** line; later subfeatures require newer versions.

## 7.4 Default storage and scope

```text
~/.claude/projects/<project>/memory/
├── MEMORY.md
├── user_role.md
├── feedback_testing.md
└── ...
```

**A.** A git repository defines the project identity; worktrees/subdirectories share the same auto-memory directory. Outside git, the project root is used.

**A.** Auto-memory is machine-local by default and is not automatically synchronized across machines/cloud environments.

## 7.5 `CLAUDE_CODE_PROJECT_DIR_NAME`

**A.** In **v2.1.234+**, setting `CLAUDE_CODE_PROJECT_DIR_NAME` beside `CLAUDE_CONFIG_DIR` can intentionally force the project-directory identity under the config directory, allowing environments launched with that config to share one auto-memory directory even across repositories.

## 7.6 `autoMemoryDirectory`

**A. Current correction.** `autoMemoryDirectory` can be read from **user, project, local, policy, or `--settings`** scopes.

- value must be absolute or `~/`-relative;
- project/local values are subject to the same workspace-trust rule used for hooks/settings.

This corrects stale claims that project/local values are categorically rejected.

## 7.7 `MEMORY.md` is the startup routing index

**A.** Auto-memory uses:

```text
MEMORY.md          # concise index, one line per memory
specific topic files
```

At startup, Claude Code loads only the first:

```text
200 lines
OR
25 KB
whichever comes first
```

Topic files such as `user_role.md`/`feedback_testing.md` are **not** startup-loaded; Claude opens them on demand.

## 7.8 Index overflow behavior

**A.** The 200-line/25-KB limit applies to `MEMORY.md`, not the whole memory directory.

- near the limit, Claude gets a reminder to compact/reorganize the index;
- an over-limit write still succeeds;
- Claude Code then returns an error/request to rewrite the index;
- content past the threshold is dropped from the next startup load;
- topic files can remain larger because they are read selectively.

## 7.9 Visible save/recall activity

**A.** Claude Code shows indicators such as:

```text
Saved 2 memories
Recalled 2 memories
```

when actively writing/reading auto-memory.

## 7.10 `modified` frontmatter timestamp

**A.** In **v2.1.214+**, when Claude writes a memory file that already has YAML frontmatter, Claude Code records/updates a `modified` ISO-8601 timestamp.

**A.** It does not add frontmatter merely to create this field if the file has none.

## 7.11 Transcript cleanup is separate

**A.** `cleanupPeriodDays` transcript pruning excludes auto-memory files. `MEMORY.md` and topic files survive until Claude/user edits or deletes them.

## 7.12 `/memory` vs `/context`

**A.** `/memory` lists user/project instruction and memory locations, including some nonexistent files that can be created, toggles auto-memory, and can open the memory directory/file in an editor.

**A.** `/context` tells you what actually loaded into the current model context.

```text
file exists/configured ≠ file loaded
```

**A historical UI note:** before v2.1.216, `/memory` could wait for a GUI editor to close before returning.

## 7.13 Natural-language routing

**A/C.** Current docs distinguish intents like:

```text
“Remember X”          → auto-memory
“Add X to CLAUDE.md”  → explicit human-authored instructions
```

This is a useful separation between adaptive learned state and explicit policy/guidance.

---

# 8. Claude Code subagent persistent memory

Primary source:

- https://code.claude.com/docs/en/sub-agents

## 8.1 Scopes and locations

**A.** A subagent can request:

| Scope | Location | Intended use |
|---|---|---|
| `user` | `~/.claude/agent-memory/<agent-name>/` | cross-project agent learning |
| `project` | `.claude/agent-memory/<agent-name>/` | project-specific/shareable |
| `local` | `.claude/agent-memory-local/<agent-name>/` | project-specific/private |

## 8.2 Global enablement still controls it

**A.** If auto-memory is disabled using `autoMemoryEnabled` or `CLAUDE_CODE_DISABLE_AUTO_MEMORY`, a subagent `memory:` declaration has no effect; the agent launches without ordinary memory instructions/tool access for that feature.

## 8.3 Startup behavior

**A.** When enabled, subagent memory gets memory instructions and its own `MEMORY.md` routing context following the same general startup-index pattern, with file tools needed to manage that memory available as documented.

## 8.4 Main-agent auto-memory is not automatically inherited

**A.** An ordinary non-fork subagent does **not** automatically receive the main conversation's auto-memory directory/content.

**A.** A **forked** subagent inherits parent conversational/system context as part of fork semantics, which is different from sharing the same persistent auto-memory directory.

## 8.5 Resumption ≠ persistent memory

**A/C.** Subagent session/conversation resumption can preserve conversational history. That remains conceptually distinct from its user/project/local persistent memory files.

---

# 9. Claude API Memory Tool

Primary source:

- https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

## 9.1 Client-side persistent abstraction

**A.** Anthropic exposes a logical `/memories` filesystem-style tool, but the developer implements storage. Possible backends include filesystem, database, object storage, encrypted service, etc.

A model-facing file abstraction does **not** prove a physical filesystem backend.

## 9.2 Tool declaration and model support

**A.** Current documentation uses approximately:

```json
{"type":"memory_20250818","name":"memory"}
```

No custom input schema is supplied by the caller.

**A.** The feature is documented for supported Claude 4+ models.

## 9.3 SDK helpers

**A.** Current SDK documentation includes helpers such as:

- Python/C#: `BetaAbstractMemoryTool`;
- TypeScript: `betaMemoryTool`;
- Java: `BetaMemoryToolHandler`;
- Python/TypeScript local filesystem: `BetaLocalFilesystemMemoryTool`;
- Go/Ruby: implement the loop directly under current docs;
- PHP: `BetaRunnableTool`.

SDK names are version-sensitive and should be verified before implementation.

## 9.4 Command set

**A.** Model-facing commands include:

```text
view
create
str_replace
insert
delete
rename
```

## 9.5 `view` details

**A.** Reference behavior includes:

- directory listing up to a bounded depth (the reference helper lists up to two levels);
- hidden paths/`node_modules` excluded by the reference helper;
- text output line-numbered from 1;
- line numbers formatted in a fixed-width field with tab separator;
- files with more than **999,999 lines** should return a line-limit error;
- `view_range` supports selected ranges and `-1` semantics as documented;
- the tool description supports image views for `.jpg`, `.jpeg`, `.png`;
- long text views are truncated at approximately **16,000 characters**, prompting ranged follow-up reads.

## 9.6 `create`

**A.** Reference helper behavior errors if the file already exists. The tool description itself says “creates or overwrites,” so Anthropic explicitly treats overwrite-vs-error as an implementation choice; callers should not infer one universal backend semantic from the helper.

## 9.7 `str_replace`

**A.** `old_str` must resolve uniquely; missing/duplicate matches produce errors. `new_str` is optional; omission deletes the matched text.

## 9.8 `insert`

**A.** Inserts after a specified line; `0` means the beginning. Valid bounds are tied to current file length as documented.

## 9.9 `delete`

**A.** Can delete files/directories recursively under the memory root but should not permit deletion of the logical `/memories` root itself.

## 9.10 `rename`

**A.** Reference behavior does not overwrite an existing destination and does not rename the logical memory root.

## 9.11 Memory-first protocol

**A.** Anthropic's built-in system/tool guidance tells Claude to inspect the memory directory before work so relevant durable state can be recovered after interruption/context reset.

The intended long-running workflow is roughly:

```text
start
  ↓
inspect durable memory
  ↓
work one bounded objective
  ↓
record progress/state
  ↓
verify
  ↓
future session resumes from durable state
```

## 9.12 Error handling

**A.** Memory-tool implementation errors should be returned through tool results with an error indicator (`is_error`) so Claude can recover rather than silently assume a mutation succeeded.

## 9.13 Just-in-time retrieval

**A.** The Memory Tool is a context-engineering primitive: keep durable state external and read only what the current request needs.

## 9.14 Memory + compaction

**A/C.** Anthropic recommends combining persistent memory with context editing/server-side compaction:

```text
active context       = temporary working cognition
compaction summary   = compressed current-run continuity
persistent memory    = cross-session durable state
source systems       = canonical reality
```

Persistent memory survives context reset/compaction because it exists outside the active window.

## 9.15 Security requirements

**A.** Anthropic recommends:

- canonicalizing paths;
- restricting operations to the memory root;
- rejecting `../`, `..\`, and encoded traversal variants;
- file-size limits;
- paging large reads/listings;
- sensitive-data validation;
- appropriate encryption/access control;
- expiration/removal of stale memory.

## 9.16 Expiration is explicitly recommended

**A/C.** Anthropic recommends periodically removing old/unaccessed memory where appropriate.

> **Forgetting/garbage collection is a system-health feature, not only a privacy action.**

---

# 10. Managed Agent Memory Stores

Primary source:

- https://platform.claude.com/docs/en/managed-agents/memory

## 10.1 Fresh sessions + durable stores

**A.** Managed Agent sessions start with fresh context. Memory Stores provide durable preferences, project conventions, past mistakes, and domain context across sessions.

## 10.2 Beta headers

**A.** Current Managed Agents requests use:

```text
managed-agents-2026-04-01
```

while Memory Store endpoints use:

```text
agent-memory-2026-07-22
```

**A.** Do **not** combine both on a Memory Store request; current docs say that returns HTTP 400. Session endpoints—including attaching a Memory Store—still use the Managed Agents header. The list-memories endpoint behaves consistently under either accepted path as documented.

## 10.3 Store model

**A.** A Memory Store is a workspace-scoped collection of text documents. It has a `memstore_...` identifier plus human-readable name/description. The description is supplied to the agent as context about the store.

## 10.4 Current hard limits

**A.** Current official limits:

- **100 kB (~25K tokens) per memory document**;
- **10,000 memories per Memory Store**;
- **8 Memory Stores per session**;
- per-attachment instructions capped at **4,096 characters**.

The previous v1.0 “2,000 memories/store” figure was stale and is superseded.

## 10.5 Attachments are fixed at session creation

**A.** Stores are attached in `resources[]` when the session is created. They cannot be freely added/removed from a running session under current docs.

Self-hosted environments accept Memory Store resources under the documented resource model.

## 10.6 Access modes

**A.** Attachments support:

```text
read_write   # default
read_only
```

Use read-only for reference/shared material when mutation is not needed.

## 10.7 Mount paths

**A.** Attached stores appear under `/mnt/memory/` using a sanitized display-name slug, but the exact path is returned as `mount_path` in the session resource. **Use the returned path rather than constructing it.**

**A.** The `/mnt/memory` parent is read-only/controlled; supported writes must target an actual writable store mount.

**A.** A note describing each mount—display name, mount path, access mode, description, attachment instructions—is automatically added to the agent system prompt.

## 10.8 Managed cloud vs self-hosted storage

**A.** Anthropic-managed sandboxes expose the store as durable mounted storage.

**A.** Self-hosted sandboxes do **not** receive a live remote mount. The environment worker downloads a local copy and synchronizes it with the durable store.

## 10.9 Self-hosted synchronization

**A.** Current docs describe:

- reconciliation after tool calls;
- at most one sync per interval, **15 seconds by default**;
- final sync when the session ends normally;
- another self-hosted session sees a change only after the relevant workers sync;
- writes outside real store directories under `/mnt/memory/` are not durable store state;
- read-only store `write`/`edit` mutations are refused and not uploaded.

Therefore self-hosted memory is an **eventually synchronized local copy**, not an instantly coherent distributed filesystem.

## 10.10 Event stream

**A.** Agent reads/writes to memory mounts appear as ordinary `agent.tool_use` / `agent.tool_result` events for the underlying file tools.

## 10.11 Listing semantics

**A.** Memory listing is returned in a stable server-defined order.

`path_prefix`:

- must end with `/`;
- is path-segment aware.

`depth`:

- omitted or `0` = whole subtree;
- `1` = immediate children;
- other values currently return 400.

## 10.12 Basic vs full projections

**A.** Current API reference supports `basic`/`full` views. Retrieve operations default to full content; list/create/update can default to basic metadata. Listing with `view=full` currently caps the page `limit` at 20.

## 10.13 Read/create/update/delete

**A.** Individual memory operations include:

- retrieve full content;
- create at path;
- update content;
- rename/move path;
- update content and path together;
- delete.

**A.** Create does not silently overwrite an existing memory path; use update for an existing memory.

**A.** At the 10,000-memory store limit, new-path creates/agent writes fail while existing memories remain readable/editable.

## 10.14 Optimistic concurrency

**A.** `content_sha256` can be supplied as an update precondition.

```text
read content + hash A
  ↓
concurrent writer changes memory
  ↓
update expecting A
  ↓
409 precondition failure
  ↓
re-read latest
merge
retry
```

**A.** Current reference adds a useful idempotency nuance: if the precondition is stale but stored content/path already exactly matches the requested final state, the server can return 200 instead of 409.

## 10.15 Every mutation creates immutable version history

**A.** Every memory mutation creates a `memver_...` version record.

Versions support audit of:

- operation;
- actor;
- timestamp;
- historical content.

Actors can include agent sessions, API keys, Console users, and service accounts in the documented model.

## 10.16 Version retention

**A.** Versions are store-level audit records and survive deletion of the live memory while retained.

Current documented retention:

- versions are retained for **30 days after they are written**;
- recent versions of a live memory are kept regardless of age;
- infrequently changed live memories can therefore retain history beyond 30 days;
- older history beyond guarantees should be exported if long-term audit is required.

## 10.17 No dedicated restore endpoint

**A.** To roll back, retrieve the desired retained historical version and write its content back with update (or recreate if the live memory was deleted and the version is still retained).

## 10.18 Redaction

**A.** Historical-version redaction removes content while preserving the audit record of who/what/when.

**A.** The current head of a live memory cannot be redacted directly; write a new head or delete the live memory first, then redact the old historical version.

## 10.19 Store lifecycle

**A.** Store operations include create/retrieve/update/list/archive/delete.

**A.** Archived stores are excluded from ordinary lists unless requested.

**A.** Archiving:

- makes a store read-only;
- prevents attachment to new sessions;
- is **one-way under current docs; there is no unarchive**.

**A.** Deleting a store permanently removes the store with all its memories and version history.

## 10.20 Security: persistent prompt injection

**A.** Anthropic explicitly warns that untrusted prompt/web/tool content can cause an agent with write access to persist malicious instructions, which future sessions may then read as memory.

Recommended pattern:

```text
trusted/learning state       → writable
shared reference/standards   → read-only where possible
untrusted transient content  → do not blindly persist
```

## 10.21 Best-practice store architecture

**A/C.** Anthropic recommends focused stores rather than one global dump. A useful pattern is:

- per-user writable memory;
- shared read-only reference memory;
- project-scoped memory;
- new writable store after an old store grows too large, optionally keeping the old one read-only;
- prune/delete stale memory or run Dreams before capacity becomes a problem.

---

# 11. Dreams — current consolidated source of truth

Primary overview:

- https://platform.claude.com/docs/en/managed-agents/dreams

Current API reference:

- https://platform.claude.com/docs/en/api/http/beta/dreams/create
- https://platform.claude.com/docs/en/api/typescript/beta/dreams

## 11.1 Research preview and API volatility

**A.** Dreams are a research-preview capability and can require access approval.

**A.** Dream endpoints use:

```text
dreaming-2026-04-21
```

The Managed Agents beta header alone does not grant Dream endpoint access. Examples can send both where needed.

**A.** The API reference explicitly says Dreams request/response shapes are volatile in research preview and may change without the deprecation guarantees of GA endpoints.

## 11.2 Why Dreams exist

**A.** Anthropic says incremental agent memory accumulates:

- duplicates;
- contradictions;
- stale entries;
- fragmented organization.

Dreams let Claude perform a broader consolidation pass over accumulated memory plus historical sessions.

## 11.3 Inputs

**A.** A Dream uses:

- one input Memory Store;
- **1–100** historical Managed Agent sessions.

If only sessions exist, create an empty Memory Store first and use it as the memory-store input.

## 11.4 Default output behavior: `create_new`

**A.** The overview page describes the default behavior:

```text
input Memory Store
+ sessions
  ↓
clone input into NEW output store
  ↓
write consolidated memory to output
```

With default:

```json
{"type":"create_new"}
```

the input store is **not mutated**.

The output-store ID appears in `outputs[]` after the pipeline reaches the relevant stage; a running Dream can briefly have an empty `outputs[]`.

## 11.5 Current API also supports `update_existing`

**A — critical current reconciliation.** The newer/current API reference exposes:

```json
{
  "type": "update_existing",
  "memory_store_id": "..."
}
```

In the current EAP, the target must be the Dream job's own input Memory Store, so the Dream **consolidates that store in place**.

Therefore the correct current statement is:

```text
DEFAULT create_new
  → input not mutated; new output store

OPTIONAL update_existing (EAP)
  → input store is also destination; consolidation occurs in place
```

This supersedes the v1.1 categorical claim that a Dream can never mutate its input.

**A.** Current API reference also documents a target-store-held conflict if another in-place Dream is still pending/running or final writes are still landing; the target must no longer be held before retrying.

## 11.6 Model configuration

**A.** `model` can be supplied as a model ID string or a config object including:

```text
id
speed: standard | fast
```

Not every model supports `fast`; invalid model/speed combinations fail at create time, and fast mode uses premium pricing where supported.

Current research-preview supported models include:

- `claude-opus-5`;
- `claude-fable-5`;
- `claude-opus-4-8`;
- `claude-opus-4-7`;
- `claude-sonnet-5`;
- `claude-sonnet-4-6`.

This list is preview-state and may change.

## 11.7 Optional instructions

**A.** Dream instructions are optional and currently limited to **1–4096 characters**.

They steer high-level synthesis: what to emphasize, preserve, merge/drop, and how to organize output. They are not a precision line-editing interface; targeted edits belong in the Memory Store API.

## 11.8 Asynchronous lifecycle

**A.** Dream statuses:

```text
pending
running
completed
failed
canceled
```

Dream objects include inputs, outputs, model config, instructions, processing `session_id`, timestamps, usage, and error detail.

## 11.9 Processing session

**A.** The underlying processing session can be streamed/observed. The Dream's processing session is archived rather than deleted at terminal state, preserving the transcript for later inspection.

## 11.10 Failed/canceled output

**A.** With `create_new`, failed/canceled Dreams can leave a partially populated output store. Anthropic does not automatically delete that store; inspect or delete/archive it explicitly.

## 11.11 Input availability is a real dependency

**A.** Deleting/archiving an input Memory Store or deleting an input session while a Dream is pending/running can cause failure such as:

```text
input_memory_store_unavailable
input_session_unavailable
```

Current API also documents other Dream errors such as timeout/internal errors, organization-store-limit errors, and oversized input-store errors.

## 11.12 Usage/billing

**A.** Dream resources expose cumulative usage across pipeline stages including input/output/cache token counts.

**A.** Billing uses ordinary model token pricing for the selected model/speed and scales roughly with input store/session size and pipeline work.

## 11.13 Archiving/canceling the Dream is separate from output-store lifecycle

**A.** Cancel/archive operations apply to the Dream job/resource. They do not automatically mean “delete the output Memory Store.” Output store lifecycle is managed through Memory Store operations.

## 11.14 Two-speed memory architecture

**A/C.** Anthropic now publicly exposes both:

```text
FAST LOOP
session → incremental memory writes
```

and:

```text
SLOW LOOP
memory + many sessions
  ↓
Dream
  ↓
deduplicate / reconcile / reorganize / generalize
```

With default `create_new`, consolidation is reviewable/non-destructive. With current EAP `update_existing`, consolidation can be in-place.

---

# 12. Cross-product architectural principles

## 12.1 Externalize durable state

**A/C.** Consumer Claude, Claude Code, API Memory, and Managed Agents all separate persistent state from the model's active context window.

## 12.2 Keep active context sparse

**A/C.** Claude Code explicitly uses a small startup index + lazy topic reads. API Memory is explicitly JIT. Consumer implementation evidence strongly suggests a similar compact listing + detailed file reads.

## 12.3 Source of truth ≠ memory

**A/C.** Repositories, databases, email, calendars, documents, APIs, and live systems should remain authoritative. Memory should preserve user-specific durable residue and pointers to those systems rather than stale copies.

## 12.4 Preserve provenance

**A/B/C.** Consumer capture distinguishes stated/observed/inferred provenance; Managed Memory records actors/versions; Claude Code separates feedback/project/reference types.

## 12.5 Scope memory

**A/C.** Scopes exist at multiple levels:

- non-project vs Project consumer memory;
- project-scoped Past Chat Search;
- Claude Code user/project/local/subagent memory;
- workspace Memory Stores;
- per-session store attachments;
- read-only vs read-write access.

## 12.6 Make durable state editable/auditable

**A/C.** Consumer Topics are editable; Claude Code memory is plain Markdown; API memory is developer-owned; Managed Memory has immutable versions/redaction.

## 12.7 Concurrency matters

**A/B/C.** Consumer capture uses `if_version`; Managed Memory uses `content_sha256`; Dreams `update_existing` adds target-store holding/conflict concerns.

## 12.8 Memory is a security boundary

**A/C.** Writable durable state can turn transient prompt injection into persistent compromise. Use trust boundaries, read-only stores, constrained paths, provenance, and explicit instruction authority.

## 12.9 Forgetting/consolidation are necessary

**A/C.** API Memory recommends expiration; captured consumer memory includes pruning/capacity consolidation; Managed Agents expose Dreams.

## 12.10 Memory is not instruction; instruction is not enforcement

```text
SOURCE OF TRUTH
  what is actually true

MEMORY
  durable context Claude should know

INSTRUCTIONS / CLAUDE.md / RULES
  how Claude should generally behave

SKILLS / PROCEDURES
  how recurring work should be performed

HOOKS / PERMISSIONS / POLICY
  deterministic constraints

TESTS / OBSERVATION
  proof the real outcome is correct
```

---

# 13. Complete failure taxonomy

A serious Claude-like memory architecture must handle at least:

1. **Capture failure** — useful durable information never admitted.
2. **Over-capture** — transient/noisy detail becomes persistent.
3. **Provenance failure** — model/tool/inference content promoted to user truth.
4. **Entity duplication** — one subject becomes several memories.
5. **Misrouting** — fact stored under wrong semantic subject/domain.
6. **Staleness** — old state stays active after change.
7. **Contradiction** — incompatible states coexist unresolved.
8. **Retrieval failure** — right memory exists but isn't read.
9. **Ranking/routing failure** — right memory loses to irrelevant context.
10. **Scope failure** — intended state unavailable due namespace boundary.
11. **Scope leakage** — state crosses Project/user/org/agent boundary improperly.
12. **Over-personalization** — memory influences irrelevant tasks.
13. **Under-personalization** — relevant memory ignored.
14. **Correction failure** — new state does not supersede/reconcile old state.
15. **Forgetting failure** — removed state persists or is recreated.
16. **Dependency-erasure failure** — derived facts survive after their sole source is erased.
17. **Concurrency failure** — writer clobbers a newer write.
18. **Target-store contention** — in-place consolidation/write target remains held.
19. **Memory poisoning** — untrusted input persists malicious durable state.
20. **Instruction escalation** — memory data incorrectly gains policy authority.
21. **Capacity failure** — index/store exceeds usable limits.
22. **Entropy failure** — duplicates/stale fragments degrade recall.
23. **Consolidation failure** — cleanup merges distinct facts or preserves bad state.
24. **Audit failure** — no reliable origin/change history.
25. **Retention mismatch** — actual deletion/retention differs from user expectation.
26. **Model-use failure** — correct context reaches Claude but is misused.
27. **Source-of-truth divergence** — memory becomes stale duplicate of live reality.
28. **Compaction confusion** — temporary context summary mistaken for durable memory.
29. **History/memory conflation** — resumed/searchable transcripts mistaken for synthesized memory.
30. **Preview-drift failure** — fast-moving API behavior changes while docs/spec assumptions remain stale.

---

# 14. Experiments that could reduce remaining unknowns

## 14.1 Consumer Claude

- natural durable fact vs explicit “remember X” write latency;
- repeated vs one-off memory admission;
- stable identity vs temporary project-state routing;
- direct user fact vs Claude inference vs tool result vs user-confirmed tool result;
- aliases/entity resolution;
- correction chronology;
- explicit forgetting + dependent-fact deletion;
- simultaneous Chat/cloud-Cowork writes;
- many-topic sparse retrieval;
- Project leakage tests;
- Incognito read/write boundary;
- import/export fidelity;
- hundreds/thousands of memories for scale/pruning.

## 14.2 Claude Code

- verify 200-line vs 25-KB startup cutoff independently;
- relevant content beyond cutoff;
- index-overflow reminders/errors;
- worktrees vs independent repos/machines;
- `autoMemoryDirectory` user/project/local + workspace trust;
- `modified` frontmatter behavior;
- transcript cleanup vs auto-memory;
- ordinary vs forked subagent inheritance;
- globally disabled auto-memory + subagent `memory:` field;
- `/context` before/after compaction;
- additional-directory instruction opt-in;
- rule brace-expansion/glob edge cases.

## 14.3 API / Managed Agents / Dreams

- `content_sha256` stale precondition and idempotent-final-state 200 behavior;
- simultaneous self-hosted workers and 15-second visibility;
- read-only mutation attempts;
- writes outside returned mount path;
- 10,000-memory limit behavior;
- delete live memory then retrieve version history;
- redact old version while preserving audit;
- Dream `create_new` vs `update_existing` semantics;
- target-store-held in-place Dream conflict;
- Dream cancellation/failure partial output;
- input archive/delete mid-Dream;
- consolidation quality with contradictions/duplicates;
- original read-only store + consolidated writable store rollout pattern.

---

# 15. Hard unknowns

## 15.1 Consumer retrieval — U

Unknown:

- whether embeddings are used;
- which embedding model, if any;
- whether vector search/database is used;
- lexical vs semantic router internals;
- recency/frequency/salience weights;
- thresholds;
- max files read per turn;
- memory token budget;
- whether main Claude selects files or a separate router does;
- reranking/top-K;
- fallback behavior when all relevant files do not fit.

## 15.2 Consumer memory writing — U

Unknown:

- exact background writer model/service;
- exact post-turn prompt;
- classifier involvement;
- admission threshold/formula;
- durability/repetition scoring;
- precise merge/conflict algorithm;
- additional periodic maintenance beyond turn-level filing;
- whether ordinary consumer memory runs a hidden Dreams-like consolidator.

## 15.3 Consumer storage — U

Unknown:

- physical database/object-store technology;
- exact backend schema;
- one-file-to-one-record mapping;
- total account capacity;
- per-topic capacity;
- cache/shard/tenant architecture;
- exact Chat/cloud-Cowork consistency model;
- exact deletion propagation;
- exact production provenance representation.

## 15.4 Consumer decay/consolidation — U

Unknown:

- automatic merging outside capacity pressure;
- memory decay from non-use;
- whether retrieval reinforces retention;
- whether salience changes from access/repetition;
- hidden periodic consolidation;
- relationship between legacy migration and new memory maintenance.

## 15.5 Cross-surface writers — U

Confirmed:

```text
Chat ↔ cloud Cowork
```

Unknown:

- complete list of surfaces able to write the shared consumer store;
- whether Claude in Chrome/Excel/PowerPoint or other surfaces write directly or through Chat/Cowork mediation;
- whether all surfaces use the captured provenance/file schema;
- whether `if_version` is universal production behavior.

---

# 16. Claims we must not make without new evidence

Do **not** claim:

- “Claude consumer memory definitely uses a vector database.” — **unknown**.
- “Every memory is inserted into every prompt.” — evidence points to selective loading.
- “Anthropic physically stores consumer Markdown files on disk.” — unproven.
- “Claude Code and Claude Chat share one store.” — no official evidence.
- “Projects search all chats globally.” — false; project boundaries apply.
- “Deleting a source chat deletes current generated memory.” — false in current topic-based memory.
- “Memory guarantees Claude follows a preference.” — memory is context, not enforcement.
- “Dreams always create a separate output and never mutate input.” — **stale**; default `create_new` is non-mutating, but current EAP `update_existing` can consolidate the input in place.
- “Dreams always modify the input.” — also false; default remains `create_new`.
- “The Fable 5.1 capture is an official Anthropic product contract.” — false; Class B only.
- “Managed Memory Store capacity is 2,000.” — stale; current docs say **10,000**.
- “`autoMemoryDirectory` can never come from project/local settings.” — stale; current docs say any supported settings scope, with workspace trust.
- “Incognito means zero retention.” — false.
- “Monthly Recap directly ingests raw Gmail/Drive content.” — false under current docs.
- “Compaction deletes Claude Code persistent memory.” — false.
- “Resumed subagent conversation history is the same thing as persistent subagent memory.” — false distinction.

---

# 17. Canonical architecture

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
                  │                       compact routing/index
                  │                                │
                  │                         JIT detailed reads
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
                    explicit correction / forget
                                  │
                    background/agent memory writes
                                  │
                       concurrency-safe persistence
                                  │
                                  ▼
                              FUTURE USE
                                  │
                       pruning / consolidation
                                  │
                                  ▼
                                DREAMS
                  create_new or EAP update_existing
```

---

# 18. High-confidence fact checklist

The following are all high-confidence/current unless noted:

1. Modern consumer memory is individual categorized Topics/files, not one daily summary for normal users.
2. The major current consumer transition happened July 10, 2026.
3. Chat and cloud Cowork share memory.
4. Local Cowork does not share the cloud consumer memory system.
5. Projects have isolated memory spaces and dedicated summaries.
6. Project Knowledge/RAG is distinct from Project memory.
7. Project RAG is currently available across Free/Pro/Max/Team/Enterprise.
8. Past Chat Search is separate RAG, with Project boundaries.
9. Past Chat Search is currently Pro/Max/Team/Enterprise.
10. Memory and chat search have separate controls.
11. Pause stops memory reads and writes without deleting stored memory.
12. Reset deletes generated memory including Project memory.
13. Deleting a source chat does not automatically delete an existing current Topic.
14. Topics are individually editable/deletable.
15. Memory is included in data exports.
16. Consumer memory supports import/export.
17. Imported memory extraction is experimental and work-focused.
18. Sensitive memory is excluded by default and separately opt-in.
19. Incognito neither reads nor writes ordinary memory/history.
20. Incognito can still receive non-memory personalization.
21. Incognito is retained roughly 30 days by default, subject to organization policy.
22. Team/Enterprise have organization-level memory controls.
23. Enterprise memory entries are encrypted at rest.
24. Monthly Recap is derived and separate from memory.
25. Monthly Recap excludes Incognito/Health/Cowork/Claude Code.
26. Raw Gmail/Drive connector content is not directly used in Monthly Recap.
27. Claude Code separates `CLAUDE.md`/rules from auto-memory.
28. Claude Code treats both as context, not deterministic configuration.
29. Auto-memory has user/feedback/project/reference types.
30. Auto-memory avoids codebase-reconstructible facts.
31. Auto-memory is on by default in supported versions and can be disabled.
32. Auto-memory is machine-local by default.
33. Worktrees/subdirs of one repo share one auto-memory directory.
34. `autoMemoryDirectory` can come from supported user/project/local/policy/explicit settings with trust rules.
35. `MEMORY.md` is the startup index.
36. Startup index load is first 200 lines or first 25 KB, whichever comes first.
37. Topic files are lazy/on-demand.
38. Over-limit index writes still succeed but trigger a rewrite warning/error and excess is not loaded next time.
39. Claude Code displays memory save/recall activity.
40. `modified` frontmatter timestamps exist in v2.1.214+ when frontmatter already exists.
41. Transcript cleanup does not delete auto-memory.
42. `/memory` and `/context` serve different inspection roles.
43. Project-root `CLAUDE.md` is reread/reinjected after `/compact`.
44. Subagents can have user/project/local persistent memory.
45. Globally disabling auto-memory also disables subagent memory behavior.
46. Ordinary subagents do not automatically inherit main-agent auto-memory.
47. API Memory Tool is client-side and filesystem-like under `/memories`.
48. API Memory supports view/create/replace/insert/delete/rename semantics.
49. API Memory is designed for JIT retrieval and persistence across context resets.
50. Managed Agent Memory Stores are workspace-scoped persistent text stores.
51. Managed Store current capacity is 10,000 memories.
52. Each Managed memory is capped at 100 kB (~25K tokens).
53. Up to 8 stores can be attached per session.
54. Store attachments are chosen at session creation.
55. Read-only/read-write access is filesystem-enforced.
56. Exact `mount_path` should be read from the session resource.
57. Self-hosted stores are synchronized local copies, not live mounts.
58. Self-hosted default sync interval is approximately 15 seconds.
59. Managed memory supports stable listing/path-prefix/depth semantics.
60. `content_sha256` provides optimistic concurrency.
61. Each mutation creates immutable version history.
62. Version history can survive live memory deletion while retained.
63. Historical redaction preserves audit metadata.
64. Archive is one-way under current Managed Memory docs.
65. Store deletion removes contained memories/version history.
66. Writable memory is a documented prompt-injection persistence risk.
67. Dreams are research preview and asynchronous.
68. Dreams take one Memory Store plus 1–100 sessions.
69. Default Dream output is `create_new`, producing a separate cloned output without mutating input.
70. Current EAP/API also supports `update_existing`, consolidating the input store in place.
71. Dream instructions are capped at 4096 characters.
72. Dream model config can include standard/fast speed where supported.
73. Dream API shape is explicitly volatile in research preview.
74. Failed/canceled `create_new` Dreams can leave partial output stores.
75. Dream processing has observable session/usage state.
76. Durable memory systems need consolidation/expiration, not unbounded accumulation.

---

# 19. Evidence registry

## Consumer Claude

- Current memory/search/Projects/governance: https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
- Release timeline: https://support.claude.com/en/articles/12138966-release-notes
- Topics + Chat/cloud-Cowork shared memory: https://claude.com/blog/claudes-memory-works-everywhere-and-you-decide-whats-in-it
- Import/export: https://support.claude.com/en/articles/12123587-import-and-export-your-memory-from-claude
- Project RAG: https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects
- Incognito: https://support.claude.com/en/articles/12260368-use-incognito-chats
- Monthly Recap: https://support.claude.com/en/articles/15672559-see-your-monthly-recap

## Claude Code

- Memory / `CLAUDE.md` / rules / auto-memory: https://code.claude.com/docs/en/memory
- Context window / compaction: https://code.claude.com/docs/en/context-window
- Subagents: https://code.claude.com/docs/en/sub-agents

## Claude Platform

- API Memory Tool: https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool
- Context-engineering examples: https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools
- Managed Agent Memory: https://platform.claude.com/docs/en/managed-agents/memory
- Memory API reference: https://platform.claude.com/docs/en/api/http/beta/memory_stores/memories
- Memory Versions: https://platform.claude.com/docs/en/api/http/beta/memory_stores/memory_versions
- Dreams overview: https://platform.claude.com/docs/en/managed-agents/dreams
- Dreams create API: https://platform.claude.com/docs/en/api/http/beta/dreams/create
- Dreams type/API reference: https://platform.claude.com/docs/en/api/typescript/beta/dreams

## Class B implementation evidence

- Fable 5.1 captured system prompt: https://github.com/elder-plinius/CL4R1T4S/blob/93b0ae6fb503db6642e58f9d6352db973a900cdc/ANTHROPIC/Claude-Fable-5.1.md
- Opus 5 same-source capture: https://github.com/elder-plinius/CL4R1T4S/blob/93b0ae6fb503db6642e58f9d6352db973a900cdc/ANTHROPIC/OPUS-5.md

---

# 20. Version reconciliation / changelog

## Version 1.2 — September 8, 2026

Reconciled the evidence ledger against current Anthropic API/product documentation after v1.1 verification.

Critical correction:

- v1.1 said Dreams never mutate the input store. Current API reference now exposes **`output_behavior: update_existing`**, which in the current EAP writes consolidated memory into the Dream's own input Memory Store. The **default** remains non-destructive `create_new`.

Also made explicit:

- Dreams research-preview volatility and model `speed` configuration;
- target-store-held contention for in-place Dreams;
- exact current Managed Memory basic/full view and idempotent-precondition nuance;
- exact API Memory Tool view limits and command semantics;
- Claude Code 1,000-pattern/4-MiB path-rule expansion budget;
- current `CLAUDE.md` 4-MiB maximum load and under-200-line guidance;
- `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` behavior;
- `InstructionsLoaded`, `/doctor`, `/context`, `/memory` distinctions;
- current auto-memory directory/settings rules and versioned features;
- all v1.1 material retained in denser evidence-ledger form.

## Version 1.1 — September 8, 2026

Expanded v1.0 into an evidence ledger and corrected stale Managed Store capacity (`2,000` → current `10,000`) plus stale `autoMemoryDirectory` scope behavior.

## Version 1.0 — September 8, 2026

Initial canonical Claude-memory source-of-truth document.

---

# 21. Final canonical conclusion

As of September 8, 2026:

> **Claude memory is best understood as a family of external, persistent, selectively retrieved context systems—not as knowledge stored inside the model itself.**

Consumer Claude uses editable categorized Topics/files plus Project-scoped memory and separate historical-chat RAG. Chat and cloud Cowork share consumer memory. Claude Code exposes a transparent local index + topic-file architecture alongside human-authored instruction files. The Claude API exposes a developer-owned filesystem-like memory primitive. Managed Agents extend that pattern with scoped persistent stores, current 10,000-memory capacity, read/write access control, audit/version history, optimistic concurrency, managed/self-hosted synchronization, and Dreams for broader consolidation.

The final architecture rule is:

> **Source systems hold truth. Memory preserves durable user/project/agent context that is hard or expensive to reconstruct. Historical retrieval recovers exact episodes. Scope controls what can flow where. Sparse retrieval controls what reaches context. Provenance controls what should be trusted. Explicit corrections repair state. Versioning protects shared writes. Security boundaries prevent poisoned memory from becoming policy. Expiration and consolidation prevent long-lived memory from decaying into duplicates, contradictions, and stale assumptions.**

Within the evidence corpus listed above, this Version 1.2 document is the canonical exhaustive source of truth. Anything not established here belongs in the Unknowns registry until new evidence appears.
