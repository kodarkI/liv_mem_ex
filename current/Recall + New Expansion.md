Recall + New Expansion — Core Solution Model
1. Core concepts
New Expansion
Purpose: determine what the incoming prompt means and what needs to be investigated.
It transforms:
USER PROMPT
    ↓
meaning
    ↓
intent
    ↓
concepts
    ↓
entities
    ↓
relationships
    ↓
questions
    ↓
retrieval needs
It can generate questions that require:
current reasoning
factual knowledge
entity/relationship lookup
historical memory
temporal state
previous decisions
prior experiences
New Expansion does not itself become memory.
It creates an investigation specification.

Recall
Purpose: answer memory-directed investigation questions using the living memory.
Recall operates over:
experiences
entities
relationships
states
temporal history
truth status
provenance
uncertainty
retention
supersession
scope
Recall is therefore not simply:
semantic similarity search
It is:
contextual reconstruction of what the living memory currently knows and historically knew about a retrieval question.

Agent
Purpose: reason and act using the enriched context.
The agent receives:
original user prompt
+
expanded interpretation
+
recalled memory context
Then it performs the actual task.
If it discovers a missing memory dependency, it can invoke Recall again.

2. The fundamental flow
The cleanest version is:
USER PROMPT
     │
     ▼
┌──────────────────┐
│  NEW EXPANSION   │
│                  │
│ intent           │
│ concepts         │
│ entities         │
│ relationships    │
│ questions        │
│ retrieval needs  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ MEMORY HOOK      │
│                  │
│ execute memory  │
│ retrieval needs │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│     RECALL       │
│                  │
│ vector           │
│ graph            │
│ temporal         │
│ state            │
│ truth            │
│ provenance       │
│ governance       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ MEMORY CONTEXT   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│      AGENT       │
│                  │
│ reason           │
│ solve            │
│ act              │
└────────┬─────────┘
         │
         ▼
   memory needed?
      /       \
    yes        no
     │          │
     ▼          ▼
  RECALL      OUTPUT
     │
     └──────► AGENT
That is the architecture I would lock.

3. The crucial boundary
The cards strongly favor keeping this distinction:
Component
Main question
New Expansion
What do I need to investigate?
Recall
What does living memory know about it?
Agent
What should I do with what I know?

This prevents the three responsibilities from collapsing into one giant prompt.

4. New Expansion design
The expansion output should be structured around investigation questions, not merely expanded prose.
Conceptually:
ExpansionResult

intent
concepts[]
entities[]
relationships[]
questions[]
retrieval_needs[]
constraints[]
A retrieval need could look conceptually like:
retrieval_need:
    question:
        "What architecture did we previously choose for authentication?"

    type:
        historical_decision

    entities:
        [authentication]

    temporal_requirement:
        historical + current

    priority:
        high
The expansion layer can generate several such questions.

5. Recall design
Recall receives those retrieval needs.
It then performs:
retrieval question
       │
       ├── vector candidates
       ├── graph candidates
       ├── temporal candidates
       └── scope candidates
              │
              ▼
        candidate merge
              │
              ▼
      context reconstruction
              │
              ├── related experiences
              ├── state history
              ├── supporting memories
              ├── contradictions
              └── superseded memories
              │
              ▼
        governance/scoring
              │
              ▼
         recalled context

6. Recall should return context, not just records
This is important.
Don’t make the interface:
recall()
→ [memory1, memory2, memory3]
Make the conceptual result:
RecallResult

answering_context
current_state
historical_context
supporting_experiences
related_entities
relationships
superseded_information
provenance
confidence / epistemic status
retrieval_explanation
The agent needs meaningful context, not a bag of chunks.

7. Retrieval provenance
Every recall should be explainable internally:
Why was this memory retrieved?

semantic_match
entity_match
graph_path
temporal_relation
state_validity
truth_status
scope
direction
supersession
retention
This becomes especially valuable when debugging why the agent remembered something.

8. Two kinds of recall
The cards confirmed the earlier distinction, but now it fits cleanly into the architecture.
Initial recall
USER
 ↓
EXPANSION
 ↓
retrieval needs
 ↓
RECALL
 ↓
AGENT
This is the memory hook.
Its job is to make the agent memory-aware before substantive execution.
On-demand recall
AGENT
 ↓
"I need previous context about X"
 ↓
RECALL
 ↓
AGENT
Its job is to fill knowledge gaps discovered during execution.
They use the same Recall system.
Only the trigger differs.

9. What should NOT happen
Don’t do:
USER
 ↓
RECALL EVERYTHING
 ↓
AGENT
That defeats the purpose of expansion.
The prompt should first establish what is relevant.
Don’t do:
USER
 ↓
AGENT
 ↓
maybe memory
as the only mechanism.
You lose the opportunity to provide known context before reasoning begins.
Don’t do:
NEW EXPANSION = MEMORY RECALL
They are related but not identical.
Expansion generates investigation intent.
Recall fulfills memory investigation.

10. One unified retrieval-intent interface
This is probably the cleanest design decision from everything we’ve uncovered.
Whether the retrieval need comes from:
user
expansion
agent
it becomes:
RetrievalIntent
Then Recall doesn’t care who generated it.
For example:
RetrievalIntent
{
    question
    entities
    concepts
    temporal_scope
    relationship_scope
    required_state
    historical_depth
    priority
    source
}
source could be:
user
expansion
agent
That preserves provenance without duplicating the retrieval system.

11. Complete architecture
                ┌──────────────┐
                 │ USER PROMPT  │
                 └──────┬───────┘
                        ▼
              ┌──────────────────┐
              │  NEW EXPANSION   │
              │                  │
              │ understand       │
              │ decompose        │
              │ identify context │
              │ generate         │
              │ questions        │
              └────────┬─────────┘
                       ▼
                RETRIEVAL INTENTS
                       │
                       ▼
              ┌──────────────────┐
              │  MEMORY HOOK     │
              └────────┬─────────┘
                       ▼
              ┌──────────────────┐
              │     RECALL       │
              │                  │
              │ Vector           │
              │ Graph            │
              │ Temporal         │
              │ State            │
              │ Truth            │
              │ Provenance       │
              │ Governance       │
              └────────┬─────────┘
                       ▼
                MEMORY CONTEXT
                       │
                       ▼
              ┌──────────────────┐
              │      AGENT       │
              │                  │
              │ reason           │
              │ plan             │
              │ execute          │
              └────────┬─────────┘
                       │
                 ┌─────┴─────┐
                 │           │
          memory needed?      │
                 │           │
                yes          no
                 │           │
                 ▼           ▼
              RECALL       OUTPUT
                 │
                 └────► AGENT

12. Solution-development examples
Example A — continuation
User:
“Continue the architecture we were designing.”
Expansion discovers:
concept: retrieval architecture
entity: living memory
intent: continuation
It generates:
“What retrieval architecture had we already established?”
Recall retrieves the existing architectural history.
Agent receives that context and continues rather than restarting from zero.

Example B — new problem requiring old memory
User:
“Why is this retrieval result wrong?”
Expansion generates:
“What retrieval rules govern this memory?”
Recall finds:
scoring dimensions
state validity
supersession rules
relevant prior architecture
Agent diagnoses the problem using that context.

Example C — agent discovers a gap
User:
“Design the implementation.”
Expansion + initial recall provide the known architecture.
During execution the agent realizes:
“I need the previous decision about retention levels.”
Agent creates another RetrievalIntent.
Recall searches.
Context returns.
Agent continues.
Same memory system, different trigger.

Final compressed model
The whole concept can now be reduced to this:
NEW EXPANSION
    =
"What needs to be understood?"

RETRIEVAL INTENT
    =
"What specific information should I look for?"

RECALL
    =
"What does living memory know about it?"

MEMORY HOOK
    =
"Run relevant recall before the agent begins substantive work."

AGENT
    =
"What should I do with the combined current prompt + expanded understanding + memory?"

ON-DEMAND RECALL
    =
"What additional memory does the agent need after discovering a gap?"
And the single architectural principle tying everything together is:
New Expansion generates the questions; Recall answers the memory questions; the Agent uses the resulting context and can request further Recall whenever its reasoning discovers another memory dependency.

