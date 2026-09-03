# Living Systems Architecture v0.6

## Operational and Canonical Integration Architecture

**Proposed implementation-guidance successor:** this document, replacing the
implementation mapping in `Living memory system doc v5.txt` and `Living memory
architecture system v5.txt` only after ratification

**Conceptual status:** v0.5 philosophy retained
**Operational status:** v0.6 proposed normative contract; not yet ratified
**Implementation status:** Transitional; not all contracts are implemented
**Production status:** Not authorized by this document

## 1. Purpose and version decision

Version 0.5 established the philosophy of Living Memory:

> Living memory is continuity through change.

That philosophy remains correct and is not replaced by v0.6.

Version 0.6 is necessary because the implementation now has operational
boundaries that v0.5 describes only conceptually:

- asynchronous MCP admission;
- durable mutation jobs;
- Dispatcher and worker ownership;
- source-persisted handoff;
- Memory Intelligence classification child jobs;
- `UNKNOWN` and `CLOSEOUT_PENDING` recovery states;
- explicit finalization;
- Audit Outbox delivery;
- canonical versus compatibility stores;
- State, Relationship, and Event Relation divergence;
- bounded Recall and Context Expansion;
- evidence-based anti-circularity.

The v0.6 change is therefore a **proposed operational and canonical integration
architecture**, not another broad conceptual rewrite or an operational release.

V0.6 establishes:

1. what each architectural concept means;
2. which component owns each decision;
3. which records are authoritative;
4. which records are derived or proposals;
5. how asynchronous execution preserves continuity;
6. how legacy implementations migrate without rewriting history;
7. what evidence is required before a capability is called complete.

### 1.1 Evidence basis

This revision was compared against:

- `Living memory system doc v5.txt`;
- `Living memory architecture system v5.txt`;
- the repository `references/01` through `references/18` corpus;
- architecture and protocol audits under `.living-memory/docs/`;
- current `.living-memory/core/`, `.living-memory/scripts/`, schemas, tests,
  and protocol records;
- `memory_intelligence_design.md`;
- the current `system_minestone.md` roadmap.

The reference corpus remains the source for the conceptual philosophy and
governance intent. Current code and runtime evidence determine implementation
status; neither is allowed to silently rewrite the conceptual model.

Until v0.6 is explicitly ratified, the v5 documents remain the active
conceptual authority and v0.6 is a proposed implementation-guidance successor.
Ratification requires Architecture, Builder, Validator, Project Manager, and
Owner review of the open decisions and milestone evidence.

During the transition:

- v5 governs the inherited conceptual philosophy;
- approved existing ADRs and operational contracts govern current runtime
  behavior;
- v0.6 is non-binding design guidance and cannot authorize implementation or
  rollout;
- after ratification, v0.6 governs operational mapping and canonical migration,
  while v5 remains historical conceptual provenance;
- an operational conflict must be recorded and resolved explicitly, never by
  silently choosing whichever document is newer.

## 2. Inherited philosophy

The following v0.5 principles remain authoritative.

### 2.1 Continuity through change

The system preserves origin, sequence, identity, current State, Relationships,
History, relevant context, and Direction while allowing the present to become
different from the past.

### 2.2 Experience is not the whole of memory

An Experience is what arrived, occurred, or was asserted. It is source evidence.
It may later contribute to State, Relationship, Event Relation, Direction,
History, Recall, or Skill-related evidence, but those are separate meanings.

### 2.3 History is not current State

An earlier State may remain historically meaningful while no longer being the
current State. A correction adds new evidence and a new transition; it does not
rewrite the earlier occurrence.

### 2.4 Possibility is not commitment

A generated or observed possibility can be retained as a candidate without
becoming an established Relationship, effective State, active context, or
explicit Skill.

### 2.5 Remembered is not active

A record may remain retained and retrievable while being dormant or outside the
current context. Relevance, activation, and Attention are separate decisions.

### 2.6 Skills operate on memory

Skills may retrieve, compare, organize, relate, reconstruct, or update memory
through governed interfaces. A Skill is not itself the memory, and Skill
execution does not automatically create a new retained Experience.

## 3. Canonical questions

The architecture is organized around distinct questions:

| Question | Authority/concept |
|---|---|
| What arrived or happened? | Experience |
| What is this continuing thing? | Entity |
| What is true about it at a time and scope? | State |
| What is connected to what? | Relationship |
| How are occurrences, claims, and transitions connected? | Event Relation |
| What happened along the way? | History |
| What should be available in the present? | Relevance / Context Membership |
| What should receive focus now? | Attention |
| Where is the continuing system going? | Direction / Intent |
| What can be reused as a capability? | Skill, only after governed emergence |
| What is currently safe and complete operationally? | Readiness projection |

These questions must not collapse into a single personalized-memory object,
LLM output, projection, or status field.

## 4. Canonical architecture

```text
CURRENT EXPERIENCE / CONVERSATION
              |
              v
HOST CAPTURE AND MCP ADMISSION
              |
              v
DURABLE PARENT MUTATION JOB
              |
              v
MUTATION DISPATCHER
              |
              v
MUTATION WORKER
              |
              v
IMMUTABLE SOURCE EXPERIENCE
              |
              +--------------------------+
              |                          |
              v                          v
SOURCE-PERSISTED HANDOFF       CLOSEOUT / ACTIVE TURN
              |                          |
              v                          v
BOUNDED CONTEXT RETRIEVAL      STEP2D RECOVERY
              |                          |
              v                          v
MEMORY INTELLIGENCE CHILD JOB  FINALIZATION AUTHORITY
              |
              v
CLASSIFICATION ENVELOPE
              |
              v
PROPOSALS AND ROUTE COMMANDS
       +------+-------+--------+
       |              |        |
       v              v        v
ENTITY AUTHORITY  STATE   RELATIONSHIP / EVENT RELATION AUTHORITY
       |              |        |
       +--------------+--------+
                      v
                   HISTORY
                      |
       +--------------+----------------+
                      |
                      v
             HISTORICAL INTEGRATION
                 |            |
                 v            v
       RELEVANCE / ATTENTION  AUDIT OUTBOX
                 |            |
                 v            v
       LIVING MEMORY PROJECTION  PUBLISHED AUDIT EVIDENCE
       |
       v
CURRENT CONTEXT / DIRECTION / NEXT STATE
       |
       v
NEW EXPERIENCE
```

The flow is asynchronous after admission. A host may receive `ACCEPTED` before
source persistence, classification, Governance, finalization, or audit delivery
has completed.

## 5. Current implementation versus v0.6 target

### 5.1 Transitional implementation

The current implementation contains strong but distributed primitives:

- MCP admission and durable mutation jobs;
- worker-start reservations;
- FIFO and lease/version fencing;
- source recording and response closeout;
- Step2D recovery history and finalization;
- typed Event Relation creation in some paths;
- State snapshots and supersession;
- Relationship lifecycle implementations;
- bounded Recall and continuity projections;
- standalone Attention allocation;
- heuristic pattern detection.

These components are not yet one fully aligned runtime. Current evidence shows:

- Dispatcher scheduling is distributed across `mcp_server.py` and mutation-job
  helpers rather than encapsulated in one Dispatcher;
- source persistence and classification do not yet have a durable child-job
  handoff;
- `MemoryControl` still contains heuristic semantic writes and source-field
  mutations;
- `data/states` and `data/state_snapshots` coexist;
- legacy and V1 Relationship implementations coexist;
- Event Relation writer and reader endpoint names diverge;
- Recall and historical traversal remain partially validated;
- Capability and Skill lifecycle features remain primarily specified;
- some tests and completion reports overstate behavioral evidence.

### 5.2 Evidence vocabulary

Every architecture report and milestone gate must distinguish:

- **Specified** — described by an approved contract or architecture document.
- **Implemented** — runtime code exists.
- **Tested** — reproducible automated tests pass.
- **Observed** — behavior works against current repository data or a real MCP
  lifecycle.
- **Reviewed** — an independent validator checked the evidence.
- **Authorized** — an explicit owner approved rollout.

No specification, schema, fixture, projection, or positive-only test is enough
to claim operational completion.

## 6. Authority and record taxonomy

### 6.1 Source records

| Record | Authority | Mutability |
|---|---|---|
| Experience | Source recording service | Source content immutable; later interpretation is separate |
| Entity | Entity Authority | Append-only identity assertions and governed lifecycle |
| State Snapshot | State Authority | Immutable snapshot; successor supersedes predecessor |
| Entity Relationship | Relationship Authority | Governed lifecycle and append-only changes |
| Event Relation | Event Relation Authority | Typed, evidence-backed, append-only/superseded |
| History Event | History authority | Append-only |
| Direction | Direction authority | Versioned and evidence-backed |
| Attention Allocation | Attention authority | Versioned attention decision |

### 6.2 Operational records

| Record | Authority | Meaning |
|---|---|---|
| Mutation Job | Durable mutation job store | Parent or derived execution lifecycle |
| Receipt | Admission/closeout protocol | Host-visible operation evidence |
| Recovery History | Step2D | Classification and reconciliation evidence |
| Finalization Record | Step2D finalization | Authorization and version-fenced commit |
| Source-Persisted Handoff | Protocol/handoff authority | Derived work eligibility after source persistence |
| Classification Child Job | Derived job store | Memory Intelligence execution |
| Classification Envelope | Memory Intelligence | Versioned interpretation artifact |
| Proposal | Memory Intelligence/route authority | Non-authoritative candidate change |
| RouteCommand | Routing authority | Durable delivery to a domain authority |
| Outbox Event | Audit Outbox | Delivery of audit facts |
| Readiness Projection | Derived operational projection | Current safety/readiness observation |

### 6.3 Derived records

Historical Integration assertions, Context Membership, representations, encoded
units, Recall results, Continuity projections, ranking scores, and Attention
recommendations are derived. They may be regenerated and must not replace source
or authority records.

## 7. Operational lifecycle

### 7.1 Admission

The MCP boundary validates:

- operation type;
- request schema;
- source idempotency key;
- source fingerprint;
- interaction and host lineage;
- parent operation identity;
- required response target;
- readiness restrictions applicable to admission.

The mutation store creates or recovers one parent job. The admission result
returns:

- `ACCEPTED` or `NOT_ADMITTED`;
- `operation_id`;
- `receipt_id`;
- `interaction_id`;
- job identity;
- exact lineage;
- replay indicator;
- bounded diagnostics.

`ACCEPTED` means durable admission only. It does not mean source persistence,
classification, closeout, Governance acceptance, finalization, or full audit.

### 7.2 Dispatcher scheduling

The Mutation Dispatcher:

- acquires a bounded leadership lease;
- scans durable parent and derived jobs;
- selects the oldest eligible FIFO job;
- reserves worker startup through the job store;
- launches the worker with a start token and expected version;
- observes process handles only as advisory evidence;
- invokes startup reconciliation and watchdogs;
- releases in-memory capacity after durable state is re-read.

The Dispatcher does not:

- own source truth;
- mark success from process exit code;
- finalize a response;
- classify an `UNKNOWN` operation without Step2D evidence;
- commit State, Relationship, or Event Relation changes;
- use an in-memory queue as authority.

### 7.3 Worker startup and claim

The Dispatcher reserves a `STARTED` job using:

- worker-start token;
- start expiry;
- expected version;
- queue sequence;
- reservation timestamp.

The worker claims only if:

- status is still `STARTED`;
- token matches;
- expected version matches;
- reservation is not expired;
- FIFO predecessor conditions permit claim.

Successful claim transitions the job to `PROCESSING` and records:

- worker PID;
- lease ID;
- lease generation;
- lease expiry;
- claim time;
- incremented version.

PID liveness and process exit are advisory. Version, lease generation, and
durable CAS are authoritative.

### 7.4 Source persistence

The worker invokes the recording service to persist the raw Experience.

Source persistence must preserve:

- raw content;
- source role;
- immutable source hash;
- received and occurred times;
- operation identity;
- interaction identity;
- exact host lineage;
- `in_response_to`, when applicable;
- source provenance;
- authoritative Entity ID only when already resolved.

Source recording must not invoke an external LLM or wait for semantic routing.

### 7.5 Source-persisted handoff

After authoritative source persistence, the system records a
`SOURCE_PERSISTED` handoff. The handoff joins:

- source Experience ID and hash;
- parent operation and receipt;
- parent job version;
- interaction and lineage;
- source role and response target;
- classification policy version;
- deterministic handoff identity;
- derived-job eligibility.

The handoff lifecycle is:

`RECORDED → CLASSIFICATION_SCHEDULED → CLASSIFICATION_ATTACHED`

or:

`RECORDED → SCHEDULING_UNKNOWN → RECONCILIATION_REQUIRED`

Source and handoff should be committed together through the transaction journal
when the storage contract permits. If not, a prepared transaction manifest and
recovery procedure must make the outcome explicit.

The handoff is not a source Experience and does not authorize semantic commits.

### 7.6 Memory Intelligence child job

The child job runs after source persistence and handoff discovery:

`CREATED → STARTED → PROCESSING → SUCCEEDED`

Alternative outcomes:

`STARTED/PROCESSING → UNKNOWN → RECOVERY_PENDING → SUCCEEDED | FAILED |
DEFERRED | DEAD_LETTERED`

Child `SUCCEEDED` means the Classification Envelope is durable. It does not mean
that a proposal was accepted, applied, or fully audited.

### 7.7 Governance and authority commit

The child job emits proposals and RouteCommands. Governance chooses:

- `ACCEPT`;
- `REJECT`;
- `DEFER`;
- `REQUEST_RESOLUTION`.

The domain authority then produces a separate commit outcome:

- `COMMITTED`;
- `CONFLICT`;
- `UNKNOWN`;
- `FAILED`;
- `REJECTED`;
- `DEFERRED`.

`ACCEPT` is not `COMMITTED`.

### 7.8 Closeout and finalization

Response closeout remains owned by the recording/closeout protocol and Step2D
finalization. Classification must not:

- complete the active turn;
- invalidate the current-turn marker;
- create a response when one already exists;
- convert `CLOSEOUT_PENDING` to success;
- finalize a source-unknown or source-conflict operation.

### 7.9 Outbox publication

Audit Outbox delivery is independent of source and authority commitment.

Outbox lifecycle:

`PENDING → PUBLISHING → PUBLISHED → ACKNOWLEDGED → COMPLETE`

Failure lifecycle:

`PUBLISHING → RETRY_PENDING → PUBLISHING`

or:

`PUBLISHING → DEAD_LETTERED`

Outbox retry uses the original event identity and idempotency key. Publication
does not change source truth, job result, Step2D status, or authority status.

## 8. Memory Control and Governance

### 8.1 Memory Control

Memory Control is the first semantic boundary after source availability. It may:

- perceive source content;
- extract deterministic hints;
- identify candidate Entities;
- request bounded context;
- invoke Memory Intelligence;
- classify possible operations;
- create proposal-only route commands;
- report uncertainty and conflicts.

Memory Control must not:

- make final truth decisions;
- mark a Relationship established or active;
- mark State current;
- commit identity merges;
- authorize recovery or finalization;
- treat LLM confidence as Governance;
- rewrite raw source content.

### 8.2 Memory Intelligence

Memory Intelligence is the interpretation implementation behind Memory Control.
It combines:

- deterministic source hints;
- bounded historical/context retrieval;
- LLM interpretation;
- deterministic output validation;
- proposal construction;
- provenance and anti-circularity checks.

It is not a new source authority and not a replacement for Governance.

### 8.3 Memory Governance

Governance decides whether a proposal can be accepted, rejected, deferred, or
sent for resolution. It records:

- proposal identity;
- evidence references;
- policy version;
- decision reason;
- reviewer/authority;
- expected authority version;
- decision timestamp;
- required next action.

Governance must not rely on:

- LLM confidence alone;
- projections;
- process exit codes;
- missing evidence interpreted as negative evidence;
- a Relationship's existence as proof of activation;
- a State's persistence as proof of current truth.

### 8.4 Source-first rule

The normative sequence is:

`source admission → source persistence → source-persisted handoff → bounded
context → classification → proposal → Governance → authority commit → History
→ projection`

No downstream interpretation may be treated as durable authority before the
source Experience exists.

## 9. Domain contracts

### 9.1 Entity: identity anchor

An Entity is a persistent subject that can continue through State change.

Entity types may include:

- person;
- agent;
- project;
- concept;
- object;
- place;
- document;
- conversation;
- process;
- event;
- goal;
- file;
- code element;
- Skill.

An Entity is not a complete personalized profile. A person profile is assembled
from Entity identity, identity assertions, State, Relationships, Experiences,
History, Direction, and relevant context.

#### Entity identity assertions

Identity assertions answer:

> Why should this mention or record be treated as the same continuing Entity?

They should contain:

- assertion ID;
- assertion text or normalized identity fact;
- source Experience IDs;
- evidence class;
- provenance;
- scope;
- asserted time;
- confidence/uncertainty;
- Governance decision when the assertion changes identity authority.

Identity assertions must not be used for mutable current properties.

#### Entity authority rules

- LLM output may propose Entity matches or creation.
- A single unambiguous match may be resolved.
- Multiple plausible matches remain `AMBIGUOUS`.
- No-match may produce an Entity creation proposal.
- Merge and split require explicit Governance and lineage.
- Null Entity IDs never match one another.
- Entity resolution is required before canonical State or Relationship commit.
- Entity identity is not inferred from a projection alone.

### 9.2 State: time-scoped properties

State answers:

> What is true about this Entity in this scope and time?

State is not identity and not full History.

#### Canonical State target

V0.6 selects `data/states/` as the target canonical new-write store because it is
the store currently used by Memory Governance and retrieval. `data/state_snapshots/`
and the alternative `StateManager` field shape remain compatibility inputs during
migration.

This is a target authority decision, not a claim that migration is complete.

#### State snapshot contract

Required concepts:

- immutable snapshot ID;
- resolved Entity ID;
- typed fields;
- scope;
- asserted time;
- effective time, when supported;
- snapshot/persistence time;
- source Experience IDs;
- predecessor/supersession link;
- status;
- Governance decision ID;
- authority version.

Target statuses:

- `PROPOSED`;
- `CURRENT`;
- `HISTORICAL`;
- `CONTESTED`;
- `SUPERSEDED`.

State transitions create new snapshots. They do not mutate old snapshots into a
different historical meaning.

#### State authority rules

- A classifier may propose fields and change kind.
- A resolved Entity is required for commitment.
- Ambiguous temporal claims remain proposed or contested.
- The classifier cannot set `CURRENT`.
- Governance acceptance is required before authority commit.
- Previous current State becomes historical or superseded.
- Every committed change creates a History event.
- Current-state selection is deterministic for Entity and scope.
- State fields must not be stored as a second mutable authority on Entity.

### 9.3 Entity Relationship: governed semantic edge

An Entity Relationship answers:

> What persistent subject is connected to what other persistent subject, and with
> what governed meaning?

It is a first-class edge with:

- subject Entity;
- predicate/relation type;
- object Entity or explicitly typed value endpoint;
- category;
- evidence;
- provenance;
- scope;
- temporal validity;
- lifecycle status;
- contradiction and supersession links;
- Governance decision;
- authority version.

#### Relationship authority target

V0.6 selects the V1 semantic record shape over `data/relationships/` as the
target contract, but V1 status transitions must not bypass Governance.

Legacy `MemoryGovernance` relationship methods become compatibility adapters or
must delegate to the selected Relationship Authority. There must not be two
independent new-write authorities.

#### Relationship lifecycle

The canonical progression is:

`POSSIBLE → CANDIDATE → SUPPORTED → ESTABLISHED → ACTIVE`

Other governed outcomes include:

- `DORMANT`;
- `HISTORICAL`;
- `INVALIDATED`;
- `SUPERSEDED`;
- `ARCHIVED`.

These dimensions remain distinct:

- lifecycle/truth status;
- relevance;
- activation;
- Attention.

`ACTIVE` does not mean true, and `ESTABLISHED` does not mean present context.

#### Relationship authority rules

- LLM output creates only a proposal.
- Null or unresolved endpoints cannot be committed.
- Confidence does not create `ESTABLISHED` or `ACTIVE`.
- Evidence must be loaded from authoritative records.
- An LLM hypothesis cannot independently support another LLM hypothesis.
- Contradictions preserve both evidence paths.
- Relationship activation is a separate relevance/attention decision.
- Every accepted lifecycle change creates History evidence.
- Existing V1 records must be read without silently upgrading their status.

### 9.4 Event Relation: historical edge

An Event Relation answers:

> How are occurrences, claims, State transitions, and interpretations connected?

Event Relation is not Entity identity and is not automatically an Entity
Relationship.

#### Identity versus edge

| Concept | Role |
|---|---|
| Entity | Persistent identity anchor/node |
| Identity assertion | Evidence that a mention refers to an Entity |
| Experience | Source occurrence/assertion |
| State | Time-scoped properties of an Entity |
| Entity Relationship | Governed semantic edge between persistent subjects |
| Event Relation | Typed historical edge between occurrences, claims, States, or interpretations |

The fact that two Events mention the same Entity does not itself establish a
persistent Entity Relationship.

#### Canonical Event Relation endpoint contract

New records must use:

- `source_endpoint`;
- `source_endpoint_type`;
- `target_endpoint`;
- `target_endpoint_type`;
- `predicate`;
- `evidence_refs`;
- `origin_experience_id`;
- `scope`;
- `asserted_at`;
- `effective_at`, when applicable;
- `assertion_status`;
- `provenance`;
- `governance_decision_id`, when governed;
- `idempotency_key`;
- `version`.

Allowed endpoint types must be explicit, including as applicable:

- `EXPERIENCE`;
- `HISTORY_EVENT`;
- `ENTITY`;
- `STATE`;
- `RELATIONSHIP`;
- `CLAIM`;
- `INTERPRETATION`;
- `PATTERN`.

Legacy `source_event_id`/`target_event_id` and `source_event`/`target` forms may
be read through compatibility adapters only. New writes must use one canonical
form.

#### Deterministic and semantic Event Relations

Deterministic protocol relations include:

- response `CONTINUES` exact User Experience;
- State snapshot `SUPERSEDES_STATE_FROM` predecessor;
- Governance decision `APPLIES_TO` proposal;
- closeout `COMPLETES` active turn.

LLM-proposed semantic relations include:

- `CAUSES`;
- `RESULTS_FROM`;
- `CONTRADICTS`;
- `PROVIDES_EVIDENCE_FOR`;
- `DEPENDS_ON`;
- `REACTIVATES_RELEVANCE_OF`;
- `RECURS_FROM`;
- `INITIATES_TRANSITION`;
- `CONFIRMS_STATE`.

Semantic relations remain proposals until Governance and Event Relation
Authority commit them.

Rules:

- temporal order does not prove causality;
- evidence does not automatically prove truth;
- Event Relation status is independent of endpoint status;
- multiple predicates may connect the same endpoints;
- a generic `related_to` edge must not replace meaningful typed predicates;
- cycles may be preserved and flagged, but cannot be used as independent proof;
- an integration assertion is not automatically a committed Event Relation.

### 9.5 History

History retains:

- origin;
- sequence;
- source events;
- State transitions;
- Relationship changes;
- Governance decisions;
- corrections;
- contradictions;
- supersession;
- recovery and finalization outcomes.

History is append-only. A later interpretation adds a record or relation; it
does not rewrite the source occurrence.

### 9.6 Living Memory projection

#### Context Membership contract

Context Membership is a governed/derived assertion that a record should be
available to a named context within a bounded scope and time. It is not source
truth, Entity identity, State, Relationship truth, or Attention focus.

Each membership record contains:

- `membership_id`;
- `context_id` and `context_type`;
- `member_id` and `member_type`;
- `membership_kind`;
- `status`;
- `relevance_reason`;
- `evidence_refs`;
- `provenance`;
- `scope`;
- `evaluated_at`;
- `effective_from` and `effective_until`, when applicable;
- `attention_state`, if present, as an observation rather than authority;
- `policy_version`;
- `idempotency_key`;
- `supersedes`/`superseded_by`, when the membership decision changes.

Target statuses are:

`PROPOSED → ACTIVE → DORMANT → RELEASED | EXPIRED | SUPERSEDED`

`CONTESTED` and `REVIEW_REQUIRED` are non-active waiting states. A membership
may return from `DORMANT` to `ACTIVE` only after a new bounded evaluation; it
does not become active merely because it was historically active.

The Relevance/Context Membership authority owns:

- scope and policy evaluation;
- evidence loading;
- expiry and release;
- reactivation;
- membership idempotency;
- projection inputs.

Memory Intelligence may emit an `attention_review_signal` or membership
proposal, but it cannot set `ACTIVE` or `FOCUSED`. Attention owns focus, while
Context Membership owns availability. A projection may display either decision
but cannot create it.

Living Memory is the present-facing derived coherence of:

`identity + current State + relevant Relationships + relevant History +
Direction + validated recovery/readiness evidence`

The projection must retain distinctions among:

- retained/available;
- relevant;
- activated;
- attended/focused;
- current;
- historical;
- dormant;
- possible/candidate;
- contested;
- recovery-required.

Projection data is regenerable and non-authoritative.

### 9.7 Historical Integration

Historical Integration is the distinct layer between History and present-facing
Living Memory. It connects and interprets durable records without becoming a
second authority.

It may:

- traverse bounded Event Relation chains;
- connect Experiences through shared Entity and lineage evidence;
- compare current and historical State;
- identify recurring transitions and patterns;
- expose indirect Relationship paths;
- produce integrated-context assertions and bounded explanations;
- provide reconciliation evidence to Memory Intelligence and Recall.

It must not:

- rewrite source Experiences or History;
- turn an inferred path into a direct Relationship;
- turn a temporal sequence into causality;
- promote an Event Relation proposal;
- treat a projection or integration assertion as independent source evidence.

The existing `HistoricalIntegration` implementation and
`integration_assertions` are transitional derived components. Their outputs are
inputs to Recall, ContextBootstrap, and reconciliation, not replacements for
canonical Entity, State, Relationship, or Event Relation records.

#### Integration Assertion contract

An Integration Assertion is a bounded derived explanation or path result. It is
not a canonical Event Relation and cannot be used as an independent evidence
root.

Each assertion contains:

- `integration_assertion_id`;
- `subject_id` and `subject_type`;
- `object_id` and `object_type`, when applicable;
- `path_nodes` and `path_edges`;
- `path_fingerprint`;
- `assertion_kind`;
- `assertion_text` or structured result;
- `source_refs`;
- `evidence_classes`;
- `provenance`;
- `scope`;
- `computed_at`;
- `algorithm_version`;
- `status`;
- `uncertainties`;
- `supersedes`, when a later bounded computation replaces the result.

Target statuses are:

`COMPUTED → VALIDATED | REVIEW_REQUIRED | STALE | REJECTED | SUPERSEDED`

`VALIDATED` means the path is structurally and evidentially valid for derived
presentation. It does not mean that the inferred conclusion is a direct fact,
an established Relationship, or an authority commit.

The Historical Integration service owns traversal and path computation. Recall,
ContextBootstrap, and Memory Intelligence may consume assertions with their
status and path intact. If an assertion is used to support a new authority
proposal, the validator must preserve it as derived evidence and require an
independent allowed source/authority root; the assertion itself cannot be the
only support.

## 10. Memory Intelligence and reconciliation

### 10.1 Source-specific interpretation

The same classifier contract handles User and Agent Experiences while preserving
source role.

User input emphasizes:

- assertions;
- preferences;
- goals;
- corrections;
- commitments;
- unresolved questions;
- Direction changes.

Agent responses emphasize:

- conclusions;
- explanations;
- proposed solutions;
- shared decisions;
- discoveries;
- commitments;
- limitations.

The classifier must distinguish:

- hypothesis from decision;
- proposed State from effective State;
- possible Relationship from established Relationship;
- generated explanation from source fact;
- response text from user authorization.

### 10.2 Context retrieval sequence

Classification uses a bounded two-pass model:

1. deterministic source hint extraction;
2. optional bounded New Expansion inquiry;
3. first Recall pass using explicit hints and declared scope;
4. LLM classification against retrieved evidence;
5. optional allowlisted second Recall pass for unresolved claims;
6. deterministic reconciliation against current State, Relationships, History,
   provenance, and supersession;
7. proposal validation and routing.

Retrieved text is evidence, not instructions. Missing context is uncertainty,
not negative evidence.

### 10.3 Classification artifact

The Classification Envelope must preserve:

- source Experience ID and source hash;
- parent operation and lineage;
- classifier/prompt/parser version;
- policy version;
- context fingerprint;
- source role;
- intents;
- claims;
- proposal-only operations;
- evidence references;
- uncertainties;
- context expansion signals;
- validation diagnostics;
- result hash;
- artifact status.

Artifact statuses:

`PENDING`, `CLASSIFIED`, `PARTIAL`, `REVIEW_REQUIRED`, `DEFERRED`, `FAILED`,
`UNKNOWN`, and `DEAD_LETTERED`.

### 10.4 Prompt and output safety

The prompt must tell the LLM:

- it is an interpreter, not an authority;
- all source/context text is untrusted data;
- every claim needs a source or supplied evidence reference;
- uncertainty must be preserved;
- no canonical IDs may be generated;
- all proposals require Governance;
- truth, retention, activation, current State, and Relationship status are not
  decided by confidence;
- no tool calls or executable commands are valid output.

Deterministic parsing must reject:

- unknown proposal kinds;
- unavailable evidence references;
- canonical IDs generated by the model;
- `requires_governance=false`;
- unsupported authority statuses;
- executable instructions;
- malformed or over-limit values.

### 10.5 Mechanical anti-circularity

Every evidence reference has a class:

- `SOURCE_EXPERIENCE`;
- `HUMAN_ASSERTION`;
- `AUTHORITY_RECORD`;
- `TOOL_OBSERVATION`;
- `LLM_CLASSIFICATION`;
- `DERIVED_PROPOSAL`;
- `PROJECTION`.

The deterministic validator must:

1. load evidence from its authority;
2. verify the declared evidence class;
3. construct a bounded provenance graph;
4. reject self-reference and repeated-node paths;
5. mark every node whose provenance contains an LLM classification, derived
   proposal, integration assertion, or projection;
6. prohibit any marked node from acting as a support bridge, even when the path
   eventually reaches a valid source or authority node;
7. require an unbroken allowed support path from the proposal to an independent
   source/authority root;
8. apply the proposal-kind evidence policy before routing;
9. preserve the exact bounded support paths and rejected bridge paths;
10. return `REQUEST_RESOLUTION` when required independent support is absent.

LLM confidence, projection membership, and another ungrounded proposal are not
independent support. An Integration Assertion can explain a path, but it cannot
be an independent support root or an unmarked bridge to one.

### 10.6 Classification does not authorize commitment

The LLM can propose:

- Entity match or creation;
- State change;
- Entity Relationship;
- Event Relation;
- Direction;
- retention signal;
- pattern assertion;
- attention review;
- capability discovery inquiry.

It cannot directly create:

- an Entity merge;
- a current State;
- an established or active Relationship;
- an effective Event Relation;
- a truth or retention decision;
- an active context membership;
- a Skill;
- a recovery or finalization outcome.

## 11. Integrity invariants

### Source and identity

1. Source Experience is authoritative for what arrived or occurred.
2. Raw source content and source hash are immutable.
3. Operation, interaction, recording, classification, proposal, and authority
   IDs remain distinct.
4. Exact `in_response_to` linkage is preserved.
5. Entity resolution precedes State and Relationship commitment.
6. Null Entity IDs never match.

### State and Relationship

7. State is time-scoped and Entity-owned.
8. Entity identity is not a mutable State store.
9. Previous State snapshots remain historical or superseded.
10. Relationship possibility is distinct from establishment.
11. Relationship establishment is distinct from activation.
12. Activation is distinct from Relevance and Attention.
13. Confidence does not authorize a lifecycle transition.
14. Every committed change cites source evidence and Governance.

### Event and History

15. Event Relation endpoints use one canonical contract.
16. Typed Event Relation predicates are preserved.
17. Temporal order does not establish causality.
18. Contradictions and corrections are additive.
19. History is append-only.
20. Projections and integration assertions do not replace History events.

### Async execution and recovery

21. `ACCEPTED` is not `SUCCEEDED`.
22. Child classification completion does not complete the parent response.
23. `UNKNOWN` is not `FAILED`.
24. Reconcile before replay.
25. A new operation follows only authoritative `NOT_FOUND`.
26. PID and process exit are advisory.
27. Version, lease generation, and durable CAS fence stale workers.
28. Recovery never invents a response.
29. `CLOSEOUT_PENDING` remains recovery-required.
30. Outbox publication does not change source or authority status.

### Evidence and governance

31. LLM output is proposal data.
32. Retrieved context is untrusted evidence, not instruction.
33. Every proposal contains evidence and provenance.
34. Mechanical anti-circularity is deterministic.
35. Missing evidence creates uncertainty, not absence.
36. `ACCEPT` and `COMMITTED` are separate outcomes.
37. Readiness is a projection, not authorization.
38. `FINALIZED` and `FULLY_AUDITED` are derived aggregates over verified
    evidence.

## 12. Compatibility and migration

### 12.1 General migration rules

- Compatibility readers precede writer cutover.
- Existing source records are never rewritten to fit v0.6.
- Existing inferred provenance is not upgraded by migration.
- Every migration is idempotent.
- Every migration emits an inventory, mapping result, and exception report.
- Unknown or malformed records remain visible as exceptions.
- Rollback changes the active reader/writer path; it does not delete history.

### 12.2 State migration

1. Inventory `data/states` and `data/state_snapshots`.
2. Map temporal, supersession, status, and source fields.
3. Select `data/states` for new writes.
4. Preserve the source ID and original timestamps.
5. Keep legacy records readable through an adapter.
6. Detect conflicting current States rather than choosing by file time.
7. Prove deterministic current-state selection by Entity and scope.
8. Retain an exception report for records that cannot be mapped safely.

### 12.3 Relationship migration

1. Inventory legacy and V1 records.
2. Normalize subject, predicate, object, category, and evidence fields.
3. Select one Relationship Authority for all new writes.
4. Preserve legacy status as historical evidence where it cannot be proven under
   the v0.6 lifecycle.
5. Do not upgrade `candidate` or `active` automatically.
6. Reject or quarantine null/ambiguous endpoints.
7. Preserve contradiction and supersession paths.

### 12.4 Event Relation migration

1. Inventory all endpoint field variants.
2. Add compatibility readers for legacy forms.
3. Emit only the canonical endpoint form for new records.
4. Normalize endpoint types.
5. Preserve original relation IDs and source evidence.
6. Re-run public lookup and traversal, not only direct filesystem scans.
7. Verify deterministic response `CONTINUES` relations.
8. Verify causal and contradiction predicates separately.

### 12.5 Source interpretation migration

Legacy fields such as `interpretation_status`, `routed_to`, and
`retention_level` may be read as historical compatibility metadata. New derived
classification, retention, truth, and routing results belong in append-only
derived or authority records. The raw source content and source identity remain
unchanged.

## 13. Readiness and completion predicates

### 13.1 Readiness

| State | Meaning |
|---|---|
| `BLOCKED` | Safety violation, incomplete closeout, or invalid active ownership |
| `RECOVERY_REQUIRED` | Current uncertainty, recovery backlog, source conflict, or dead-letter work |
| `DEGRADED` | Work is active or audit is pending without a recovery blocker |
| `ONLINE` | Required active, recovery, closeout, and audit work is clear |

Readiness is observational. It does not authorize admission, classification,
finalization, or authority writes.

### 13.2 Finalization aggregates

`FINALIZED` requires:

- authoritative source persistence;
- recovery closed or explicitly not required;
- parent job succeeded under its closeout contract.

`FULLY_AUDITED` requires:

- `FINALIZED`;
- required audit Outbox completion.

Source persistence, child classification, Governance acceptance, or outbox
acknowledgement alone cannot establish either aggregate.

## 14. A–I implementation roadmap

### 14.1 Execution order

The labels are architectural workstreams, not permission to execute in numeric
order. The recommended implementation order is:

`A → B → C → D → F → G → E → H → I`

E may be designed during B, but controlled-concurrency implementation and
rollout remain gated by D, the required F authority contracts, and the relevant
G retrieval evidence. State, Relationship, and Event Relation contract work may
be prepared in parallel, but shared recording/MCP writer changes have one
integration owner.

### A — Trust the Evidence

Establish:

- reproducible tests;
- isolated roots;
- live-data census;
- current mutation/recovery/outbox status;
- evidence classification;
- active-turn and closeout status.

Exit evidence:

- clean test report;
- operation/recovery inventory;
- environment and command record;
- independent validation.

### B — Complete Operational Design

Freeze:

- Dispatcher ownership;
- source-persisted handoff;
- child-job lifecycle;
- RecoveryResponse;
- Step2D boundaries;
- finalization;
- Outbox lifecycle;
- parent/child status relationship;
- idempotency and fencing.

Exit evidence:

- approved state machines;
- authority matrix;
- failure matrix;
- durable contract artifacts;
- named approvers.

### C — Reliable Execution

Implement:

- serialized Dispatcher;
- `max_workers=1`;
- durable handoff;
- classification child-job scheduling;
- RecoveryResponse through governed closeout;
- existing lease/version/token fencing.

Do not enable broad concurrency or direct semantic authority writes.

### D — Recovery Correctness

Prove:

- worker crash;
- startup timeout;
- claim and lease expiry;
- parent `UNKNOWN` reconciliation;
- source persisted but closeout pending;
- classification child crash;
- duplicate artifact and proposal replay;
- late worker fencing;
- source conflict;
- outbox retry/dead letter.

### E — Controlled Concurrency (conditional performance gate)

Only after D, the required F authority contracts, and the relevant G retrieval
evidence:

- concurrent transport;
- bounded worker pools;
- conflict-aware scopes;
- parallel independent child jobs;
- serialized State/Relationship/closeout conflicts;
- feature flags and rollback.

Concurrency is a performance option, not a prerequisite for pure classification.

### F — Canonical State, Relationship, and History Integration

Complete:

- State authority selection and migration;
- Relationship authority selection and migration;
- Event Relation endpoint normalization;
- deterministic response and transition relations;
- source evidence requirements;
- History emission;
- Memory Intelligence proposal routing;
- mechanical anti-circularity;
- real-data authority validation.

F is the first milestone that enables production semantic proposal application.

### G — Retrieval and Recall Closure

Complete and observe:

- archived Experience indexing;
- historical/superseded filtering;
- malformed evidence handling;
- Entity reverse-index coverage;
- ranking and negative queries;
- conversation-chain reconstruction;
- stale encoded-unit reconciliation;
- bounded context fingerprints and provenance.

### H — ContextBootstrap and Attention Integration

Integrate:

- current State;
- Relationships;
- History;
- Direction;
- recovery/readiness;
- bounded Recall;
- Attention allocation;
- Context Membership.

Prove that projections and Attention do not authorize writes.

### I — Skills and Closed Learning Loop

Implement only after the preceding evidence supports it:

- Candidate Capability;
- attention-governed discovery inquiry;
- Skill record and lifecycle;
- invocation/outcome evidence;
- outcome attribution;
- Skill lineage and provenance;
- coordination;
- governed promotion, revision, retirement, and evolution.

Patterns and classifier output alone do not constitute Skills or a closed
learning loop.

## 15. Validation matrix

| Capability | Specified | Implemented | Tested | Observed | Reviewed | Authorized | Current v0.6 treatment |
|---|---:|---:|---:|---:|---:|---:|---|
| Source Experience persistence | Yes | Yes | Partial/strong | MCP evidence exists | No | No | Preserve as source authority |
| MCP admission | Yes | Yes | Yes | Yes in bounded flows | Partial | No | Preserve boundary |
| Mutation Dispatcher | Yes | Partial/distributed | Partial | Not complete | No | No | Implement in C |
| Source-persisted handoff | Yes in v0.6 design | No | No | No | No | No | Implement in C |
| Step2D recovery | Yes | Substantial | Focused | Bounded evidence | Partial | No | Complete D gate |
| Response closeout | Yes | Substantial | Focused | Bounded evidence | Partial | No | Preserve authority |
| Memory Intelligence child job | Yes in design | No | No | No | No | No | Implement after C |
| Entity resolution | Yes | Partial | Partial | Limited | No | No | Consolidate in F |
| State snapshots | Yes | Yes in competing paths | Focused | Partial | No | No | Select authority in F |
| Relationship lifecycle | Yes | Yes in competing paths | V1-focused | Partial | No | No | Consolidate in F |
| Event Relation creation | Yes | Yes in paths | Partial | Traversal mismatch risk | No | No | Normalize in F |
| Event Relation traversal | Yes | Partial/broken in paths | Partial | Not complete | No | No | Fix in F |
| History | Yes | Yes | Partial | Bounded | No | No | Preserve append-only |
| Historical Integration | Yes | Partial | Partial | Not complete | No | No | Restore as derived integration layer |
| Recall | Yes | Partial | Partial | Live gaps remain | No | No | Close in G |
| Context Membership | Yes | Partial | Partial | Limited | No | No | Keep derived and integrate in H |
| ContextBootstrap | Yes | Partial/Phase 1A | Focused | Not mandatory flow | Partial | No | Complete in H |
| Attention | Yes | Standalone | Direct tests | Normal-flow evidence absent | No | No | Integrate in H |
| Pattern recognition | Yes | Heuristic | Proxy tests | Limited | No | No | Keep as candidate evidence |
| Mechanical anti-circularity | Yes | Partial | Partial | Not complete | No | No | Complete in F |
| Capability discovery | Yes | No complete runtime | No | No | No | No | Implement in I |
| Skill lifecycle | Yes | No | No | No | No | No | Implement in I |
| Closed learning loop | Yes | No | No | No | No | No | Implement in I |

## 16. Open decisions requiring approval

The following must be approved before implementation claims are upgraded:

1. Final State authority and migration compatibility duration.
2. Final Relationship Authority and V1/legacy delegation strategy.
3. Final Event Relation schema and predicate registry.
4. Classification and handoff storage paths.
5. Child-job versus Outbox scheduling implementation details.
6. Exact provider, model, residency, retention, and protected-data policy.
7. Maximum source/context/evidence/proposal limits.
8. Proposal-kind policies may require stricter independent evidence than the
   v0.6 minimum; they may not weaken the mechanical anti-circularity rule.
9. Authority review requirements for Entity creation, merge, State contest, and
   causal Event Relations.
10. Readiness precedence for derived child-job uncertainty.
11. Migration treatment for records with null IDs, malformed endpoints, or
    unprovenanced evidence.
12. Rollback behavior for authority writer cutover.

Open decisions are not implementation permission. They are explicit blockers to
the affected rollout gate.

## 17. Non-goals

V0.6 does not:

- replace the v0.5 Living Memory philosophy;
- make LLM output authoritative;
- rebuild Recall from zero;
- introduce unrestricted concurrency;
- remove the global worker lock;
- create automatic Skills;
- treat patterns as capabilities;
- treat projections as source truth;
- rewrite historical source records;
- perform manual live-data repair;
- authorize deployment;
- declare the complete v5 architecture operational.

## 18. Final architectural principle

The v0.6 architecture is living because it preserves continuity through change
while keeping each kind of change accountable to its proper authority:

```text
Experience records what arrived.
Entity preserves identity.
State records what is true in a time and scope.
Relationship records governed semantic connection.
Event Relation records historical connection.
History preserves how meaning and condition changed.
Memory Intelligence proposes interpretations.
Governance decides what may become authoritative.
Authorities commit State, Relationship, and Event Relation.
Step2D reconciles uncertainty and protects finalization.
Outbox publishes audit evidence.
Projections present the current coherent view.
```

The canonical operational rule is:

> **Source first. Interpret second. Govern third. Commit fourth. Record History.
> Project last.**

That rule preserves the original v0.5 philosophy while making the next phase
implementable, recoverable, and auditable.
