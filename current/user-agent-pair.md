Core concept: User–Agent Interaction Pair
The fundamental semantic unit is one user interaction paired with one canonical agent response.
The pair represents the completed interaction that can subsequently enter the living-memory processing flow.
1. Normal case
User record → Agent processing → Canonical agent response
Then:
Canonical response → response hook → living-memory processing via MCP
The agent’s internal execution is not itself the canonical pair.
It can contain multiple events, tool calls, retries, intermediate results, and other activity, but these belong to the processing history surrounding the pair.

2. User exists, response does not yet exist
User → awaiting_response
The user record remains valid.
Do not block the entire user record while waiting.
When the response arrives:
User → Agent response → completed pair
If the response cannot be recovered, the interaction remains explicitly incomplete.

3. Agent response exists, user record does not
Agent response → orphan_response
Do not fabricate a user record.
Preserve the response and attempt correlation/recovery.
If the original user can be established:
orphan response → recovered user → canonical pair
If not:
orphan response → permanently/unresolved unpaired record
The information is preserved without creating false history.

4. Duplicate user or agent records
Duplicates are not automatically new interactions.
They should go through:
detect → correlate → determine whether same interaction → reconcile → canonicalize
The system should preserve provenance so that the canonical record does not erase the fact that duplicate observations existed.

5. Multiple agent outputs
There is one canonical agent response per interaction pair.
Multiple things may happen during execution, but they do not automatically create multiple canonical agent responses.
Conceptually:
1 User + 1 Canonical Agent Response = 1 Interaction
Everything else is supporting execution/event history.

6. Agent explicitly wants to write memory
This is allowed, but it is not the same operation as the canonical response.
There are therefore two distinct pathways:
Interaction pathway
User → Agent → Canonical Response → Response Hook → Living Memory
and
Explicit memory-operation pathway
Agent → Explicit Memory Write → Living Memory
The response hook still exists even if an agent also has the ability to explicitly write memory.
That distinction is important because:
“The agent said it” ≠ “the agent explicitly requested that it become memory.”

The compressed state model
If I reduce the entire card reading into states, it becomes:
User received
 → awaiting_response
User + canonical response established
 → paired
Response missing but recoverable
 → recovery
Response reconstructed
 → paired_reconstructed
User missing but response exists
 → orphan_response
Duplicate detected
 → reconciliation
Cannot establish relationship
 → unresolved
And the normal successful path is simply:
user → processing → canonical_agent_response → paired → memory_processing
That’s the core.

One more card for the whole architecture:
Ace of Swords
This is the card I would want at the end of this reading.
It says stop expanding the conceptual problem and establish the clean rule.
The remaining complexity is mostly state handling, not a reason to redesign the fundamental relationship.
So the core rule I would carry into solution development is:
A user–agent interaction has one canonical user record and one canonical agent response. Processing may be parallel and may contain many internal events, but canonical pairing occurs at the interaction boundary. Missing records remain explicitly incomplete or orphaned until correlation/recovery establishes the relationship. Duplicates are reconciled rather than treated as new truth. The canonical agent response is captured by the response hook and becomes the formal input to living-memory processing, while explicit agent memory writes remain a separate operation.
