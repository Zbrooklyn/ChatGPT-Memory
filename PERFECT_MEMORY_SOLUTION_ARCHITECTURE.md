# Perfect Memory Solution Architecture

## Master Source of Truth for Solving the Researched Agent-Memory Failure Space

**Status:** Canonical architecture target

**Repository:** `Zbrooklyn/ChatGPT-Memory`

**Purpose:** Define the architecture that systematically addresses the real-world agent-memory failures documented in `RESEARCHED_MEMORY_FAILURES.md` while preserving the core principles established in `THE_PERFECT_MEMORY.md` and `SINGLE_FILE_SEMANTIC_MEMORY_LEDGER_V1.md`.

---

## 1. Canonical answer

The strongest solution to the researched memory failures is not a larger vector database, a smarter summarizer, or more retrieval heuristics.

It is a **governed temporal memory system**:

> **One authoritative temporal memory history + one controlled mutation path + immutable provenance and scope + rebuildable derived indexes + freshness-aware retrieval + deterministic current-state projection + action/outcome reconciliation + verified maintenance and recovery.**

The shortest form is:

> **One truth. One writer. Verifiable history. Disposable indexes. Minimal context. Always reconcile with reality.**

Literal perfection is impossible because models, tools, external systems, storage media, networks, and humans can all be wrong. The objective is therefore not metaphysical perfection. The objective is an architecture in which every known class of memory corruption is either prevented, detected, bounded, recoverable, or explicitly represented as uncertainty.

---

## 2. What this architecture is solving

The researched failure set includes, among other things:

- junk and hallucinated memories,
- important facts never being stored,
- system prompts and recalled memories being re-ingested,
- stale truth surviving after correction,
- old and new facts remaining simultaneously active,
- duplicates from repeated extraction or concurrency,
- false deduplication across users,
- memory rot and uncontrolled growth,
- provenance loss,
- identity and tenant mixing,
- cross-session contamination,
- secrets entering durable memory,
- memory poisoning,
- adversarial memory exfiltration,
- successful API responses with no durable write,
- deletes that remove too little or too much,
- deleted memory remaining searchable,
- expired memory still being recalled,
- stale writers overwriting newer truth,
- partial commits,
- SQLite locking or corruption,
- stale or corrupt embeddings,
- vector, graph, UI, and database representations disagreeing,
- embedding-model migrations breaking retrieval,
- rerankers silently not running,
- relevant candidates being truncated before reranking,
- memory being retrieved but ignored,
- memory retrieval being skipped on turns such as “continue,”
- bad retrieval creating search loops,
- compaction losing pending memories,
- compaction summaries becoming durable “facts,”
- recursive summarization losing load-bearing details,
- stale tasks surviving compaction,
- maintenance systems silently falling back,
- backup/restore resurrecting deleted or superseded memory,
- and long-lived context becoming too expensive and cognitively harmful.

The architecture must treat all of these as one connected state-integrity problem rather than independent retrieval bugs.

---

## 3. Foundational principle: memory is governed state, not an LLM feature

An LLM may:

- interpret evidence,
- propose memories,
- classify memories,
- identify contradictions,
- suggest consolidation,
- retrieve memories,
- reason from memories,
- and propose corrections.

An LLM must **not** be able to unilaterally redefine durable truth.

The system boundary is:

```text
LLM / USER / TOOL / EXTERNAL WORLD
                │
                ▼
             EVIDENCE
                │
                ▼
        MEMORY CANDIDATES
                │
                ▼
        MEMORY CONTROL PLANE
                │
                ▼
      SINGLE AUTHORIZED WRITER
                │
                ▼
       CANONICAL MEMORY LEDGER
```

The core design rule is:

> **Models can propose state. Only the governed memory runtime can commit state.**

---

## 4. Canonical architecture

```text
                           REAL WORLD
                               │
                               ▼
                      Evidence / Observations
                               │
                               ▼
                       MEMORY CANDIDATES
                               │
                    ┌──────────┴──────────┐
                    │   MEMORY CONTROL    │
                    │       PLANE         │
                    │                     │
                    │ write policy        │
                    │ scope               │
                    │ authorization       │
                    │ provenance          │
                    │ trust               │
                    │ temporal validity   │
                    │ sensitivity         │
                    │ contradiction       │
                    │ deduplication       │
                    │ entity resolution   │
                    └──────────┬──────────┘
                               │
                     single mutation path
                               │
                    atomic/versioned commit
                               │
                               ▼
             ┌────────────────────────────────┐
             │       CANONICAL LEDGER         │
             │                                │
             │ evidence-linked memory history │
             │ temporal truth                 │
             │ decisions                      │
             │ commitments                    │
             │ procedures                     │
             │ outcomes                       │
             │ supersession history           │
             └───────────────┬────────────────┘
                             │
                     rebuildable only
            ┌────────────────┼─────────────────┐
            ▼                ▼                 ▼
      memory.sqlite      embeddings      world-state
      FTS / metadata     semantic index   projection
            │                │                 │
            └────────────────┼─────────────────┘
                             ▼
                 RETRIEVAL + REVALIDATION
                             │
                             ▼
                   MINIMUM WORKING MEMORY
                             │
                             ▼
                        AGENT REASONS
                             │
                             ▼
                            ACTS
                             │
                             ▼
                      OBSERVE REALITY
                             │
                             ▼
                 EXPECTED ↔ ACTUAL STATE
                        RECONCILIATION
                             │
                             └──────────────↺
```

---

## 5. Authority model

### 5.1 One logical authority

There must be exactly one place from which durable memory truth can be reconstructed.

For SF-SML V1, that authority can be:

```text
MEMORY_LEDGER.md
```

The permanent invariant is not necessarily “one physical file forever.” It is:

> **One canonical memory authority. Everything else is derived.**

If physical storage later moves to SQLite, PostgreSQL, an event store, or another engine, the one-authority invariant remains.

### 5.2 Derived state is never authority

The following are disposable acceleration structures:

- `memory.sqlite`,
- vector embeddings,
- FTS indexes,
- graph indexes,
- entity indexes,
- temporal projections,
- caches,
- current-state views,
- context summaries,
- retrieval telemetry.

The required recovery property is:

> Delete every derived artifact, retain the canonical ledger and required evidence, rebuild, and lose no durable truth.

### 5.3 Generation identity

Every successful canonical mutation increments a monotonically increasing ledger generation:

```text
8410
8411
8412
8413
```

Every derived artifact records:

```text
source_ledger_generation: 8413
```

This makes stale indexes, stale writers, stale projections, and stale backups observable instead of invisible.

---

## 6. Exactly one authoritative mutation path

Many readers are allowed.

Only one coordinated mutation path is allowed.

```text
                        READERS
                          │
         ┌────────────────┼────────────────┐
         ▼                ▼                ▼
      Agent A          Agent B          Indexer
         │                │
         └────── memory proposals ────────┘
                          │
                          ▼
                   MEMORY MANAGER
                 single coordinated writer
                          │
                   generation / CAS
                          │
                    atomic commit
                          │
                          ▼
                   CANONICAL LEDGER
```

Forbidden as independent writers:

```text
agent edits ledger directly
shell edits ledger directly
background reviewer rewrites ledger directly
compactor rewrites ledger directly
indexer rewrites ledger directly
janitor rewrites ledger directly
another session rewrites ledger directly
```

All of them must submit proposals through the same mutation protocol.

This directly addresses the real production failure where stale memory representations overwrote newer `MEMORY.md` state.

---

## 7. Evidence before memory

Every consequential memory must be grounded in evidence.

Evidence may include:

- user statements,
- assistant statements,
- tool responses,
- API responses,
- files,
- database observations,
- screenshots,
- external-system state,
- action receipts,
- test results,
- system events,
- timestamps,
- or prior authoritative memory records.

Evidence is not automatically truth.

The system must preserve the distinction between:

```text
observation
user assertion
external assertion
model inference
hypothesis
verified conclusion
disproven conclusion
```

A model-generated inference may never silently become an observed fact.

---

## 8. Memory formation pipeline

The durable write path is:

```text
experience / evidence
        ↓
candidate extraction
        ↓
write-policy filter
        ↓
scope + authorization check
        ↓
sensitivity check
        ↓
entity resolution
        ↓
duplicate / false-dedup check
        ↓
contradiction + temporal analysis
        ↓
provenance attachment
        ↓
mutation proposal
        ↓
version / CAS validation
        ↓
atomic commit
        ↓
read-after-write verification
        ↓
derived-index update or rebuild
```

The write-policy question is not merely “is this memorable?” It is:

```text
Does this matter later?
Is it allowed to be stored?
What scope owns it?
How trustworthy is the source?
Is it observation or inference?
Does it duplicate or contradict existing memory?
Does it supersede something?
How volatile is it?
What evidence supports it?
```

---

## 9. Canonical memory object

A consequential durable memory should conceptually support:

```yaml
id: mem_000123

generation_created: 8413

type: decision

scope:
  user: ...
  organization: ...
  project: ...
  agent: ...
  task: ...
  session: ...

statement: ...

epistemic_status: verified
authority: authoritative
confidence: 1.0

created_at: ...
updated_at: ...
observed_at: ...
valid_from: ...
valid_to: null

status: active

source:
  type: tool_observation
  evidence_id: ev_009821

supersedes: []
superseded_by: []
depends_on: []

freshness:
  class: stable
  revalidate_after: null

sensitivity: normal
retention: durable
```

Not every memory requires every field, but the system must support these concepts.

---

## 10. Temporal truth and supersession

Memory must distinguish history from current truth.

Bad model:

```text
favorite_player = Ronaldo
favorite_player = Messi
```

with both active.

Correct model:

```text
mem_100
favorite_player = Ronaldo
valid_from: T1
valid_to: T2
status: superseded

mem_205
favorite_player = Messi
valid_from: T2
valid_to: null
status: active
supersedes: mem_100
```

History is preserved without confusing old truth with current truth.

Normal correction should prefer supersession/versioning over destructive overwrite.

---

## 11. Supersession is not deletion

A correction and a privacy/security deletion are different operations.

### Correction

Preserve historical truth and mark the old state superseded.

### Deletion

When policy requires genuine forgetting:

```text
resolve exact authorized scope
        ↓
remove/redact canonical authority as policy requires
        ↓
invalidate affected derived artifacts
        ↓
rebuild/update indexes
        ↓
search for supposedly deleted information
        ↓
verify absence
        ↓
apply backup-retention policy
```

A successful delete API response is not proof of deletion.

Read-after-delete verification is mandatory for consequential deletion.

---

## 12. Immutable scope and path-complete authorization

Scope is part of memory identity, not optional metadata.

At minimum the architecture must be able to distinguish:

```text
user
organization
project
agent
task
session
entity
```

Fields that determine ownership/security should not be casually mutable after creation.

Authorization must apply consistently to every path:

- semantic search,
- lexical search,
- direct lookup,
- context builders,
- graph traversal,
- entity resolution,
- deduplication,
- consolidation,
- deletion,
- export,
- backup/restore,
- debugging tools,
- maintenance jobs.

A system is not secure if the main search API enforces tenant scope but a graph lookup or maintenance path does not.

---

## 13. Scope-aware deduplication

Identical text does not imply identical memory identity.

The deduplication key must consider scope and semantics.

Bad:

```text
hash(statement)
```

Better conceptually:

```text
scope
+ entity identity
+ memory type
+ temporal interval
+ normalized claim
+ provenance relationship
```

The system must defend against both:

- false negatives: duplicates remain separate,
- false positives: legitimate distinct memories get collapsed.

---

## 14. Derived indexes must be disposable and generation-bound

Embeddings and indexes exist to make retrieval fast, not to define truth.

Every derived index must include enough metadata to answer:

```text
Which ledger generation produced me?
Which schema version produced me?
Which embedding model produced me?
Which tokenizer/configuration produced me?
```

If an index is stale or incompatible:

```text
fail closed for consequential retrieval
or
rebuild before use
```

Do not silently serve a stale index as current memory.

Changing embedding models should be a rebuild event, not a memory migration.

---

## 15. Deterministic current-world projection

The system should not maintain a second manually edited “current state” authority.

Instead derive current views from the temporal ledger:

```text
CURRENT_PROJECT_STATE
CURRENT_RELATIONSHIPS
CURRENT_PREFERENCES
CURRENT_COMMITMENTS
CURRENT_DECISIONS
CURRENT_PROCEDURES
CURRENT_OPEN_FAILURES
```

This gives:

> **one historical authority → deterministic current-state projection**

rather than two competing truths.

---

## 16. Retrieval is a governed decision process

Vector similarity alone is insufficient.

Candidate retrieval should be allowed to use:

```text
semantic relevance
lexical relevance
entity match
scope
permission
memory type
temporal validity
freshness
authority
confidence
provenance quality
decision relevance
failure relevance
open commitments
dependencies
current objective
```

The runtime should retrieve a broad enough candidate set that reranking can actually recover the right memory.

Reranking must be observable so a configured reranker cannot silently fail to execute.

The target is:

> **maximum expected decision value per context token**

not “top-k cosine similarity.”

---

## 17. Minimum sufficient working memory

Persistent memory should not be dumped wholesale into context.

The context builder creates the smallest trusted state needed for the current objective.

Conceptually:

```text
objective
  ↓
relevant entities
  ↓
current authoritative state
  ↓
important history/rationale
  ↓
open commitments
  ↓
known failures/constraints
  ↓
uncertainties needing revalidation
  ↓
minimum working context
```

Large memory should result in better selective reconstruction, not ever-larger prompts.

---

## 18. Freshness and revalidation

Remembered truth is not necessarily current truth.

Every memory that can become stale should have a freshness policy.

Example classes:

```text
stable
revalidate_if_important
volatile
```

Examples:

- historical decision: stable,
- user's long-term preference: usually stable but correctable,
- production deployment state: revalidate if consequential,
- flight status: volatile,
- account balance: volatile,
- current API availability: volatile.

For consequential action:

```text
retrieve remembered state
        ↓
check freshness policy
        ↓
reobserve if necessary
        ↓
act
```

This closes the observation-to-action staleness gap.

---

## 19. Action/outcome reconciliation

Command success is not world-state success.

Bad:

```text
deploy command returned success
        ↓
write “production is deployed”
```

Correct:

```text
intended state
        ↓
perform action
        ↓
observe actual state
        ↓
compare expected ↔ actual
        ↓
verify outcome
        ↓
commit reconciled memory
```

Every meaningful action should answer:

```text
What did I intend?
What actually happened?
Did they match?
What durable state changed?
What remains unresolved?
```

The full memory loop is:

> **remember → decide → act → observe → reconcile → learn**

---

## 20. Compaction and durable memory are separate systems

Context compaction is a working-context optimization.

Durable memory is persistent governed state.

They must not be conflated.

### Durable-memory path

```text
conversation / evidence
        ↓
validated memory formation
        ↓
canonical ledger
```

### Context-management path

```text
large working context
        ↓
compression / summarization
        ↓
temporary working summary
```

A compaction summary is not automatically evidence that its claims are true and must never become authoritative merely because it was generated during compaction.

Before compaction or shutdown, pending durable-memory writes must be flushed or explicitly retained for recovery.

---

## 21. Consolidation and memory hygiene

Long-lived memory requires hygiene, but hygiene itself is dangerous.

The system should periodically identify candidates such as:

```text
duplicates
contradictions
stale memories
expired temporary state
unresolved hypotheses
unverified inferences
dangling entities
superseded state
low-value noise
failed maintenance
repeated corrections
```

Consolidation should normally propose changes through the same Memory Manager rather than rewriting the authority independently.

Maintenance must be observable:

```text
requested
started
completed
failed
degraded
fell_back
verified
```

Silent fallback from consolidation to append-only behavior is not acceptable.

---

## 22. Backup and recovery

Backup must preserve a coherent authoritative generation.

The safest recovery root is:

```text
canonical ledger generation N
+ required evidence
+ schema/version metadata
+ integrity metadata
```

Restore that generation.

Then regenerate:

```text
memory.sqlite
embeddings
FTS
graph indexes
entity indexes
world-state projection
caches
```

Do not independently restore an old vector database alongside a newer ledger.

Every restore must verify that derived artifacts were rebuilt from the restored canonical generation.

This prevents deleted, expired, or superseded memories from being resurrected by mismatched backup generations.

---

## 23. Concurrency and transaction protocol

For each authoritative mutation:

```text
1. Read canonical generation N.
2. Build mutation against N.
3. Validate scope, provenance, temporal semantics, and policy.
4. Attempt compare-and-swap / versioned commit from N → N+1.
5. If generation changed, reject and recompute against the new state.
6. Commit atomically.
7. Read back canonical state.
8. Verify expected mutation exists exactly once.
9. Update/rebuild derived structures.
10. Verify derived generation = canonical generation.
```

This is the core defense against stale writers, duplicate concurrent writes, lost updates, and partial state.

---

## 24. Security and trust boundaries

Persistent memory creates a long-lived attack surface.

The system must distinguish:

```text
trusted instruction
authorized user assertion
untrusted external content
tool output
retrieved web content
model inference
stored memory
```

Stored memory is not automatically trusted instruction.

Memory-security controls must cover both directions:

### Ingestion security

Prevent untrusted content, prompt injection, secrets, and irrelevant data from becoming authoritative memory.

### Retrieval/disclosure security

Prevent authorized stored memory from being exfiltrated through adversarial queries, cross-scope retrieval, graph traversal, or debugging endpoints.

### Integrity security

Preserve enough provenance/integrity metadata to detect unauthorized or inconsistent mutation.

---

## 25. Observability

A perfect-memory architecture must be diagnosable.

Useful observability includes:

```text
ledger_generation
last_verified_generation
index_generation
write proposals accepted/rejected
write conflicts
read-after-write failures
read-after-delete failures
contradictions detected
supersessions
revalidation requests/results
retrieval candidates
reranker execution
retrieval misses
scope-denied accesses
maintenance status
backup generation
restore generation
rebuild status
corruption checks
memory-use telemetry
```

Telemetry used for decay or cleanup must itself be verified; bad usage counters must not cause useful memories to be discarded.

---

## 26. Failure-to-control mapping

| Failure family | Primary architectural defense |
|---|---|
| Formation corruption | evidence layer + governed write gate |
| Omission | candidate capture + explicit write verification |
| Temporal corruption | validity intervals + supersession + revalidation |
| Provenance corruption | immutable evidence references + epistemic status |
| Scope/identity corruption | immutable scope + path-complete authorization |
| Mutation corruption | one writer + version/CAS + atomic commit |
| Deletion/forgetting corruption | canonical deletion + read-after-delete verification |
| Derived-state drift | canonical authority + generation-bound rebuildable indexes |
| Retrieval failure | hybrid candidate generation + observable reranking |
| Retrieval-policy failure | objective-aware context builder + expected-memory detection |
| Adherence failure | typed, authority-aware working context + evaluation |
| Context/compaction corruption | hard separation between compaction and durable memory |
| Durability/transaction failure | atomic writes + restart/crash/corruption recovery |
| Lifecycle/hygiene failure | governed consolidation, expiry, archival, and maintenance observability |
| False deduplication | scope-aware memory identity |
| Adversarial extraction | retrieval/disclosure authorization and red-team tests |
| Backup resurrection | generation-consistent backups + rebuild derived state |
| Lossy recursive summarization | preserve evidence/history; summaries never authoritative |

---

## 27. Non-negotiable invariants

```text
ONE CANONICAL MEMORY AUTHORITY

ONE AUTHORIZED MUTATION PATH

MANY READERS, ONE COORDINATED WRITER

ATOMIC VERSIONED WRITES

MONOTONIC LEDGER GENERATION

READ-AFTER-WRITE VERIFICATION

READ-AFTER-DELETE VERIFICATION

NO DIRECT EMBEDDING AUTHORITY

NO DIRECT VECTOR-STORE AUTHORITY

NO DIRECT GRAPH AUTHORITY

NO DIRECT SUMMARY AUTHORITY

NO DIRECT AGENT-GENERATED FACT AUTHORITY

OBSERVATION AND INFERENCE MUST REMAIN DISTINCT

IMMUTABLE OR STRICTLY GOVERNED SCOPE / IDENTITY

SCOPE-AWARE DEDUPLICATION

PATH-COMPLETE AUTHORIZATION

PROVENANCE FOR CONSEQUENTIAL MEMORY

EXPLICIT TEMPORAL SUPERSESSION

FRESHNESS / REVALIDATION POLICY

DERIVED ARTIFACTS ARE GENERATION-BOUND

CURRENT STATE IS A PROJECTION, NOT A SECOND AUTHORITY

PENDING WRITES FLUSH BEFORE COMPACTION / SHUTDOWN

CONTEXT COMPACTION IS NOT DURABLE MEMORY

CONSOLIDATION IS A VERIFIED MUTATION

BACKUP / RESTORE MUST PRESERVE GENERATION CONSISTENCY

ADVERSARIAL MEMORY-DISCLOSURE TESTS

INDEX REBUILD TEST

RESTART RECOVERY TEST

CONCURRENCY TEST

CRASH / PARTIAL-WRITE RECOVERY TEST

CORRUPTION RECOVERY TEST

EMBEDDING-MIGRATION REBUILD TEST

PERIODIC VERIFIED MEMORY HYGIENE

ACTION OUTCOMES MUST BE RECONCILED WITH REALITY
```

---

## 28. V1 physical architecture

The simplest serious implementation remains small.

```text
MEMORY_LEDGER.md
    canonical authoritative temporal memory

memory.sqlite
    rebuildable index
    metadata
    FTS
    semantic retrieval references

memory_manager
    only writer
    remember
    correct
    supersede
    delete
    consolidate
    revalidate
    reconcile

context_builder
    authorize
    retrieve
    rerank
    check freshness
    reconstruct minimum sufficient working memory
```

### V1 may defer

- graph database,
- distributed consensus,
- complex autonomous forgetting,
- multiple vector databases,
- elaborate ontology,
- recursive summary trees,
- separate world-model database,
- microservices,
- high-frequency multi-writer distribution.

Complexity should be added only when observed failure or scale requires it.

---

## 29. V1 required mutation operations

At minimum:

```text
remember()
correct()
supersede()
delete()
revalidate()
consolidate()
reconcile()
```

And read-side operations:

```text
retrieve()
get_current_state()
get_history()
get_evidence()
build_context()
verify_generation()
```

---

## 30. V1 acceptance tests

A memory system should not be called reliable because simple recall demos pass.

The minimum acceptance suite should test:

### Durability

```text
write
kill process
restart from nothing
recover exact durable memory
```

### Rebuildability

```text
delete memory.sqlite
remove embeddings/indexes/caches
rebuild from canonical ledger
verify equivalent retrieval/current state
```

### Supersession

```text
A is current
new evidence establishes B
A becomes historical
B becomes current
retrieval does not present A as current
```

### Concurrency

```text
two writers propose conflicting/duplicate updates
only one valid canonical transition commits
stale writer is rejected/recomputed
```

### Scope isolation

```text
Alice and Bob store identical text
both retain independent memories
neither can infer or retrieve the other's memory
```

### Delete verification

```text
delete exact scoped memory
verify canonical absence/redaction per policy
verify search absence
verify graph/index/cache absence
```

### Index drift

```text
force stale derived generation
consequential retrieval must detect mismatch
rebuild or fail closed
```

### Embedding migration

```text
change embedding model/dimension
rebuild derived semantic index
canonical memory remains unchanged
```

### Compaction

```text
compact context repeatedly
canonical history remains unchanged
pending durable writes survive
summary text cannot become authoritative without validation
```

### Backup/restore

```text
backup generation N
mutate to N+M
restore N
rebuild every derived artifact from N
verify no N+M derived state survives
```

### Revalidation

```text
retrieve volatile memory
force staleness threshold
consequential action requires live reobservation
```

### Outcome reconciliation

```text
action command reports success
world state intentionally differs
memory must record actual observed state, not intended state
```

### Adversarial security

```text
attempt prompt-injection persistence
attempt cross-scope retrieval
attempt stored-memory exfiltration
attempt unauthorized graph/direct read
attempt metadata scope mutation
```

### Long-horizon hygiene

```text
run sustained memory accumulation
verify duplicates/stale/contradictory state remains bounded and auditable
verify maintenance failures are observable
```

---

## 31. Fresh-context acceptance standard

After complete loss of active model context, a fresh agent should be able to reconstruct the relevant state and continue intelligently without forcing the user to repeat durable information.

It should be able to determine:

- what is currently true,
- what used to be true,
- what changed,
- why it changed,
- what evidence supports current truth,
- what remains uncertain,
- what is stale and needs revalidation,
- what decisions were made and why,
- what commitments remain open,
- what procedures are known,
- what failed before,
- what must not be repeated,
- what action is appropriate next.

And it must do so without:

- treating old truth as current,
- leaking another scope's memory,
- trusting unverified model inference as observation,
- drowning the model in history,
- or depending on an unrebuildable vector/index representation.

---

## 32. Relationship to the other repository documents

### `THE_PERFECT_MEMORY.md`

Defines the ideal properties and conceptual standard for perfect agent memory.

### `RESEARCHED_MEMORY_FAILURES.md`

Documents the real-world evidence: production bugs, long-running-agent failures, operational incidents, security problems, and failure families that this architecture is intended to solve.

### `SINGLE_FILE_SEMANTIC_MEMORY_LEDGER_V1.md`

Defines the detailed SF-SML implementation direction and single-file semantic-ledger approach.

### `PERFECT_MEMORY_SOLUTION_ARCHITECTURE.md`

This document defines the **canonical solution architecture** that connects the ideal requirements, the researched failure evidence, and the V1 implementation strategy.

The relationship is:

```text
THE_PERFECT_MEMORY.md
        │
        │ defines the target
        ▼
RESEARCHED_MEMORY_FAILURES.md
        │
        │ proves what breaks in reality
        ▼
PERFECT_MEMORY_SOLUTION_ARCHITECTURE.md
        │
        │ defines the governed solution
        ▼
SINGLE_FILE_SEMANTIC_MEMORY_LEDGER_V1.md
        │
        │ implements the simplest serious V1
        ▼
      RUNTIME
```

---

## 33. Final architecture definition

> **A perfect-memory solution for an agentic AI harness is a governed temporal state system in which observations and evidence are transformed into durable memory only through a controlled, provenance-preserving, scope-aware, versioned mutation path; one canonical authority preserves historical and current truth; all indexes, embeddings, graphs, summaries, and projections are derived and rebuildable; retrieval reconstructs the minimum sufficient trusted context while enforcing authorization, temporal validity, freshness, and epistemic status; consequential claims are revalidated when needed; actions are followed by observation and expected-versus-actual reconciliation; and maintenance, deletion, compaction, consolidation, backup, restore, migration, and recovery are themselves treated as verified state transitions rather than trusted side effects.**

---

## 34. Ultimate operating principle

The architecture should make it difficult for the system to become confidently wrong over time.

The most dangerous memory failure is not simply forgetting.

It is durable corruption of the agent's model of reality.

Therefore the ultimate standard is:

> **Preserve evidence. Govern truth. Version change. Minimize context. Revalidate reality. Verify every mutation.**

And the permanent shorthand remains:

> **One truth. One writer. Verifiable history. Disposable indexes. Minimal context. Continuous reconciliation.**
