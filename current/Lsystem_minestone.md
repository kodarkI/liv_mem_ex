# Living Memory System Milestone Roadmap

**Status:** Updated roadmap — use this version instead of the original A–E list
**Scope:** Operational foundation, retrieval readiness, and controlled execution
**Decision:** Follow the roadmap direction, but follow the revised gates and
dependencies in this document.
**Current posture:** Not ready for broad feature expansion or production
rollout. Step 2E and unrestricted concurrency remain deferred.

> The filename preserves the requested `system_minestone.md` spelling.
>
> **Terminology:** Step 2E means the project's controlled operational rollout
> gate. Milestone E below means the technical controlled-concurrency milestone;
> neither is authorized merely because the other is complete.

## 1. Why the roadmap was updated

The original roadmap is directionally correct, but it starts too late. Current
repository evidence shows that the project still needs:

- a reproducible validation baseline;
- reconciliation of current nonterminal mutation/recovery work;
- explicit RecoveryResponse and closeout semantics;
- a clear Dispatcher ownership model;
- live-data validation of retrieval and lineage;
- canonical State, Relationship, and Event Relation integration.

The older completion summaries and percentage claims must not override current
runtime evidence. A feature is not fully complete merely because its schema,
code, or fixture tests exist.

## 2. Status vocabulary

Each milestone must report these dimensions separately:

- **Specified** — defined in architecture or governance documents.
- **Implemented** — production code exists.
- **Tested** — automated tests pass in a reproducible environment.
- **Observed** — behavior passes against current repository data or a real MCP
  lifecycle.
- **Independently reviewed** — a separate validation review cleared the result.
- **Release-authorized** — Owner/Architecture approval exists for rollout.

Do not use `complete` when a critical behavior is only specified, implemented,
or fixture-tested.

## 3. Authority rules

The following boundaries apply to every milestone:

| Concern | Authority |
|---|---|
| User and agent evidence | Source Experience records |
| Current State | Governed State records |
| Relationships | Governed Relationship records |
| History | Append-only History/Event records |
| Mutation execution | Mutation jobs and workers |
| Recovery | Step2D recovery history and source evidence |
| Active continuity position | Validated checkpoint and recovery state |
| Present-facing context | Derived continuity projection |
| Recall | Governed source-based retrieval |
| Attention | Governed Attention Allocation records |
| Audit delivery | Audit Outbox and Dispatcher |

Projections, summaries, readiness, attention, and recall scores must not
replace source truth, authorize recovery, or silently promote a record.

## 4. Current assessment

| Area | Current assessment | Decision |
|---|---|---|
| MCP admission and durable jobs | Substantial implementation | Stabilize and verify |
| Step2D recovery | Substantial bounded implementation | Complete live gate |
| Response closeout | Implemented in bounded scope | Verify all invariants |
| State | Standalone implementation, split authority risk | Consolidate before integration |
| Relationship | Standalone lifecycle implementation, schema risks | Consolidate before integration |
| Event Relations | Records exist, producer/reader mismatch remains | Fix before history claims |
| Recall | Multiple useful paths, live-data defects remain | Keep Milestone 4 open |
| ContextBootstrap | Implemented and unit-tested, not mandatory runtime path | Defer Phase 1B |
| Attention | Isolated mechanics exist | Integrate later |
| Skills and learning loop | Mostly design-only | Future work |

## 5. Dependency graph

The critical path is:

`A → B → C → D`

Then:

- **G** (retrieval/recall closure) may proceed after **C** and the required
  State/Relationship contract work; it does not require broad concurrency
  rollout.
- **E** (controlled concurrency) requires **D** and the State/Relationship
  authority gate in **F**, and should be started only if measured throughput
  requires it.
- State, Relationship, and Event Relation contract work may proceed in parallel
  with B and C, but canonical MCP integration should wait for the recording
  contract to stabilize.
- ContextBootstrap Phase 1B, Attention integration, Skills, and advanced
  reasoning follow reliable source, recovery, and retrieval behavior.

## Milestone A — Trust the Evidence

### Objective

Create a clean, reproducible validation baseline and establish the real current
state before implementing more features.

### Work

- Isolate tests from live `.living-memory/data/` runtime artifacts.
- Fix import/path configuration and incomplete test collection.
- Replace tests that only print or return values with assertions.
- Record command, environment, exit code, duration, and test artifacts.
- Census active and archived Experiences, Entities, Relationships, States,
  encoded units, lineage, recovery, and mutation jobs.
- Classify current `UNKNOWN`, `PROCESSING`, and `CLOSEOUT_PENDING` work using
  the canonical recovery path.
- Reconcile original operation IDs before retrying anything.

### Exit criteria

- Required test targets run to completion in a clean environment.
- Environment failures are distinguishable from code failures.
- No active turn remains indefinitely stuck in `PROCESSING`.
- Every nonterminal operation has an authoritative classification.
- No manual JSON repair or history rewriting is used.
- A live-data baseline is recorded and independently reviewable.

### Current gate status

**Not passed.** As of this roadmap review, the current runtime audit observed an
active turn in `PROCESSING`,
nonterminal `STARTED` and `PROCESSING` jobs, `CLOSEOUT_PENDING` work,
`UNKNOWN` jobs, and a pending audit-outbox backlog. These are evidence to
reconcile, not permission to retry or manually edit records. The exact counts
must be regenerated by the Milestone A census because the live store changes.

The supporting acceptance authority is
`.living-memory/docs/MILESTONE_4_OPERATIONAL_REMEDIATION_PLAN.md` together with
the current Step2D/runtime status projection. The status is intentionally
volatile and must be re-read when Milestone A begins.

The census must include, at minimum:

- `active_turn.json` and `current_turn.json`;
- mutation jobs, leases, worker-start reservations, and worker processes;
- Step2D history, recovery backlog, and readiness projection;
- audit-outbox records and delivery status;
- transaction-journal/prepared-transaction state;
- Event Relation endpoint-shape compatibility;
- active and archived Experiences, Entities, Relationships, States, encoded
  units, and all required lineage fields.

### Non-goals

No new feature, concurrency refactor, rollout, deletion, or broad cleanup.

## Milestone B — Complete Operational Design

### Objective

Define the durable lifecycle before changing worker execution behavior.

### Required design areas

- Dispatcher lifecycle: admission, queueing, claim, lease, execution, retry,
  terminal update, and observability.
- RecoveryResponse lifecycle: response pairing, provenance, unavailable
  outcomes, reconstruction, and idempotent replay.
- Step2D integration: watchdog, fencing, reconciliation, finalization, and
  readiness ownership.
- Outbox ownership: publication and acknowledgement must remain separate from
  source recording, job completion, and recovery closure.
- Outbox state machine: define `PENDING`, `CLAIMED`/`IN_FLIGHT`, `PUBLISHED` or
  `ACKNOWLEDGED`, `COMPLETE`, retry, and dead-letter/manual-review states.
- FIFO semantics, lock ordering, conflict scopes, and future worker limits.

### Required invariants

- `ACCEPTED` is not `SUCCEEDED`.
- Source persistence alone is not closeout completion.
- `CLOSEOUT_PENDING` is nonterminal and recovery-eligible.
- `UNKNOWN` is not converted to success without source reconciliation.
- Exact retries reuse the original operation identity.
- A late worker is fenced by status, version, lease, or token checks.
- Recovery never invents a response.
- Derived `FINALIZED` and `FULLY_AUDITED` projections cannot replace source
  authority.
- `FINALIZED` requires source persistence, recovery closure or an explicit
  not-required result, and successful job completion.
- `FULLY_AUDITED` additionally requires `OUTBOX_COMPLETE`; outbox delivery
  cannot promote a job or bypass response closeout.
- Outbox leases, retries, duplicate acknowledgements, and dead-letter behavior
  are idempotent and observable.

### Exit criteria

Architecture, Project Manager, Builder, Validator, and Owner agree on the
contracts, ownership boundaries, state vocabulary, and acceptance matrix. The
agreement is recorded in a durable acceptance artifact containing the
transition table, ownership matrix, retry/idempotency rules, outbox semantics,
failure matrix, named approvers, and evidence references.

## Milestone C — Reliable Execution

### Objective

Implement a durable Dispatcher without changing concurrency semantics first.

### Scope

- Implement the Dispatcher with `max_workers=1`.
- Preserve the durable mutation job store, version fencing, leases, and Step2D.
- Make progress independent of a new MCP request.
- Implement RecoveryResponse through the shared closeout finalizer.
- Preserve exact user/agent lineage and response provenance.
- Keep the existing global worker lock as the safe fallback.

### Exit criteria

- Every admitted job is claimed, completed, or recoverable.
- Startup failures are visible and do not block FIFO forever.
- Worker crashes do not duplicate source Experiences.
- Response persistence, recovery evidence, receipt, active-turn completion,
  current-turn invalidation, and terminal closeout remain coupled.
- Replays and reconciliations are idempotent.
- A real MCP begin-turn-to-response lifecycle passes in a clean environment.

### Non-goals

No unrestricted parallel workers, lock removal, new distributed lease service,
or Step 2E rollout.

## Milestone D — Recovery Correctness

### Objective

Prove Dispatcher and Step2D behavior together under failure and interruption.

### Required tests

- worker crash tests;
- startup-timeout tests;
- claim and lease-expiry tests;
- `UNKNOWN` operation classification and reconciliation;
- late-worker fencing;
- duplicate admission and replay;
- source-persisted but incomplete closeout;
- unavailable-response recovery;
- active-turn interruption and completion;
- recovery backlog and readiness transitions;
- outbox publication after, not before, finalization prerequisites.

### Exit criteria

- Step2D and Dispatcher pass together against real operational evidence.
- `UNKNOWN` remains durable and recoverable.
- No late or stale owner can mutate a newer job version.
- No unresolved accepted-baseline recovery backlog remains, or every remaining
  item has an explicit manual-review owner and status.
- Recovery history remains append-only.
- Independent validation clears the failure matrix.

### Current gate status

**Not passed until Milestone A is complete and the combined Dispatcher/Step2D
failure matrix has been observed against real operational evidence.**

## Milestone E — Controlled Concurrency

### Objective

Increase throughput only after serialized execution and recovery are proven.

### Scope

- Concurrent transport with a single response writer.
- Bounded request execution.
- Bounded worker pools behind feature flags.
- Conflict-aware scoped serialization.
- Parallel execution only for proven-independent jobs.
- Global serialization fallback for unclassified or sensitive operations.

### Required safeguards

- Preserve FIFO where jobs share a lineage or protocol resource.
- Preserve atomic job-store updates and lock ordering.
- Separate transport concurrency from mutation concurrency.
- Measure latency, duplicate prevention, stale leases, recovery backlog, and
  closeout failures.
- Define rollback and disable conditions before enabling parallel workers.

### Exit criteria

- Concurrency tests pass without malformed JSON/JSONL or lost history.
- Independent jobs can run in parallel without affecting shared active-turn,
  state, relationship, or closeout resources.
- Crash recovery remains correct under concurrent load.
- Feature-flag rollback is demonstrated.
- Explicit rollout approval exists.

Controlled concurrency is not a prerequisite for retrieval correctness. It is a
separate, opt-in performance milestone and must not be used to bypass unresolved
source, recovery, State, Relationship, or outbox defects.

## 6. Follow-on feature gates

The A–E operational roadmap is necessary but does not complete the whole v5
architecture. These feature tracks follow the operational gates.

### F — Canonical State, Relationship, and History Integration

- Select one authoritative State store/API.
- Select one Relationship schema/API.
- Standardize Event Relation endpoint fields across writers, readers,
  traversers, validators, and tests.
- Decide how existing records using the current endpoint fields are migrated or
  read compatibly; do not silently discard them.
- Ensure MCP recording creates the required typed response relation
  idempotently where the contract requires it.
- Attach source Experience evidence to every State and Relationship change.
- Prove current/historical State and Relationship behavior on real data.
- Prove that unreadable or unprovenanced relations cannot support a history or
  recall claim.
- Complete recording-time classification/routing so a canonical Experience can
  produce governed State, Relationship, Direction, and Pattern proposals when
  the evidence warrants them; proposals must not bypass Governance.

State and Relationship are not design-only and do not need to wait to exist.
Their **canonical MCP integration** does need a stable recording contract.

### G — Retrieval and Recall Closure

Keep the retrieval gate open until the real-data remediation plan passes:

1. archived Experience indexing;
2. historical and superseded governance filtering;
3. null/malformed Relationship evidence;
4. null/malformed Graph evidence;
5. Entity reverse-index coverage;
6. ranking and false-positive rejection;
7. conversation-chain reconstruction;
8. stale encoded-unit reconciliation.

Then consolidate the recall surface behind one bounded, governed retrieval
contract. Do not rebuild recall from zero and do not use a projection as source
authority.

Pattern detection remains an intermediate, evidence-producing capability. It
must be bounded, source-linked, and tested as a proposal signal; it must not be
treated as automatic Capability or Skill creation.

### H — ContextBootstrap and Attention Integration

- Make the ContextBootstrap runtime boundary explicit.
- Keep it read-only and non-authorizing.
- Integrate current State, Relationships, Direction, recovery, and Recall.
- Use the current Attention vocabulary:
  `NOT_ATTENDING`, `MONITOR`, `QUEUED`, `FOCUSED`, `URGENT`.
- Make attention govern activation and focus without changing source truth.

### I — Skills and Closed Learning Loop

Implement only after reliable Recall, State, Relationships, History, Direction,
and Attention exist:

- Candidate Capability;
- capability discovery inquiry;
- Skill record and versioning;
- Skill invocation evidence;
- Candidate Experience from material Skill outcomes;
- outcome attribution;
- Skill lineage and provenance;
- Skill coordination;
- governed promotion, rejection, retirement, and evolution.

A pattern is not automatically a capability, and a successful outcome is not
automatically proof that a Skill caused it.

## 7. What to follow now

Follow the roadmap **after updating it as above**.

### Immediate order

1. Milestone A: validation baseline and live operational census.
2. Milestone B: Dispatcher, RecoveryResponse, and Step2D contracts.
3. Milestone C: serialized Dispatcher and shared closeout finalizer.
4. Milestone D: failure, recovery, and real MCP verification.
5. Milestone F contract/schema consolidation for State, Relationship, and
   Event Relations.
6. Retrieval/Recall live-data closure.
7. Milestone E controlled concurrency, only if throughput still requires it.
8. ContextBootstrap and Attention integration.
9. Skills and advanced learning-loop work.

This ordering differs slightly from the original list: **State/Relationship
contract consolidation should happen before broad concurrency**, because shared
state and lineage resources must have one authority before parallel execution
is enabled. Their full MCP integration should still wait for Milestones B–D.

## 8. Completion policy

A milestone can be closed only when its critical behavior is:

1. implemented;
2. tested in a reproducible environment;
3. observed against the required live or clean-room evidence;
4. independently reviewed;
5. explicitly accepted by the appropriate owner.

Old percentage reports, schemas, task checkboxes, generated summaries, and
fixture-only passes are supporting evidence only. They cannot close a gate.

## 9. Final decision

**Update the roadmap; do not follow the original A–E version unchanged.**

The original roadmap correctly identifies validation, Dispatcher design,
reliable execution, recovery testing, and controlled concurrency. The updated
version adds the missing authority reset, RecoveryResponse contract, live-data
retrieval gate, State/Relationship integration gate, and explicit boundaries
for ContextBootstrap, Attention, and Skills.

The project is not blocked from all feature work. It is blocked from claiming a
coherent, production-ready Living Memory runtime until the operational and
live-validation gates pass.
