I'll review the actual project implementation against the documented Living Memory cycle, enumerate every current component, and draw the architecture as it exists—not as an idealized future design. I'll keep the review focused on the meaning and memory-composition layers you named, while using the real Living Memory MCP for this turn and recording the completed review afterward.

The new MCP ingress was reached, but the persisted active execution still expects the previous host lineage, so it correctly rejected the newly generated lineage rather than silently creating a parallel turn. I'm going to retry against the already active governed lineage, which is the only non-bypass path available here; then I'll use the repository implementation and design documents to assess alignment.

The real MCP boundary is still rejecting this turn because the persisted active execution and the submitted host lineage do not agree. I will not edit persistence directly or bypass that decision. I'll first inspect the current MCP transition contract and the implementation/documents needed for the review, then retry the governed turn with the lineage the server currently requires.

The retry confirms the issue is not the user's old isolated identity: the server has a completed, invalidated turn whose `host_event_id` is already bound to one lineage, so reusing it is correctly rejected. That is a protocol concern outside the requested meaning/lifecycle review. I'll proceed with the architecture analysis using source and design documents, and report the MCP recording limitation separately rather than treating it as a memory-layer behavior.

I'll finish the review by tracing the actual semantic path—not the protocol or enforcement path:

1. **Input and experience handling**
2. **Perception/identification and routing**
3. **Recall candidate generation, scoring, and composition**
4. **Relationships, history, direction, integration, and attention**
5. **Lifecycle of memory records**
6. **Gaps between the documented life cycle and the implementation**

Then I'll provide a component inventory and a current architecture diagram, explicitly leaving out state, protocol, enforcement, and guard mechanisms.

The main path is now clear. I'm checking the remaining semantic pieces that determine whether the implementation actually completes the cycle: response-to-experience recording, historical integration, governance promotion, and the on-disk record types. I'll continue excluding protocol/enforcement/state details from the final architecture; "state" here means the lifecycle concept only where the design requires it, not the operational state machinery.

I found the decisive distinction:

- Your **design is a continuity-composition architecture**.
- Your **current runtime is primarily an experience-retrieval architecture with relationship/direction signals**.

Relationships and directions already influence relevance, but they are not consistently materialized into the context delivered to the LLM. Historical integration and several lifecycle operations exist as components, yet they are not all on the normal recall path. I'll make that explicit in the diagram and review rather than presenting intended behavior as implemented behavior.

## Scope

This review covers only the **semantic Living Memory system and lifecycle**.

I intentionally exclude:

- operational State machinery;
- protocol and host-lineage handling;
- enforcement layers;
- guards and recovery controls.

## Executive conclusion

Your system is **conceptually aligned** with the Living Memory lifecycle, but the implementation is currently **partially aligned**.

The design describes:

> New Experience → identify meaning and connections → selectively recall relevant memory → integrate it with the present → determine Direction → produce the next experience.

Your implementation currently does this well for:

- recording Experiences;
- semantic matching;
- relevance scoring;
- graduated recall depth;
- Direction-aware scoring;
- relationship graph discovery;
- retention and archival;
- recording agent responses;
- building a canonical continuity projection.

However, the runtime does **not yet fully compose**:

> past Experiences + Relationships + Direction + History + Integration + current prompt

into the context delivered to the LLM.

In practice, the LLM usually receives:

> current prompt + a list of relevant past Experiences

while Relationships, Directions, and other memory layers mostly influence **selection and scoring**, rather than appearing as structured activated context.

---

# 1. What really happens when a new prompt arrives

## Step 1: The prompt becomes an Experience

`AgentMemoryInterface.receive_user_message()` sends the message to:

- `MemoryControl.receive_experience()`

This creates an Experience record containing:

- source;
- source entity;
- occurrence time;
- received time;
- content;
- interpretation status;
- truth status;
- retention level.

The primary record is stored under:

- `.living-memory/data/experiences/`

This is the strongest part of the implementation. The incoming message is treated as an event in the continuing world rather than merely as an unrecorded prompt.

## Step 2: Memory Control interprets and routes it

`MemoryControl.route_experience()` uses simple textual heuristics to determine whether the Experience may involve:

- entity identification;
- Direction or intent;
- relationship creation;
- an update operation.

The result is a routing plan such as:

- `identify_source_entity`;
- `establish_intent`;
- `propose_relationship`;
- `state_update`.

This corresponds to the **Perception → Identification → Memory Operation** part of the documented lifecycle.

However, this layer is still mostly heuristic.

### Current limitations

- Entity identification is currently mostly a placeholder. `_handle_identify_entity()` logs the operation but does not perform full entity resolution.
- Direction detection identifies intent, but `_handle_establish_intent()` currently logs it rather than automatically creating or updating a Direction.
- Relationship extraction is simplistic. For example, a "working on" message may create a relationship from the source entity to the conversation entity rather than to a properly resolved project entity.

Therefore, the routing architecture exists, but semantic interpretation is not yet complete.

---

# 2. How recall actually works

The main recall path is:

> `ContinuousMemoryLayer` → `AutomaticRelevanceEvaluator` → `PatternRecognition` → `RecallDepthEvaluator` → `MemoryControl.recall_relevant_experiences()`

## 2.1 Candidate generation

`AutomaticRelevanceEvaluator.generate_candidates()` searches Experience records and creates candidate pools based mainly on:

- recency;
- retention level;
- semantic similarity;
- session scope.

It maintains two important groups:

1. filtered candidates;
2. recent Experiences used for semantic and continuity analysis.

It also selects bounded semantic candidates using `SemanticMatcher`.

The system does **not** simply retrieve every past Experience. It attempts to create a limited candidate set.

## 2.2 Pattern recognition

`PatternRecognition.evaluate_all_dimensions()` compares the current Experience against memory through multiple continuity dimensions:

- Entity continuity;
- Relationship continuity;
- Experience continuity;
- Semantic continuity;
- Cross-context continuity;
- Temporal continuity;
- Direction continuity;
- state-change continuity.

For this review, excluding the State dimension, the important active dimensions are:

- entity;
- relationship;
- experience;
- semantic;
- cross-context;
- temporal;
- direction.

The overall score is weighted. Direction receives the largest weight, followed by entity, relationship, and temporal continuity.

## 2.3 Semantic matching

`SemanticMatcher` provides:

- keyword extraction;
- normalization;
- synonym expansion;
- phrase matching;
- weighted semantic similarity.

This allows the system to recognize that different wording may refer to a similar concept.

For example, a new prompt about "continuing the memory work" may match an earlier prompt containing "session continuity implementation" even when the exact wording differs.

This is aligned with the design principle:

> meaningful continuity is more important than simple lexical equality.

## 2.4 Direction matching

`DirectionMatcher` loads active Directions from:

- `.living-memory/data/directions/`

It compares the current prompt and candidate Experiences with Direction intent using:

- keyword overlap;
- semantic similarity.

Direction therefore acts as a relevance signal:

> "Is this Experience connected to what the continuing system is trying to accomplish?"

This is aligned with the lifecycle.

But Direction is not consistently included in the final activated context. It primarily increases relevance scores.

## 2.5 Relationship graph discovery

`RelationshipGraph` can discover transitive context:

> A relates to B, and B relates to C, therefore C may be relevant to A.

It supports:

- multi-hop traversal;
- relationship-strength weighting;
- distance decay;
- confidence weighting;
- discovery of Experiences attached to related entities.

This is one of the strongest implementations of the relational part of the architecture.

There is one important limitation: current entity extraction from an Experience uses mainly `source_entity_id`. It does not yet perform full text-based entity recognition. Therefore, relationship-chain recall is strongest when the source entity is already known.

## 2.6 Recall depth

`RecallDepthEvaluator` converts the continuity score into levels:

- Level 0: no meaningful connection;
- Level 1: minimal context;
- Level 2: relevant context;
- Level 3: strong continuity;
- Level 4: deep reconstruction.

Conceptually, this is well aligned with the design.

But in the current implementation, the depth mainly changes the **number of Experiences retrieved**. The retrieval scope declares that higher levels should include relationships, history, Direction, event relations, and integrations, but `_activate_context()` generally returns only:

- relevant Experiences;
- count;
- depth;
- explanation.

So the depth model is implemented more fully as a scoring vocabulary than as a complete context-composition mechanism.

---

# 3. Does recall combine past Experiences, Relationships, Direction, and the new prompt?

## Intended behavior

Yes. The intended operation is:

> New prompt  
> + relevant past Experiences  
> + connected Entities  
> + Relationships  
> + active Directions  
> + relevant History  
> + Integration assertions  
> → activated current context

## Actual behavior

Partially.

The current prompt is used as the anchor for:

- semantic matching;
- continuity scoring;
- temporal matching;
- Direction matching;
- relationship-chain discovery.

Past Experiences are then retrieved through `MemoryControl.recall_relevant_experiences()`.

Relationships and Directions influence recall in several ways:

- they affect pattern scores;
- active Directions influence Experience relevance;
- relationship chains may add candidate Experiences;
- retention and truth metadata affect final scores.

However, the final LLM context is primarily formatted by:

- `ContinuousMemoryLayer.format_activated_context_for_llm()`

That formatter outputs:

- recall depth;
- explanation;
- pattern scores;
- relevant Experiences.

It does not normally output a structured section containing:

- the actual active Relationships;
- why each Relationship matters;
- the active Directions;
- relevant History chains;
- Integration assertions;
- relationship paths discovered through graph traversal.

Therefore the current implementation is better described as:

> **relationship- and direction-informed Experience recall**

rather than:

> **full relational Living Memory composition**

---

# 4. Historical Integration

`HistoricalIntegration` provides several important capabilities:

- typed event-to-event relationships;
- causal and temporal distinctions;
- relationship-chain traversal;
- context reconstruction;
- Integration assertions;
- relevance reactivation;
- recurring pattern detection;
- pattern assertions.

It is the correct conceptual home for:

> "What does this new Experience mean in relation to older Experiences?"

For example:

- a new Experience can continue an earlier one;
- an Experience can contradict an earlier claim;
- an old Experience can become relevant again;
- multiple Experiences can form a recurring pattern.

## Current integration gap

Although `HistoricalIntegration` exists, it is not consistently part of the ordinary recall path.

`MemoryControl.recall_relevant_experiences()` is labeled as routing through:

> Memory Control → Historical Integration → Relevance → Governance

But the method primarily:

1. loads Experience files;
2. filters by retention;
3. calls governance relevance scoring;
4. sorts the Experiences;
5. returns them.

It does not generally call `HistoricalIntegration` to build a contextual interpretation for every recall.

Historical Integration is instead used more explicitly for:

- user references to prior Experiences;
- agent response continuation;
- optional causal/evidence relations;
- important task grouping;
- recurring pattern detection.

This means Integration exists as a capability, but is not yet the universal semantic bridge between old memory and the current prompt.

---

# 5. Important implementation mismatches

## 5.1 Event relation field mismatch

`HistoricalIntegration.create_event_relation()` writes:

- `source_event_id`;
- `target_event_id`.

Some reader methods in the same module look for:

- `source_event`;
- `target`.

As a result, event relations created by the current writer may not be discoverable by all current traversal methods.

This directly affects:

- event-chain traversal;
- causal inspection;
- relation lookup;
- historical reconstruction.

## 5.2 Relationship schema mismatch

Current relationship records use fields such as:

- `subject_entity_id`;
- `object_entity_id`.

Some older governance methods look for:

- `subject`;
- `object`.

This can cause relationship relevance or traversal logic to miss relationships that are present in the current data.

`RelationshipGraph` is better aligned with the current field names than some of the older `HistoricalIntegration` and governance paths.

## 5.3 Context membership schema mismatch

Some membership records use:

- `status`.

Other paths use:

- `activation_status`.

The continuity loader primarily filters on `status`, so memberships created through a different path may not be loaded into the present-facing context as expected.

## 5.4 Direction creation is not automatic

The routing layer detects Direction-like language such as:

- build;
- create;
- implement;
- fix.

But the normal routing handler currently logs the detection instead of creating a Direction record.

Actual Direction creation is available through explicit methods such as:

- `MemoryGovernance.establish_direction()`;
- `AgentMemoryInterface.establish_new_direction()`.

Therefore, Direction is structurally supported but not fully automatic.

## 5.5 Recall output is narrower than the architecture

The design says Level 3 and Level 4 recall should include:

- relevant History;
- connected Experiences;
- Relationships;
- previous decisions;
- current Direction;
- cross-context memories;
- Integration assertions.

The current activated LLM context mostly contains:

- Experiences;
- pattern scores;
- recall explanation.

The other layers are used for analysis or are available in the canonical projection, but are not consistently presented as activated context.

---

# 6. Memory lifecycle review

The documented record lifecycle is:

> CREATE → VALIDATE → USE → UPDATE → SUPERSEDE → ARCHIVE → PRUNE

## Create — implemented

Experiences, Relationships, Directions, Integration assertions, patterns, and memberships receive identifiers and timestamps.

## Validate — partially implemented

Validation occurs through:

- routing;
- governance evaluation;
- relationship-type validation;
- provenance metadata;
- conflict detection;
- duplicate checks.

The limitation is that several validations are heuristic rather than semantic or model-based.

## Use — implemented

Validated or retained Experiences participate in:

- relevance scoring;
- semantic matching;
- Direction matching;
- relationship-chain discovery;
- governed recall.

## Update — partially implemented

Some updates correctly create new records or snapshots, but many Experience records are modified in place when routing, retention, or truth metadata changes.

The design prefers:

> new record/version + predecessor link + History record

The current system often performs:

> load existing JSON → modify fields → rewrite same file

This is adequate for metadata maintenance but is not a complete append-oriented lifecycle model.

## Supersede — partially implemented

Supersession exists for:

- conflicting Experiences;
- Directions;
- relationship status;
- some historical records.

The system can mark records as:

- historical;
- qualified;
- contested;
- superseded.

But successor/predecessor links are not uniformly implemented for every record type.

## Archive — implemented for Experiences

`LifecycleManager` archives old ephemeral Experiences while preserving metadata.

The current policy is primarily:

- ephemeral Experiences older than the archive threshold are moved to archived storage.

## Prune — implemented narrowly

Archived ephemeral Experiences can be pruned, with tombstones preserving:

- identifier;
- record type;
- provenance-related fields;
- retention decision;
- archive time;
- prune reason.

This aligns well with the lifecycle design.

## Lifecycle gap

The lifecycle manager mainly handles Experience archival and pruning. Relationships, Directions, Integration assertions, and other record classes do not yet share an equally complete lifecycle implementation.

So the lifecycle is:

> strong for Experience retention, partial for the wider memory graph.

---

# 7. Component inventory

## Runtime orchestration

- `AgentMemoryInterface`
  - Main semantic interface used by the agent.
- `ContinuousMemoryLayer`
  - Automatic relevance evaluation and context activation.
- `ContinuityLoader`
  - Reconstructs the present-facing memory structure.
- `CanonicalContinuityBuilder`
  - Composes identity, relationships, history, integration, Direction, and current context.
- `CanonicalContinuityProjectionStore`
  - Stores derived Current Understanding projections.

## Experience and interpretation

- `MemoryControl`
  - Creates Experiences.
  - Routes Experiences into memory operations.
  - Performs governed Experience recall.
  - Detects conflicts and duplicates.
- `AutomaticRelevanceEvaluator`
  - Builds candidate memory sets.
  - Coordinates relevance evaluation.
- `PatternRecognition`
  - Evaluates continuity dimensions.
- `SemanticMatcher`
  - Extracts concepts and calculates semantic similarity.
- `RecallDepthEvaluator`
  - Selects recall depth from continuity scores.

## Relationship and meaning

- `RelationshipManagerV1`
  - Creates semantic Relationships.
  - Adds evidence.
  - Updates confidence.
  - Detects contradictions.
- `RelationshipTypes`
  - Defines relationship categories, predicates, provenance, and epistemic projections.
- `RelationshipGraph`
  - Performs multi-hop graph traversal.
  - Finds transitive Experiences.
- `DirectionMatcher`
  - Matches current prompts and Experiences to active Directions.
- `HistoricalIntegration`
  - Creates typed Event Relations.
  - Creates Integration assertions.
  - Reconstructs context.
  - Detects recurring patterns.
  - Reactivates relevance.

## Governance and lifecycle

- `MemoryGovernance`
  - Determines retention and promotion.
  - Evaluates truth progression.
  - Establishes Directions.
  - Manages relationship progression.
  - Creates context memberships.
- `LifecycleManager`
  - Archives and prunes Experience records.
- `RetentionManager`
  - Stores standalone retention decisions.
- `HistoryManager`
  - Records significant semantic changes and lifecycle events.
- `AttentionControl`
  - Represents attention allocation and focus priority.

## Persistent semantic record collections

The current data model contains collections for:

- Experiences;
- Entities;
- Relationships;
- Directions;
- Event Relations;
- History events;
- Integration assertions;
- Pattern assertions;
- Context memberships;
- Contradictions;
- Unresolved questions;
- Attention allocations;
- Retention decisions;
- Archived Experiences;
- Tombstones;
- Derived continuity projections.

---

# 8. Current architecture

The rendered architecture shows the current semantic path:

> New Prompt  
> → Current Experience  
> → Memory Control  
> → Perception and Routing  
> → Experience and Relationship Sources  
> → Continuous Memory Layer  
> → Candidate Generation  
> → Pattern Recognition  
> → Semantic / Direction / Relationship Matching  
> → Recall Depth  
> → Governed Experience Recall  
> → Activated Context  
> → LLM  
> → New Response or Observation  
> → New Experience

The important architectural fact is:

> Your system has multiple memory structures, but the live LLM activation boundary is still Experience-centered.

Relationships, Directions, History, and Integration are present around that center, but they are not yet consistently composed into one explicit activated context package.

---

# 9. Alignment rating

| Area | Alignment |
|---|---|
| Experience as the basic incoming event | Strong |
| Selective rather than total recall | Strong |
| Semantic continuity detection | Strong |
| Graduated recall depth | Conceptually strong, runtime partial |
| Direction-aware relevance | Implemented, but mostly scoring-based |
| Relationship graph continuity | Implemented, but limited entity extraction |
| Historical Integration | Present, not fully on normal recall path |
| Truth-aware memory | Present |
| Retention and archival | Strong for Experiences |
| Full lifecycle across all record types | Partial |
| Context delivered to LLM | Mostly Experience-only |
| Automatic identification and Direction creation | Partial |

## Final assessment

Your project is **not going in a completely different direction**. The architectural foundation is clearly aligned with Living Memory.

The current gap is more specific:

> The design is a multi-layer continuity system, while the runtime recall path is still mainly a relevance-scored Experience retrieval system.

The next architectural improvement should not be another matcher or another storage type. The highest-value improvement is to make recall produce an explicit **Activated Living Context** containing:

- selected Experiences;
- the Relationships that explain their connection;
- active Directions that make them relevant;
- relevant History;
- Integration assertions;
- relationship-chain explanations;
- epistemic labels explaining what is observed, inferred, supported, contested, or historical.

That would make the implementation match the Living Memory lifecycle much more closely.
