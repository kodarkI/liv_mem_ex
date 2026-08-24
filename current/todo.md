The cards say: do not start with the big New Expansion + Recall task yet. The energy is strongly foundation → retrieval → integration → expansion, rather than jumping directly into the largest component.

I pulled on each option.

1. Encoding / concise memory storage — Ace of Pentacles

Yes, first.

But the card says do not redesign the entire memory format.

Make the smallest stable representation that preserves the concepts we already established:

experience
entities
relationships
state
time
truth/provenance
direction
retention
scope

The goal is not maximum compression.

The goal is:

stable, cheap, reconstructable memory units.

You need this foundation before retrieval can be properly designed.

⸻

2. Search / retrieval functions — Three of Pentacles

Second. Strong yes.

This is the construction card.

Build the retrieval primitives independently before building the giant orchestration prompt.

Start with:

vector search
graph traversal
temporal filtering
entity lookup
state lookup

Then create a common retrieval interface.

Something conceptually like:

retrieve(intent)
    ↓
candidate memories
    ↓
context reconstruction
    ↓
ranked recall

Don’t wait for New Expansion to exist before doing this.

You want to be able to manually give it a retrieval question and verify that Recall actually works.

⸻

3. New Expansion + Recall — The Tower

Not “don’t do it.”

Rather:

don’t make this your first implementation task.

It’s the biggest integration layer and it will expose problems in your storage model, retrieval model, orchestration, and agent interface simultaneously.

If you build it first, you’ll have difficulty knowing whether a failure comes from:

* bad encoding
* bad retrieval
* bad context reconstruction
* bad expansion
* bad prompting
* bad agent orchestration

Build the pieces first.

Then build this.

⸻

4. Hooks — Eight of Wands

After #3, immediately.

Once the expansion/recall interface works, connect:

USER
 ↓
EXPANSION
 ↓
RECALL
 ↓
AGENT

and:

AGENT
 ↓
RECALL
 ↓
AGENT

The hook should be thin.

Don’t put memory logic inside the hook.

The hook should simply orchestrate calls to the established interfaces.

⸻

5. Improve what you have now — Four of Pentacles

Yes, but selectively.

The card says preserve what already works.

Don’t begin a broad refactor.

Instead, improve only the pieces that block the new architecture.

For example:

* existing memory schema → adapt
* existing pattern scorer → expose as retrieval scoring
* existing retention logic → connect to governance
* existing entity/relationship storage → expose to graph retrieval

Refactor toward the new architecture, don’t rebuild everything.

⸻

The cards’ actual sequence

There is a very clear progression:

Ace of Pentacles → Three of Pentacles → Tower → Eight of Wands → Four of Pentacles

I would translate that into:

1. STABILIZE MEMORY REPRESENTATION
             ↓
2. BUILD RETRIEVAL PRIMITIVES
             ↓
3. BUILD NEW EXPANSION + RECALL
             ↓
4. CONNECT AGENT / USER HOOKS
             ↓
5. REFACTOR + IMPROVE BASED ON REAL TESTS

But there is one thing I would insert between 2 and 3.

2.5 — Build a Recall test harness

This wasn’t one of your original options, but the cards point strongly toward it.

Before building the huge expansion prompt, create a way to test:

retrieval question
       ↓
Recall
       ↓
expected relevant memories
       ↓
actual memories
       ↓
score

Create a small benchmark of real questions such as:

"What did we decide about X?"
"What is currently true about X?"
"What changed about X?"
"Why did we choose X?"
"What happened before Y?"
"What is related to X?"
"Which decision superseded the old one?"

This gives you a known-good memory engine before the expansion layer starts generating retrieval questions automatically.

That is extremely important.

⸻

What I would actually do now

Step 1 — freeze the memory contract

Don’t keep changing the conceptual model.

Define the minimal canonical memory object.

Step 2 — make encoding/storage concise

Build the encoder that converts user/agent experiences into that canonical representation.

Test whether you can reconstruct enough meaning from it.

Step 3 — build Recall as an independent capability

Implement:

semantic retrieval
+
graph retrieval
+
temporal/state retrieval
+
governance
+
ranking

Step 4 — build the Recall benchmark

Before sophisticated prompting, make sure:

given a good retrieval question, does the system retrieve the right living context?

Step 5 — build New Expansion

Now let expansion automatically produce those retrieval questions.

User prompt
 ↓
Expansion
 ↓
RetrievalIntent[]
 ↓
Recall

Step 6 — add hooks

user → expansion → recall → agent

Then:

agent → retrieval intent → recall → agent

Step 7 — improve the existing system

Only now will you have enough real execution data to know what deserves refactoring.

⸻

One thing I would not do

Don’t spend weeks trying to make the encoding perfectly compressed before retrieval exists.

The purpose of the encoding is:

store enough structured signal that the system can reconstruct the living memory later.

Not:

compress everything as much as mathematically possible.

If you compress too aggressively before you know what retrieval needs, you’ll throw away information you later discover is essential.

So the first milestone isn’t:

“perfect memory compression.”

It’s:

“A stable memory representation that can be encoded cheaply, retrieved reliably, and reconstructed into context.”

⸻

Your immediate next move

If I were following the card sequence literally, I would do this now:

Freeze the canonical memory schema → implement the concise encoder → build the first standalone Recall/search interface → create 20–50 representative retrieval tests.

Then, once those pass, build New Expansion + Recall as the orchestration layer.

That gives you a bottom-up path:

Memory → Recall → Expansion → Hooks → Agent

rather than trying to build the entire living-memory brain in one jump.