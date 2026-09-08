# Master Source of Truth — Real-World Agent Memory Failures

Yes. I found enough real-world evidence to materially refine our “perfect memory” design.

I cannot prove I found literally every memory failure ever observed, but I found documented examples covering essentially the full failure space: production audits, long-running internal agents, official framework troubleshooting, and reproducible issue reports across Mem0, Letta, LangGraph, CrewAI, and AutoGen.

The biggest finding is severe: one Mem0 user audited **10,134 memory records after 32 days in production and reported 97.8% as junk**. Their failures included repeated system-prompt content, cron/heartbeat noise, transient task state, hallucinated user profiles, identity confusion, sensitive information, fabricated locations, and a feedback loop where retrieved memories were re-extracted as new memories. ([github.com](https://github.com/mem0ai/mem0/issues/4573))

Letta independently reports similar behavior from long-running internal production agents: memory becomes duplicated, stale, poorly structured, and contradictory over time. Their current evaluation explicitly measures “memory rot,” hygiene, retrieval failures, failure to generalize corrections, and failure to obey memory. ([letta.com](https://www.letta.com/blog/evaluating-memory-in-production-agents/))

## 1. Bad memories get written in the first place

| Failure | Real-world evidence |
|---|---|
| **Over-memory / remembering everything** | Production Mem0 audit found thousands of boot-file restatements, system architecture dumps, heartbeat noise, and temporary task state stored as durable memory. ([github.com](https://github.com/mem0ai/mem0/issues/4573)) |
| **Hallucinated memories** | Same audit found hundreds of fabricated profiles, including fictional users, occupations, locations, routines, and employers. ([github.com](https://github.com/mem0ai/mem0/issues/4573)) |
| **Empty input generates fake memories** | Mem0 had a reproducible bug where an empty messages array could reach the extraction model and produce hallucinated memories. ([github.com](https://github.com/mem0ai/mem0/issues/5465)) |
| **System instructions become memories** | Mem0 users reproduced parts of the memory-extraction system prompt being stored as actual memories. ([github.com](https://github.com/mem0ai/mem0/issues/2736)) |
| **Recalled memories become new memories** | The 10,134-entry audit documented feedback-loop amplification: recalled context was re-extracted, causing repeated copies. ([github.com](https://github.com/mem0ai/mem0/issues/4573)) |
| **Useful information isn't remembered** | A Mem0 report found meaningful facts sometimes produced empty extraction results and were silently not persisted. ([github.com](https://github.com/mem0ai/mem0/issues/3009)) |
| **Extraction truncation destroys otherwise valid output** | Long extraction responses hitting token limits could invalidate the entire structured response and drop all extracted facts. ([github.com](https://github.com/mem0ai/mem0/issues/5428)) |
| **Corrections get logged instead of learned** | Letta's production-derived benchmark finds agents repeatedly append dated corrections instead of forming a generalized rule. ([letta.com](https://www.letta.com/blog/evaluating-memory-in-production-agents/)) |

This means the **memory write gate is load-bearing**. An LLM extracting “memories” from every conversation is not enough.

---

## 2. Old truth refuses to die

This is probably the most important class for your architecture.

A Mem0 user described production agents silently retrieving stale or conflicting preferences. Another financial-agent developer reported old financial information surviving after newer earnings data had superseded it. ([github.com](https://github.com/mem0ai/mem0/issues/5235))

Mem0 also has a concrete reproduction where:

> favorite player = Ronaldo

followed later by:

> favorite player = Messi

can result in **both memories remaining active**, making later retrieval ambiguous. ([github.com](https://github.com/mem0ai/mem0/issues/5867))

Practitioners report the same thing with general vector stores: after months, the store becomes a “museum of obsolete facts,” and teams end up adding freshness and `superseded_by` semantics themselves. ([reddit.com](https://www.reddit.com/r/LangChain/comments/1w51pcx/what_breaks_in_ai_agent_memory_after_months_in/))

This validates exactly why we proposed:

```text
valid_from
valid_to
status
supersedes
superseded_by
```

instead of merely editing text.

---

## 3. Memory accumulates until it rots

Mem0 users report that memories accumulate indefinitely without expiration or decay, causing retrieval noise, context inflation, and stale facts contaminating answers. ([github.com](https://github.com/mem0ai/mem0/discussions/5393))

Letta sees the same thing with actual long-running agents. Their memory defragmentation system exists because long-horizon memory “inevitably” becomes less organized; it splits oversized files, merges duplicates, and reorganizes memory into roughly **15–25 focused files**. ([letta.com](https://www.letta.com/blog/context-repositories/))

That is significant for our single-file idea.

It does **not** invalidate one authoritative memory source.

It does suggest:

> **one giant physical Markdown file forever is probably not the permanent optimum.**

The permanent invariant should be **one canonical memory authority**, not necessarily one physical file.

---

## 4. Duplicate memories appear

There are two distinct causes.

Normal repeated extraction creates semantic duplicates and feedback loops. ([github.com](https://github.com/mem0ai/mem0/issues/4573))

More seriously, Mem0 had a reproducible concurrency race: two parallel `add()` calls could both inspect the same stale snapshot, both conclude the memory didn't exist, and both permanently insert it. This was reported independently in the Python and TypeScript implementations. ([github.com](https://github.com/mem0ai/mem0/issues/6515))

That directly validates our requirement for:

> **one coordinated writer / transactional memory mutations / version checks**

even if many agents can read memory simultaneously.

---

## 5. Canonical memory and its index diverge

This is one of the strongest arguments for our:

> **ledger authoritative, index disposable**

architecture.

LangGraph has a reproduced case where the stored document gets updated but its old embedding remains behind. Semantic search can therefore return the document based on **text it no longer contains**. ([github.com](https://github.com/langchain-ai/langgraph/issues/8214))

Mem0 had another backend bug where an update replaced a valid embedding with a corrupt four-byte value, leaving the text intact but making the memory effectively unsearchable. ([github.com](https://github.com/mem0ai/mem0/issues/4336))

Mem0 has also had stale entity links survive deletion/update because entity cleanup could be skipped. ([github.com](https://github.com/mem0ai/mem0/issues/4863))

And one Chroma backend bug made deletion silently do nothing, so a supposedly deleted memory continued to exist. ([github.com](https://github.com/mem0ai/mem0/issues/5698))

These are classic:

```text
authoritative state ≠ derived retrieval state
```

failures.

If the index itself is treated as truth, recovery is difficult.

If:

```text
MEMORY_LEDGER
        ↓
rebuild everything
```

is guaranteed, these become repairable derived-state failures.

---

## 6. Memories exist but can't be retrieved

This happens surprisingly often.

Mem0 has reports where stored memories disappear from filtered queries because Elasticsearch metadata was mapped incorrectly. ([github.com](https://github.com/mem0ai/mem0/issues/3744))

Another bug stored memory successfully but dropped its user/agent scope metadata, making the memory effectively orphaned and unretrievable through scoped searches. ([github.com](https://github.com/mem0ai/mem0/issues/6170))

A platform bug similarly returned `SUCCEEDED` from memory creation but stored no user identity, causing `get_all()` to return zero memories later. ([github.com](https://github.com/mem0ai/mem0/issues/5224))

Neo4j graph-memory filtering has also returned empty results despite the user's memories visibly existing in the graph. ([github.com](https://github.com/mem0ai/mem0/issues/4232))

And a Mem0 plugin had auto-recall completely fail because the hook returned the wrong property name: memories were fetched but never actually inserted into the model's context. ([github.com](https://github.com/mem0ai/mem0/issues/4037))

So there are really two separate questions:

```text
Does the memory exist?

and

Can the agent actually discover and use it?
```

A perfect system must test both.

---

## 7. Retrieval returns the wrong memory

Raw vector similarity is insufficient.

A Mem0 integration was injecting memories ordered by raw vector similarity because reranking wasn't actually enabled in the plugin path. ([github.com](https://github.com/mem0ai/mem0/issues/5684))

The production-derived Letta evaluation also finds agents sometimes fail to realize that relevant external memory exists and simply answer using whatever happens to already be in context. ([letta.com](https://www.letta.com/blog/evaluating-memory-in-production-agents/))

This validates our retrieval formula:

```text
semantic relevance
+ temporal validity
+ entity
+ scope
+ authority
+ confidence
+ freshness
+ decisions
+ failures
+ open commitments
```

rather than:

```text
top_k cosine similarity
```

---

## 8. The model retrieves memory but ignores it

Storage and retrieval can work perfectly and the agent can still fail.

Letta calls this **memory adherence**.

Their real-production-derived benchmark shows agents sometimes retrieve relevant standing rules and then ignore them, override them, or even decide legitimate instructions look suspicious and delete them. ([letta.com](https://www.letta.com/blog/evaluating-memory-in-production-agents/))

That means perfect memory requires more than:

```text
memory → context
```

It requires:

```text
memory → context → correct interpretation → correct behavior
```

Memory quality and reasoning quality are coupled.

---

## 9. Sessions contaminate one another

A particularly concrete recent Mem0 bug used a last-N message buffer for extraction. When a new session contained fewer messages than the buffer size, old messages from the previous session remained and became extraction context for the new session.

The result was unrelated prior-session information influencing newly created memories. ([github.com](https://github.com/mem0ai/mem0/issues/7195))

This is exactly why we need strong:

```text
user scope
project scope
task scope
session scope
```

and explicit context boundaries.

---

## 10. Users and identities get mixed together

This is worse than stale memory because it can become a privacy problem.

The production Mem0 audit found **identity confusion**, such as models confusing the agent with the human operator. ([github.com](https://github.com/mem0ai/mem0/issues/4573))

Mem0 also had a reported entity-linking problem where memories belonging to different `user_id`/`agent_id` scopes could be associated through entity merging. ([github.com](https://github.com/mem0ai/mem0/issues/5439))

Even more serious, both its Python and TypeScript SDKs had critical bugs where an update could overwrite `user_id`, `agent_id`, or `run_id`, effectively moving memory between tenant scopes. ([github.com](https://github.com/mem0ai/mem0/issues/6277))

This makes **scope part of the memory's identity**, not optional metadata.

---

## 11. Memory leaks secrets

The 10,134-entry production audit found IP addresses, chat IDs, file paths, and sensitive configuration values entering the vector store despite not belonging there. ([github.com](https://github.com/mem0ai/mem0/issues/4573))

That means “remember everything useful” needs another condition:

> **Only if the system is allowed to persist it.**

A write policy needs:

```text
importance
+
trust
+
scope
+
sensitivity
+
retention policy
```

---

## 12. Memory can be poisoned

Persistent memory creates a particularly dangerous prompt-injection channel.

Multiple Mem0 security reports describe the attack pattern:

```text
malicious external/user input
        ↓
stored as durable memory
        ↓
retrieved in future unrelated session
        ↓
agent treats it as trusted instruction
```

([github.com](https://github.com/mem0ai/mem0/issues/5349))

There are also concerns about direct memory-store tampering because a retrieved memory may have no cryptographic evidence proving it wasn't altered. ([github.com](https://github.com/mem0ai/mem0/issues/4717))

This reinforces our rule:

> **Evidence is not authority.**

Something being stored does not make it trusted instruction.

---

## 13. Provenance gets lost

A recent Mem0 feature request identifies a fundamental limitation: after an LLM extracts a fact, the system cannot necessarily answer:

> Which exact source message produced this memory?

That makes it difficult to determine whether a memory was genuinely grounded in the user's words or invented during extraction. ([github.com](https://github.com/mem0ai/mem0/issues/7047))

This strongly validates our proposed:

```text
SOURCE
PROVENANCE
EVIDENCE
OBSERVED_AT
```

fields.

Without them, correction becomes guesswork.

---

## 14. Memory writes can silently fail

This is one of the most dangerous operational failures because the agent believes it remembered something.

Mem0 had a reproduced embedding-provider migration bug where:

```text
POST memory
→ success response
→ memory ID returned
→ nothing actually stored
```

because the vector database retained the previous embedding dimension. ([github.com](https://github.com/mem0ai/mem0/issues/4985))

CrewAI's documentation similarly warns that memory saves can fail in a background thread without crashing the agent. ([github.com](https://github.com/crewAIInc/crewAI/blob/main/docs/v1.15.12/en/concepts/memory.mdx))

Therefore:

> **write acknowledgement ≠ durable memory**

The system needs read-after-write verification for important memories.

---

## 15. Changing embedding models can break years of memory

This showed up repeatedly.

Mem0 has multiple reports of vector-dimension mismatches after changing embedding models/providers. ([github.com](https://github.com/mem0ai/mem0/issues/4985))

CrewAI explicitly warns that older memory stores may use 1536-dimensional embeddings while newer defaults use 3072 dimensions. ([github.com](https://github.com/crewAIInc/crewAI/blob/main/docs/v1.15.12/en/concepts/memory.mdx))

Letta had archival-memory paths using a different embedding configuration from the one configured for the agent. ([github.com](https://github.com/letta-ai/letta/issues/3210))

Again:

> **Embeddings should never be authoritative memory.**

They must be regenerable.

---

## 16. Current time itself can become stale

Mem0 had a bug where the “current date” used in memory extraction was evaluated at module import time.

A server starting November 11 could still tell the memory model that today was November 11 several days or weeks later. ([github.com](https://github.com/mem0ai/mem0/issues/3755))

That can corrupt:

- freshness
- relative dates
- deadlines
- temporal ordering
- “currently” assertions

Temporal memory requires real clock state, not merely timestamps on stored records.

---

## 17. Historical timestamps get corrupted

LangGraph had a reported bug where updating an existing memory item overwrote `created_at` with the update time instead of preserving the original creation timestamp. ([github.com](https://github.com/langchain-ai/langgraph/issues/8340))

That destroys part of the history.

It validates the distinction between:

```text
created_at
updated_at
observed_at
valid_from
valid_to
```

These are not interchangeable.

---

## 18. Memory disappears on restart

This is the simplest failure, but still common.

LangGraph's own persistence documentation warns that `MemorySaver`/`InMemorySaver` are RAM-only and lose state when the process restarts. ([github.com](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/persistence.mdx))

There was also a reported developer-runtime case where a configured persistent checkpointer was effectively bypassed and conversation state disappeared after restart. ([github.com](https://github.com/langchain-ai/langgraph/issues/5790))

Durability therefore has to be tested by:

```text
write
→ terminate process
→ restart from nothing
→ recover memory
```

not just by reading memory immediately after writing it.

---

## 19. Crashes can restore an impossible state

LangGraph has a report involving checkpoint ordering where, after a crash, persisted writes could be restored against a checkpoint from a different execution step.

The application could restart without throwing an obvious error but reason from internally inconsistent state. ([github.com](https://github.com/langchain-ai/langgraph/issues/8234))

That is worse than obvious memory loss.

It produces **false continuity**.

---

## 20. Batch writes can partially commit

Another LangGraph SQLite issue reports that if a later operation in a batch fails, earlier mutations may still be committed. ([github.com](https://github.com/langchain-ai/langgraph/issues/8590))

So an agent can believe:

```text
A + B + C happened atomically
```

while memory contains:

```text
A + B
```

This validates the transactional/atomicity requirement we identified earlier.

---

## 21. Multi-agent memory goes stale between observation and action

A practitioner using AutoGen described a recurring failure where one agent reads external state, another acts later, the underlying world changes in between, and nothing marks the remembered observation as stale. ([github.com](https://github.com/microsoft/autogen/discussions/8015))

This is exactly our:

> **remembered truth is not necessarily current truth**

problem.

For consequential actions:

```text
retrieve
→ check freshness
→ reobserve if necessary
→ act
```

is required.

---

## 22. Different graph components disagree about state

LangGraph documentation notes that a parent graph may not immediately see state written by a subgraph because each can operate within its own checkpoint namespace. ([github.com](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/persistence.mdx))

This is another form of competing authority.

It becomes especially dangerous when several agents independently maintain project state.

---

## 23. Context itself becomes polluted

Letta explicitly describes performance degradation from putting too much information in context as **context pollution**. ([letta.com](https://www.letta.com/blog/introducing-sonnet-4-5-and-the-memory-omni-tool-in-letta/))

Their long-running memory work therefore uses progressive disclosure and selective loading rather than putting all memory into the model. ([letta.com](https://www.letta.com/blog/context-repositories/))

So:

> bigger memory ≠ bigger prompt.

That directly supports our “minimum sufficient context” principle.

---

## 24. Memory can fill up and break the agent

An older MemGPT/Letta issue reported the agent becoming unusable when its context memory became full and it attempted archival-memory insertion. ([github.com](https://github.com/letta-ai/letta/issues/953))

The implementation has evolved substantially since then, but the failure category remains relevant:

> **memory-management operations themselves can become a critical runtime dependency.**

If memory maintenance fails, the agent should degrade gracefully rather than die.

---

## 25. The storage system itself grows without useful memory

LangGraph documents unbounded checkpoint accumulation as something that can increase storage and latency. ([github.com](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/persistence.mdx))

It also recently had a reproduced case where merely reading unknown thread IDs created persistent “ghost” in-memory entries. ([github.com](https://github.com/langchain-ai/langgraph/issues/8754))

This isn't semantic-memory corruption, but it is part of the same long-lived-agent operational failure space.

---

# The root-cause picture

After collapsing all of these vendor-specific incidents, I see **nine fundamental root causes**:

| Root cause | What it produces |
|---|---|
| **Uncontrolled memory formation** | junk, hallucinations, secrets, system-prompt memories |
| **No temporal model** | stale truth, contradictions, obsolete facts |
| **Weak identity/scope model** | user mixing, entity confusion, privacy leaks |
| **No provenance/trust model** | hallucinations become facts, poisoning |
| **Derived state treated as truth** | stale vectors, corrupted indexes, migration breakage |
| **Weak transactional semantics** | duplicates, partial commits, crash inconsistency |
| **Weak retrieval policy** | misses, irrelevant memories, wrong ranking |
| **Weak memory hygiene** | unlimited accumulation, duplication, fragmentation |
| **No reality reconciliation** | remembered intended state diverges from actual world |

And this is the important conclusion for **our** architecture:

The real-life evidence supports our approach much more strongly than the theoretical discussion did.

I would now consider these non-negotiable for the harness:

1. **One canonical memory authority.**
2. **Evidence/provenance attached to consequential memory.**
3. **Explicit temporal supersession rather than overwrite-or-append-only facts.**
4. **Derived embeddings/indexes must be disposable and rebuildable.**
5. **One coordinated mutation path with atomicity/idempotency.**
6. **Write validation + retrieval validation + post-action reality reconciliation.**
7. **Strict user/project/task scope boundaries.**
8. **Periodic hygiene/consolidation with history preserved.**
9. **Freshness/revalidation before consequential actions.**
10. **Actual restart/rebuild/corruption tests, not merely happy-path memory benchmarks.**

The biggest update I would make to our earlier design is this:

> **“Perfect memory” is less about remembering everything correctly than preventing the memory system from slowly corrupting its own model of reality.**

And based on the production evidence, I would now describe the safest architecture as:

> **Canonical evidence/history → validated memory formation → temporal authoritative state → fully rebuildable retrieval indexes → minimum working context → action → observation → reconciliation → continuous memory hygiene.**

That addresses every major real-world failure class I found.

I continued the search across current production reports and reproducible bugs in Mem0, LangGraph, Hermes, Letta, OpenClaw, Neo4j agent-memory, and practitioner reports. I’m now seeing strong saturation: additional searches mostly produce new instances of the same root failure classes.

I also excluded ordinary RAM/heap “memory leaks” unless they directly damage persistent agent memory.

The previous pass identified roughly **25 distinct failure modes**. I found another **25+ meaningful ones**.

### Additional real-world failures

| # | Failure actually observed | What happened |
|---:|---|---|
| 26 | **Scoped delete wipes everybody's memory** | A Mem0 `delete_all()` implementation called collection reset after a filtered deletion, destroying memories belonging to other users. ([github.com](https://github.com/mem0ai/mem0/issues/3928?utm_source=chatgpt.com)) |
| 27 | **“Delete all” only deletes the first page** | Mem0 silently deleted only the first 100 matching memories while returning success, leaving the remainder behind. This is particularly dangerous for deletion/privacy guarantees. ([github.com](https://github.com/mem0ai/mem0/issues/6627?utm_source=chatgpt.com)) |
| 28 | **Deleted memory remains searchable** | Users reported memories disappearing from the normal view but still returning through search. ([github.com](https://github.com/mem0ai/mem0/issues/3695?utm_source=chatgpt.com)) |
| 29 | **TTL exists but expired memories are still returned** | A memory store persisted `expires_at` but none of its read paths enforced expiration. ([github.com](https://github.com/evansenter/agent-memory-store/issues/11?utm_source=chatgpt.com)) |
| 30 | **Old background memory writer overwrites newer truth** | Hermes background reviews operate from conversation snapshots; a delayed reviewer can write stale preferences or undo newer memory changes. ([github.com](https://github.com/NousResearch/hermes-agent/issues/9055?utm_source=chatgpt.com)) |
| 31 | **Memory-file update destroys newer external writes** | A production Hermes agent lost about two hours of accumulated memory because `memory(replace)` flushed an old internal view over a newer `MEMORY.md`. Concurrent sessions make the race worse. ([github.com](https://github.com/NousResearch/hermes-agent/issues/26045?utm_source=chatgpt.com)) |
| 32 | **Protection against that race blocks legitimate writes** | A later drift guard could reject safe append operations because harmless disk/in-memory differences were treated as dangerous conflicts. ([github.com](https://github.com/NousResearch/hermes-agent/issues/42874?utm_source=chatgpt.com)) |
| 33 | **Compaction silently drops not-yet-persisted memories** | Hermes compaction reset the Hindsight turn buffer without first flushing pending retained memories. Turns around the compaction boundary disappeared. ([github.com](https://github.com/NousResearch/hermes-agent/issues/64315?utm_source=chatgpt.com)) |
| 34 | **Agent compaction destroys the human's historical record** | Hermes context compaction replaced earlier visible conversation history with a summary, preventing later inspection of the original work. ([github.com](https://github.com/NousResearch/hermes-agent/issues/70846?utm_source=chatgpt.com)) |
| 35 | **Memory exists but compaction tells the model to ignore it** | Persistent memory was correctly loaded after restart, but handoff framing labeled it background reference, causing the agent to behave as if it had forgotten everything. ([github.com](https://github.com/NousResearch/hermes-agent/issues/17251?utm_source=chatgpt.com)) |
| 36 | **Old task survives compaction and hijacks a new request** | A resumed Hermes session answered an unrelated new question using a stale `Active Task` preserved from an earlier compacted session. ([github.com](https://github.com/NousResearch/hermes-agent/issues/35344?utm_source=chatgpt.com)) |
| 37 | **Compaction summaries become “facts”** | Hermes holographic memory auto-extracted its own compaction handoff summaries into durable fact storage, so stale meta-text later resurfaced as memory. ([github.com](https://github.com/NousResearch/hermes-agent/issues/57682?utm_source=chatgpt.com)) |
| 38 | **Reusing a filter object leaks one user's scope into another write** | Mem0's TypeScript path mutated caller-owned filters; a later add with no user could silently inherit the previous user's identity. ([github.com](https://github.com/mem0ai/mem0/issues/6796?utm_source=chatgpt.com)) |
| 39 | **Caller metadata can manufacture identity scope on creation** | Mem0 allowed metadata to set `user_id`, `agent_id`, or `run_id`, enabling memories to be inserted into another scope. ([github.com](https://github.com/mem0ai/mem0/issues/6655?utm_source=chatgpt.com)) |
| 40 | **Updating metadata can move a memory to another tenant** | Python and TypeScript Mem0 bugs allowed an update to overwrite identity fields, causing a memory to disappear from Alice and appear under Bob. ([github.com](https://github.com/mem0ai/mem0/issues/6342?utm_source=chatgpt.com)) |
| 41 | **Memory authorship gets reassigned during updates** | In multi-actor memory, another actor updating a memory could overwrite its original `actor_id`, breaking provenance and ownership queries. ([github.com](https://github.com/mem0ai/mem0/issues/4490?utm_source=chatgpt.com)) |
| 42 | **Concurrent entity updates silently lose relationships** | Mem0's entity layer used an unsynchronized read-modify-write operation, allowing one concurrent update to erase another memory link. ([github.com](https://github.com/mem0ai/mem0/issues/6243?utm_source=chatgpt.com)) |
| 43 | **Concurrent sessions cause memory writes to fail immediately** | Mem0's SQLite history layer could raise `database is locked` under ordinary concurrent reads/writes because its SQLite configuration lacked the necessary contention handling. ([github.com](https://github.com/mem0ai/mem0/issues/7196?utm_source=chatgpt.com)) |
| 44 | **The canonical state database itself becomes physically corrupt** | A production Hermes `state.db` reached structural SQLite corruption under sustained concurrent activity; operation continued while the database was degraded and required offline reconstruction. ([github.com](https://github.com/NousResearch/hermes-agent/issues/98077?utm_source=chatgpt.com)) |
| 45 | **Reranking cannot recover the actually relevant memory** | Mem0 truncated retrieval to top-k before reranking, meaning a correct candidate ranked at `k+1` could never be rescued by the stronger reranker. ([github.com](https://github.com/mem0ai/mem0/issues/6448?utm_source=chatgpt.com)) |
| 46 | **Configured reranking silently never runs** | A self-hosted Mem0/Hermes integration accepted reranker configuration but failed to pass it through, silently falling back to vector similarity. ([github.com](https://github.com/mem0ai/mem0/issues/6035?utm_source=chatgpt.com)) |
| 47 | **“Continue” causes the agent to forget context** | Hermes' “trivial prompt” optimization classified words such as `continue`, `next`, and `go ahead` as turns that did not need memory retrieval—the exact turns where prior project context is often essential. ([github.com](https://github.com/NousResearch/hermes-agent/issues/79343?utm_source=chatgpt.com)) |
| 48 | **Bad retrieval triggers a memory-search infinite loop** | Low-quality semantic matches caused an agent to repeatedly invoke memory search 10+ times and even ignore explicit user instructions to stop searching memory. ([github.com](https://github.com/NousResearch/hermes-agent/issues/93485?utm_source=chatgpt.com)) |
| 49 | **Memory-use telemetry lies, causing useful memories to look stale** | Hermes' retrieval counter was never incremented through actual agent retrieval paths, so the janitor could classify frequently used facts as “never retrieved.” ([github.com](https://github.com/NousResearch/hermes-agent/issues/78801?utm_source=chatgpt.com)) |
| 50 | **Valid agent state cannot be checkpointed** | LangGraph persistence failed because runtime state contained objects its serializer could not encode. The task could execute and then persistence fail afterward. ([github.com](https://github.com/langchain-ai/langgraph/issues/6781?utm_source=chatgpt.com)) |
| 51 | **Two “sources of truth” disagree about the same memory** | Mem0 had an explicit split-brain bug where vector search contained updated information while Postgres/UI still showed the old version because UPDATE events were not synchronized. ([github.com](https://github.com/mem0ai/mem0/issues/3841?utm_source=chatgpt.com)) |
| 52 | **Deleting vector memory leaves graph memory alive** | Mem0 deletion removed Qdrant state but left corresponding Neo4j nodes and relationships behind. ([github.com](https://github.com/mem0ai/mem0/issues/3245?utm_source=chatgpt.com)) |
| 53 | **Memory update wipes unrelated metadata** | A MongoDB backend replaced the entire metadata payload during an update, removing source, category, context, and other important fields. ([github.com](https://github.com/mem0ai/mem0/issues/3966?utm_source=chatgpt.com)) |
| 54 | **Cleanup itself becomes incomplete beyond scale thresholds** | Entity cleanup scanned at most 10,000 entities, so systems above that scale could silently leave stale relationships behind. ([github.com](https://github.com/mem0ai/mem0/issues/4988?utm_source=chatgpt.com)) |
| 55 | **Session continuity becomes hugely expensive and eventually degrades reasoning** | One heavy Hermes production user reported ~2.6M tokens of overhead during a 12-hour session due to repeated full-history replay; stale context later contributed to incorrect beliefs about the execution environment. ([github.com](https://github.com/NousResearch/hermes-agent/issues/5563?utm_source=chatgpt.com)) |

There are also several important real-world behavioral failures that cut across implementations.

## Memory rot is not hypothetical

Letta's July 2026 evaluation is particularly useful because its scenarios are based on actual long-lived internal agents rather than clean toy stores. They found that real memory naturally becomes **messy, duplicated, stale, contradictory, and poorly structured**. Their benchmark specifically tests memory hygiene, generalizing corrections, retrieval, and adherence because those are failures they see in real stateful agents. ([letta.one](https://letta.one/blog/evaluating-memory-in-production-agents/?utm_source=chatgpt.com))

Their observation about corrections is particularly important:

```text
User corrects agent
        ↓
weak memory behavior
        ↓
append another dated note

instead of

identify governing rule
        ↓
repair durable rule
        ↓
supersede contradictory old state
```

That exactly matches the consolidation problem we identified earlier. ([letta.one](https://letta.one/blog/evaluating-memory-in-production-agents/?utm_source=chatgpt.com))

## Practitioners keep hitting recursive contamination

A practitioner who reported roughly 40,000 turns said about **30% of stored memories were duplicates, later contradicted facts, or the agent's own outputs re-ingested as facts**. They eventually stopped allowing agents to write directly to long-term memory. This is anecdotal rather than a controlled benchmark, but it lines up closely with the large Mem0 production audit from the previous pass. ([reddit.com](https://www.reddit.com/r/AgentsOfAI/comments/1t47qbf/whats_your_actual_agent_memory_stack_right_now/?utm_source=chatgpt.com))

That strongly reinforces:

> **The write path is at least as important as retrieval.**

## Vector memory develops semantic rot

Another practitioner describes old embeddings continuing to retrieve months-old facts alongside replacements because semantic similarity is almost unchanged. Their operational solution was explicit `supersedes` metadata plus periodic deduplication. ([reddit.com](https://www.reddit.com/r/LLMDevs/comments/1w6lnw3/how_are_you_handling_semantic_rot_in_longterm/?utm_source=chatgpt.com))

Again, that is almost exactly the temporal model we independently derived.

---

# The complete failure space is now clearer

After both passes, I would divide real agent-memory failures into **14 major families**:

| Failure family | Examples |
|---|---|
| **1. Formation corruption** | noise stored, hallucinated facts, agent output re-ingested, system prompts stored |
| **2. Omission** | important fact never written, pending memory lost during compaction |
| **3. Temporal corruption** | stale truth remains active, old and new facts coexist, expired memory recalled |
| **4. Provenance corruption** | source message unavailable, author identity overwritten, inference appears factual |
| **5. Scope/identity corruption** | cross-user bleed, user IDs mutated, entity merges cross boundaries |
| **6. Mutation corruption** | lost concurrent writes, stale reviewers overwrite new state, metadata disappears |
| **7. Deletion/forgetting corruption** | deletion incomplete, deletion too broad, deleted memories remain indexed |
| **8. Derived-state drift** | text differs from vectors, graph differs from vector store, UI differs from backend |
| **9. Retrieval failure** | wrong memory, missing memory, truncated candidate pool, reranker disabled |
| **10. Retrieval-policy failure** | memory skipped when needed, searched repeatedly when useless |
| **11. Adherence failure** | correct memory retrieved but ignored or treated as malicious |
| **12. Context/compaction corruption** | useful details disappear, stale task becomes current, summaries become durable facts |
| **13. Durability/transaction failure** | partial commits, serialization errors, locking, restart loss, DB corruption |
| **14. Lifecycle/hygiene failure** | uncontrolled growth, duplication, bad decay metrics, stale facts never retired |

That is a much more useful model than talking about “recall accuracy.”

---

# One failure deserves special attention for SF-SML

The Hermes `MEMORY.md` incident is probably the most relevant real-world example for our proposed architecture.

They effectively had:

```text
MEMORY.md
+
memory tool internal representation
+
patch/shell writers
+
multiple sessions
```

and more than one component believed it could authoritatively rewrite the file.

Result:

> **real production data loss.** ([github.com](https://github.com/NousResearch/hermes-agent/issues/26045?utm_source=chatgpt.com))

This teaches us something crucial.

Our SF-SML design must **not** merely say:

> One authoritative file.

It must say:

> **One authoritative file AND exactly one authoritative mutation path.**

That means:

```text
                     READERS
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Agent A        Agent B       Indexer
        │              │
        └────── memory proposals ─────┘
                       │
                       ▼
                MEMORY MANAGER
              single coordinated writer
                       │
                version / CAS check
                       │
                 atomic commit
                       │
                       ▼
                MEMORY_LEDGER
```

No:

```text
agent edits ledger
shell edits ledger
memory manager edits ledger
background consolidator edits ledger
another session edits ledger
```

independently.

That is now supported by a documented real production failure, not just distributed-systems theory. ([github.com](https://github.com/NousResearch/hermes-agent/issues/26045?utm_source=chatgpt.com))

---

# Another major update: compaction and memory must be separate

The Hermes failures show that compaction can:

- erase original history,
- lose pending memory,
- incorrectly demote durable memory,
- preserve old tasks as live intent,
- and generate summaries that later become “facts.” ([github.com](https://github.com/NousResearch/hermes-agent/issues/70846?utm_source=chatgpt.com))

Therefore we should establish a hard invariant:

> **Context compaction must never be the authoritative memory-writing mechanism.**

Compaction is a working-context optimization.

Memory is durable state.

They may communicate, but:

```text
conversation evidence
       ↓
memory formation pipeline
       ↓
authoritative memory
```

must remain separate from:

```text
conversation
       ↓
context compression
       ↓
temporary working summary
```

A summary can be proposed as evidence for a memory, but it should not automatically become authoritative fact.

---

# Deletion needs the same rigor as writing

Earlier we focused heavily on `remember()`.

Real-life systems show `forget()` is equally dangerous.

We've now seen:

```text
delete too little
delete too much
delete only first page
delete vector but not graph
delete UI copy but search copy survives
expiration recorded but ignored
```

all in real implementations. ([github.com](https://github.com/mem0ai/mem0/issues/3928?utm_source=chatgpt.com))

Therefore deletion must be treated as:

```text
request deletion
      ↓
resolve exact scope
      ↓
atomic authoritative mutation
      ↓
invalidate derived indexes
      ↓
rebuild/update projections
      ↓
search for supposedly deleted data
      ↓
verify absence
```

not:

```text
DELETE
→ success
```

---

# A perfect-memory V1 now needs stronger invariants

Based on the real failures—not theory—I would add these to SF-SML:

```text
ONE AUTHORITATIVE LEDGER

ONE AUTHORITATIVE WRITER

ATOMIC VERSIONED WRITES

READ-AFTER-WRITE VERIFICATION

READ-AFTER-DELETE VERIFICATION

NO DIRECT EMBEDDING AUTHORITY

NO DIRECT SUMMARY AUTHORITY

NO DIRECT AGENT-GENERATED FACT AUTHORITY

IMMUTABLE SCOPE / IDENTITY FIELDS

PROVENANCE FOR EVERY CONSEQUENTIAL MEMORY

EXPLICIT SUPERSESSION

FRESHNESS / VALIDITY

PENDING-WRITE FLUSH BEFORE COMPACTION OR SHUTDOWN

CONTEXT COMPACTION SEPARATE FROM DURABLE MEMORY

INDEX REBUILD TEST

RESTART RECOVERY TEST

CONCURRENCY TEST

CORRUPTION / PARTIAL-WRITE RECOVERY TEST

PERIODIC MEMORY HYGIENE
```

That is now much better grounded.

---

## Where I think we've reached saturation

At this point, continued searching is mostly finding vendor-specific instances that collapse into those 14 families.

The important discovery is that the largest real-world risks are **not primarily “the agent forgot something.”**

They are:

> **The agent remembered something that was wrong.**

> **The memory system silently changed who a memory belonged to.**

> **An old truth was retrieved as current truth.**

> **One representation changed while another stayed stale.**

> **A maintenance operation destroyed or corrupted memory.**

> **The system said a write/delete succeeded when durable reality disagreed.**

> **The agent's own generated artifacts became feedback into future truth.**

That substantially reinforces the architecture we were converging on:

> **Canonical evidence/history → controlled memory formation → authoritative temporal state → rebuildable indexes → selective retrieval → action → observation → reconciliation → verified maintenance.**

And it suggests that for SF-SML, **single-file is useful, but single-writer + single-authority + verified mutation is what actually prevents the failures people are experiencing in production.**