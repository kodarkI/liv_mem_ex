 Executive Summary

  Your concern is valid: you have several priorities because the project is mixing three different completion levels:

    1. Architecture specified
    2. Component implemented in isolation
    3. Integrated and proven through the canonical MCP runtime

  The v5 architecture is not fully implemented as one connected runtime. However, it is also incorrect to say that everything
   outside MCP/recording is waiting. The more accurate picture is:

    │ State, Relationship, History, Recall, Attention, and Pattern components already exist independently. Their canonical
    │ end-to-end integration is incomplete because the MCP recording path currently stops at source Experience recording and
    │ closeout.

  The main operational foundation should be stabilized before broad feature expansion.


  ────────────────────────────────────────────────────────────────────────────────────

  Status Model

  I am using these meanings:

    • Done — implemented, tested, and reasonably usable in its intended scope.
    • Partial — substantial implementation exists, but integration, schema consistency, or validation is incomplete.
    • Pending / Design-only — mostly specified in documents, with little or no runtime implementation.
    • Blocked at MCP path — the component exists, but the canonical MCP recording path does not invoke it.


  ─────────────────────────────────────────────────────────

  v5 Architecture Gap Analysis

  | v5 component | Status | What exists | What is missing |
  |---|---|---|---|
  | Current Experience / Conversation | Partial | Experiences are persisted with IDs, source, timestamps, truth, retention,
  provenance, and lineage. MCP user/response recording exists. | The MCP path does not continue into all downstream memory
  operations. |
  | Memory Control | Partial | memory_control.py receives Experiences, routes them, performs retention-related decisions, and
   exposes recall helpers. | State-change and relationship-change detection is heuristic and incomplete.
  identify_source_entity and establish_intent are currently largely detection/logging paths rather than complete mutations. |
  | Memory Governance | Partial | Retention, truth status, entity proposal, relationship progression, state snapshots,
  relevance, and activation-related methods exist in memory_governance.py. | Governance is not consistently reached from the
  canonical MCP recording flow. Some decisions mutate existing Experience metadata, so "immutable Experience" needs a clearer
   distinction between immutable source content and mutable governance metadata. |
  | Entity | Partial | Entity proposal, identity assertions, duplicate detection, lifecycle status, and initial state
  creation exist. | Automatically routed identity detection does not consistently create or resolve entities. |
  | State | Partial | State snapshots and supersession chains work through direct APIs and tests. | There are two stores:
  data/states and data/state_snapshots. The canonical authority is not fully consolidated. MCP recording does not create
  state updates. |
  | Relationship | Partial | Relationship proposal, evidence-based progression, activation/deactivation, dormant handling,
  and V1 lifecycle operations exist. | MCP recording does not create or advance relationships. Some consumers use
  subject/object, while current records use subject_entity_id/object_entity_id. |
  | History | Partial, with a current defect | History managers, state history, recovery history, and event records exist. |
  Event Relation producers write source_event_id/target_event_id, while some readers and traversers still read legacy
  source_event/target. This breaks reliable history traversal. |
  | Historical Integration | Partial | Chain traversal, path inference, event relation creation, integration assertions, and
  pattern-related logic exist. | Current writer/reader schema mismatch means the historical graph is not reliably queryable
  end to end. |
  | Living Memory projection | Partial / relatively strong read side | ContinuityLoader and canonical continuity projections
  combine identity, state, relationships, directions, history, attention, recovery, and recall-related information. | A
  projection cannot repair missing or incorrect write-side records. |
  | Direction / Intent | Partial | Explicit Direction creation and matching exist. | Automatic intent detection is not
  consistently converted into durable Direction records. |
  | Recall | Partial | Governed recall, bounded retrieval, relevance evaluation, semantic matching, temporal logic,
  graph/state scoring, and provenance-related mechanisms exist. | There are multiple recall paths. Automatic relevance is not
   consistently wired into the actual host/LLM boundary, and real-data validation still identifies retrieval defects. |
  | Attention Control | Standalone implementation | Allocation, priority, focus budget, release, reactivation, and trigger
  logic exist in attention_control.py. | Attention is not proven to control normal recall, capability discovery, or Skill
  execution. |
  | Next State | Partial | State changes, relationship changes, context membership, recovery markers, and continuity
  snapshots provide pieces of the next state. | There is no single orchestrator that computes the complete v5 "next state"
  envelope. |
  | Self-auditing | Partial | Audit classes and integrity checks exist. | Normal runtime scheduling and integration are not
  established. |
  | Pattern Recognition | Partial | Pattern records and recurring-pattern detection exist. | The bounded, attention-governed
  discovery process is not complete, and current tests are not reliable enough as acceptance evidence. |
  | Skills | Design-only | Skill schema and extensive design/reference documents exist. | No complete Skill store, lifecycle
  manager, versioning, execution registry, or coordination runtime was found. |
  | Capability Emergence | Design-only after the pattern stage | Pattern assertions and a capability-inference placeholder
  exist. | No durable Candidate Capability lifecycle exists. |
  | Skill Evolution | Design-only | Lifecycle, provenance, attribution, and evolution rules are specified. | No runtime
  implementation for promotion, revision, split, merge, retirement, or outcome-based evolution. |
  | Capability Discovery | Design-only | Attention-governed discovery is specified. | No operational bounded discovery
  service exists. |
  | Outcome Attribution | Design-only | Attribution rules are documented. | No runtime mechanism determines whether a Skill
  actually caused an outcome. |
  | Applicable Skill Sets | Design-only | Complementary, conflicting, prerequisite, and independent capability categories are
   specified. | No Skill coordination planner or execution policy exists. |
  | Closed Learning Loop | Design-only | The full loop is described in v5. | The runtime does not connect Experience →
  Pattern → Candidate Capability → Skill → Outcome → Evidence → Skill evolution. |


  ────────────────────────────────────────────────────────────────────────────────────────────

  What Is Actually Done

  The following parts are meaningfully implemented, although not necessarily fully integrated:

  1. Foundational records

  These are not merely ideas:

    • Experience records
    • Entity records
    • State records
    • Relationship records
    • Direction records
    • Retention decisions
    • Pattern assertions
    • Attention allocations
    • Recovery records
    • Event and history records

  This is supported by the existing core modules, persisted data, and focused tests.

  2. State lifecycle mechanics

  State snapshots have real implementation for:

    • current versus historical status;
    • supersession;
    • state history;
    • source Experience references;
    • state progression tests.

  The important qualification is that there are two state implementations:

    • core/state_manager.py using data/state_snapshots;
    • MemoryGovernance using data/states.

  Therefore, state is implemented but not yet cleanly authoritative.

  3. Relationship lifecycle mechanics

  Relationship lifecycle behavior is substantially implemented:

  POSSIBLE → CANDIDATE → SUPPORTED → ESTABLISHED → ACTIVE/DORMANT/HISTORICAL

  Evidence-based advancement and isolated V1 relationship operations exist.

  The missing part is not the basic relationship algorithm. It is:

    • canonical schema consistency;
    • integration into MCP recording;
    • reliable relevance activation;
    • real-data verification.

  4. Recall mechanisms

  Recall is not waiting for implementation from zero. There are already:

    • bounded recall;
    • relevance filtering;
    • semantic matching;
    • temporal continuity;
    • graph traversal;
    • state and relationship signals;
    • historical integration;
    • provenance-related output.

  The main problems are consolidation and validation, not absence.

  5. MCP admission and recording foundation

  The MCP layer already has substantial functionality:

    • JSON-RPC tool handling;
    • mutation admission;
    • deterministic operation identity;
    • receipt and operation IDs;
    • durable mutation jobs;
    • worker subprocess execution;
    • response recording;
    • Step 2D recovery;
    • readiness projection;
    • closeout handling;
    • duplicate/replay protections.

  However, this is not the same as saying the MCP system is production-complete. Current evidence still shows:

    • unresolved operational jobs;
    • pending recovery/closeout evidence;
    • incomplete live validation;
    • MCP responses without typed Event Relations in some cases;
    • no complete dispatcher execution model.

  6. ContextBootstrap Phase 1A

  ContextBootstrap is implemented and has dedicated tests for snapshot building and validation.

  But it is currently:

    │ Specified and unit-tested, but not mandatory in the production turn path.

  AgentMemoryInterface still directly loads continuity rather than requiring every host path to pass through
  ContextBootstrap.


  ───────────────────────────────────────────────────────────────────────────────────────────

  What Is Not Done

  The following are genuinely pending rather than merely waiting for a small integration fix:

  1. Dispatcher

  The durable Dispatcher execution model is not yet a completed runtime subsystem.

  Current worker launching and job handling exist, but a complete Dispatcher still needs to define and implement:

    • queue ownership;
    • claiming;
    • FIFO behavior;
    • lease handling;
    • retry scheduling;
    • worker-start failure;
    • concurrency limits;
    • conflict scopes;
    • Step 2D coordination;
    • recovery redrive;
    • observability.

  Start with max_workers=1. Do not begin with parallel mutation execution.

  2. MCP-to-domain integration

  The current MCP worker intentionally performs a narrower workflow:

    • persist user or agent Experience;
    • manage active-turn and recovery records;
    • complete response closeout;
    • create derived representations.

  It does not consistently invoke:

    • MemoryControl.route_experience();
    • state snapshot creation;
    • relationship proposal or advancement;
    • Direction creation;
    • typed Event Relation creation.

  This means the canonical MCP path currently records source facts but does not fully produce the broader Living Memory
  model.

  3. Canonical Event Relation behavior

  This is a concrete correctness gap.

  The current system has a producer/consumer mismatch:

    • writers use source_event_id and target_event_id;
    • readers and traversal code still use legacy names in places.

  Additionally, many MCP-recorded responses contain in_response_to but do not have a corresponding typed CONTINUES Event
  Relation.

  This should be fixed before claiming that conversation and causal history are operational.

  4. Canonical state authority

  The project needs one explicit decision:

    │ Is data/states or data/state_snapshots the authoritative state store?

  Until that is resolved, state may appear correct in one path and stale or incomplete in another.

  5. Skills and the learning loop

  These remain future work:

    • Candidate Capability;
    • capability discovery;
    • Skill lifecycle;
    • Skill invocation;
    • outcome evidence;
    • outcome attribution;
    • Skill lineage;
    • Skill coordination;
    • Skill evolution.
  This is the largest section of v5 that is still design-level.


  ─────────────────────────────────────────────────────────────

  Are State and Relationship Waiting for MCP and Recording?

  Short answer

  Partly, but not completely.

  They are not waiting for implementation

  State and Relationship already have standalone implementations and tests.

  You can work on them independently for:

    • schema consolidation;
    • authority decisions;
    • lifecycle validation;
    • source linkage;
    • retrieval behavior;
    • negative cases;
    • real-data validation.

  They are waiting for canonical integration

  For the actual product flow, they are waiting for the recording foundation to provide a stable integration seam.

  The current relationship is:

    │ MCP recording creates the Experience, but the downstream state/relationship orchestration is not yet attached to that
    │ recording path.

  Therefore:

  | Question | Answer |
  |---|---|
  | Do State and Relationship need MCP before they can exist? | No |
  | Do they need a stable canonical recording contract for end-to-end behavior? | Yes |
  | Should you redesign State and Relationship while recording is still changing? | No |
  | Can you consolidate schemas and write integration tests now? | Yes |
  | Should full MCP integration happen before the Dispatcher/recording contract stabilizes? | Preferably no |

  The right strategy is not to pause State and Relationship entirely. It is to separate their work into:

    1. Independent correctness work now
    2. Canonical MCP integration after the recording contract stabilizes


  ────────────────────────────────────────────────────────────────────────────────────

  Recommended Priority

  Priority 0 — Make validation trustworthy

  This is the first priority because current completion claims are difficult to trust.

  Fix:

    • test import/path configuration;
    • isolated temporary data roots;
    • tests that print failures but still pass;
    • tests that return values instead of asserting;
    • stale runtime artifact interference;
    • full-suite collection failures.

  The authoritative operational documents explicitly state that fixture-only tests and old completion reports are
  insufficient. The project needs clean evidence from:

    • current repository data;
    • active and archived Experiences;
    • Entities;
    • States;
    • Relationships;
    • encoded units;
    • real MCP execution.

  Exit condition: a clean validation command can run deterministically and produce pass/fail evidence.


  ────────────────────────────────────────────────────────────────────────

  Priority 1 — Stabilize current MCP/recording/Step2D operations

  Before adding new architecture, resolve the current operational backlog:

    • current UNKNOWN jobs;
    • CLOSEOUT_PENDING work;
    • active PROCESSING jobs;
    • recovery backlog;
    • any valid or invalid worker ownership evidence.

  This is operational cleanup and verification, not a new feature.

  Exit condition: the system can demonstrate one clean live lifecycle:

  begin_turn → worker claim → user Experience → record_response → response Experience → recovery evidence → closeout →
  completed active turn


  ──────────────────────────────────────────────────

  Priority 2 — Design the Dispatcher execution model

  The Dispatcher design should include:

    • durable queue ownership;
    • FIFO definition;
    • claim and lease semantics;
    • worker-start reservation;
    • Step 2D watchdog behavior;
    • RecoveryResponse behavior;
    • retry and redrive rules;
    • closeout ownership;
    • lock ordering;
    • future concurrency scopes;
    • readiness effects.

  Include RecoveryResponse in this design rather than designing it as an unrelated feature later.

  Do not begin with an asyncio rewrite. First define the durable execution contract.


  ──────────────────────────────────────────────────────────────────────────────────

  Priority 3 — Implement a serialized Dispatcher

  Implement the Dispatcher with:

    • one worker;
    • existing durable job store;
    • existing version fencing;
    • existing lease fields;
    • existing Step 2D recovery;
    • no removal of the global lock yet.

  The goal is to prove:

    • every admitted job gets claimed or becomes recoverable;
    • startup failures are visible;
    • worker crashes do not corrupt state;
    • recovery does not duplicate records;
    • the Dispatcher progresses without requiring a new MCP request.

  Only after this passes should you consider multiple workers or concurrent request execution.


  ────────────────────────────────────────────────────────────────────────────────────────────

  Priority 4 — Verify Dispatcher and Step2D together

  This should be treated as part of Dispatcher completion, not a separate optional phase.

  Test:

    • worker startup timeout;
    • claim failure;
    • stale lease;
    • process crash;
    • late worker;
    • duplicate admission;
    • response persistence followed by worker death;
    • incomplete closeout;
    • source conflict;
    • recovery redrive;
    • active-turn fencing;
    • readiness transitions.

  Exit condition: a real-data and clean-room test proves the combined lifecycle.


  ──────────────────────────────────────────────────────────────────────────────

  Priority 5 — Consolidate State and Relationship

  This is the first feature area that should proceed after the operational boundary is reliable.

  State work

    • choose one authoritative state store;
    • unify StateManager and MemoryGovernance behavior;
    • prove post-update state is returned as current;
    • preserve prior snapshots as historical;
    • make every state change source-backed;
    • expose state through recall and continuity projections.

  Relationship work

    • choose one relationship schema;
    • unify RelationshipManagerV1 and MemoryGovernance;
    • update all readers to canonical endpoint fields;
    • ensure evidence-backed progression;
    • create typed relation records from canonical recording;
    • verify active/dormant relevance behavior.

  These workstreams can be developed in parallel with Dispatcher design, but their final MCP integration should wait until
  the Dispatcher and recording contract are stable.


  ──────────────────────────────────────────

  Priority 6 — Repair and consolidate Recall

  Recall should not be rebuilt from scratch.

  Instead:

    • select one normative recall path;
    • preserve compatibility adapters;
    • fix archived Experience indexing;
    • reject nonsense and zero-overlap results;
    • repair historical/superseded filtering;
    • fix malformed relationship/graph evidence handling;
    • verify entity reverse indexes;
    • repair conversation-chain reconstruction;
    • reconcile stale encoded units;
    • ensure State and Relationship projections affect recall correctly.

  The current remediation plan identifies this as a real-data validation problem, not merely a design problem.


  ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

  Priority 7 — ContextBootstrap Phase 1B

  Phase 1A has a tested snapshot builder. Phase 1B should wait until the canonical operational projections are stable.

  Otherwise, ContextBootstrap will repeatedly change because it depends on:

    • active-turn state;
    • recovery state;
    • readiness;
    • recall;
    • state;
    • relationships;
    • lineage.

  Implement it after those contracts stop moving.


  ──────────────────────────────────────────────────────────────────────────────────────────────────────────────

  Priority 8 — Attention integration

  Attention mechanics exist, but they are not currently the controlling gate for recall or capability discovery.

  After recall is consolidated, integrate Attention with:

    • current State;
    • active Relationships;
    • Direction;
    • risk;
    • unresolved decisions;
    • bounded recall;
    • focus budgets;
    • release/reactivation.


  ────────────────────────────────────────────

  Priority 9 — Skills and capability evolution

  This should be last among the listed architecture areas because it depends on nearly everything else:

  Experience → State/Relationship/History → Recall → Pattern → Attention → Candidate Capability → Skill → Outcome Evidence

  Implement in this order:

    1. Candidate Capability record;
    2. bounded capability discovery;
    3. Skill record and lifecycle;
    4. Skill invocation evidence;
    5. Candidate Experience from Skill use;
    6. Outcome attribution;
    7. Skill lineage;
    8. Skill coordination;
    9. Skill evolution.


  ─────────────────────────────────────────────────

  Revised Priority List

  Your original list can be reorganized as follows:

  | Priority | Workstream | Recommendation |
  |---:|---|---|
  | P0 | Fix validation environment | Do first; all completion claims depend on it. |
  | P0 | Resolve current Step2D/MCP operational backlog | Do before new feature expansion. |
  | P1 | Design Dispatcher execution model | Include RecoveryResponse and future concurrency contract. |
  | P1 | Implement Dispatcher | Start serialized with one worker. |
  | P1 | Verify Step2D + Dispatcher | Treat as the Dispatcher acceptance gate. |
  | P1 | Implement RecoveryResponse | Define during Dispatcher design; implement before Dispatcher signoff. |
  | P2 | Consolidate State and Relationship authority | Can begin contract/schema work in parallel, but integrate through MCP
   after recording stabilizes. |
  | P2 | Repair and consolidate Recall | Use real-data validation; do not rebuild from zero. |
  | P3 | ContextBootstrap Phase 1B | Defer until readiness, recovery, recall, state, and relationship contracts stabilize. |
  | P3 | Improve hook diagnostics | Add minimum diagnostics early if needed for validation, but defer broad redesign. |
  | P4 | Attention integration | Standalone mechanics exist; connect them to recall and discovery later. |
  | P5 | Skill evolution and closed learning loop | Design exists; runtime implementation is still pending. |


  ────────────────────────────────────────────────────────────────

  Final Recommendation

  Do not treat the project as seven independent urgent priorities.

  Use this smaller strategic sequence:

    1. Make validation trustworthy.
    2. Stabilize MCP recording and Step2D operations.
    3. Design and implement a serialized Dispatcher.
    4. Prove Dispatcher + Step2D together.
    5. Consolidate State, Relationship, and Event Relation schemas.
    6. Attach those domain updates to the canonical recording path.
    7. Validate and consolidate Recall.
    8. Make ContextBootstrap and Attention operationally integrated.
    9. Build Skills and the closed learning loop.

  The most important correction to your current mental model is:

    │ State and Relationship are not design-only and do not need to wait for MCP to exist. They are partially implemented
    │ foundation components. What is waiting is their reliable integration into the canonical MCP recording lifecycle.

  Also, the v5 diagram's largest unfinished area is not State or Relationship. It is the upper-level orchestration and
  lower-level learning loop:

    • complete Memory Control routing;
    • canonical recording-to-domain integration;
    • reliable historical graph traversal;
    • attention-governed activation;
    • capability emergence;
    • Skill creation and evolution;
    • outcome attribution;
    • closed learning loop.
