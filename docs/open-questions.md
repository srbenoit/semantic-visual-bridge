# Open Questions Registry — Semantic-Visual Bridge

**Purpose.** Tracks unresolved design questions, their dependencies on each other, and their
priority. This is the session planning tool. At any point it should be clear which questions are
unblocked (can be decided now), which are blocked (waiting on another decision), and which are
load-bearing (many things wait on them).

**Statuses.**
- `Unblocked` — can be decided now; no open prerequisites.
- `Blocked` — one or more prerequisites are still open (listed in *Blocked by*).
- `In progress` — currently being worked.
- `Closed` — decision made; see Decision Log for entry.

**Priority.** Indicated as `P1` (load-bearing; many things wait on it), `P2` (significant but
not on the critical path), `P3` (important detail; can be deferred safely).

---

## Dependency Map

```
Q-001 Block Interfaces
  └──→ Q-002 Diff Vocabulary ──┐
  └──→ Q-003 Spec Representation ──┤──→ Q-004 Negotiation Protocol ──→ Q-006 Audit Report Format
       (Q-002 and Q-003 are            └──→ Q-012 Genre Revision Handling      └──→ Q-013 Finding Emission API
        co-dependent)           
  └──→ Q-010 Content Measurement Interface
  
Q-005 Genre Taxonomy (Final)
  └──→ Q-008 Layout Algorithms per Genre ──→ Q-009 Layout Stability Mechanism
  └──→ Q-011 Variation Axis Vocabulary

Q-003 Spec Representation
  └──→ Q-007 Small Multiples Spec Representation ──→ Q-008 (partial)
  └──→ Q-015 Serialization Format

Q-014 Default vs. Inferred Tension    [depends on Q-004]
```

**Critical path:** Q-001 → {Q-002, Q-003} → Q-004 → Q-006 → Q-013

**Parallel track:** Q-005 → Q-008 → Q-009

---

## Q-001 — Block Interface Specification

**Status:** Closed → see `q001-block-interfaces.md`  
**Closed by:** Q-001 session + Spec/State structure session  
**Key decisions produced:** D-026 through D-037  
**Residual open items from this work:** SS-1, SS-4, SS-6 (tracked in
`spec-and-state-structure.md`); CC-2 (finding emission API home, tracked as Q-013)

---

## Q-002 — Diff Vocabulary

**Status:** Unblocked  
**Priority:** P1  
**Blocked by:** Q-001 ✓, Spec/State structure ✓ (both complete)  
**Blocks:** Q-004, Q-012  
**Question:** What is the complete vocabulary of operations for proposing changes to the spec?
At what granularity? How are local changes distinguished from frame-shifting changes (specifically,
genre revision)?  
**Why it matters:** The diff vocabulary determines the granularity of negotiation, the Spec
Manager's behavior, and what the LLM can express as a revision. It is the language of the
negotiation loop. Getting this wrong means either negotiations that can't express what's needed
or a vocabulary so fine-grained that the LLM is back to doing geometry.  
**Design constraints from D-016 and D-004:**
- Must support semantic diff, not text diff.
- Must support stable element identifiers (addressability).
- Must distinguish local changes from frame-shifting changes (D-024).
- Must allow partial views for different consumers.
**Open sub-questions:**
- Is the diff vocabulary declarative (describe the desired state) or operational (describe the
  change)? Trade-off: declarative is LLM-friendlier; operational makes provenance clearer.
- What is the unit of addressability? Nodes and edges? Semantic role groupings? Both?
- What triggers a frame-shift determination — the diff itself, or the Genre Resolver evaluating
  the post-diff spec?
- How are rejected diffs communicated? Structured error vs. explanation vs. alternative proposal?

---

## Q-003 — Spec Representation

**Status:** Unblocked  
**Priority:** P1  
**Blocked by:** Q-001 ✓, Spec/State structure ✓ (both complete)  
**Blocks:** Q-004, Q-006, Q-007, Q-015  
**Question:** What is the logical shape and schema of the working spec?  
**Why it matters:** The spec is the source of truth for the entire system (D-004). Its shape
determines what can be expressed, what can be diffed, and what downstream blocks can consume.
A wrong choice here propagates everywhere.  
**Axes of decision:**
- *Logical shape:* Graph vs. tree vs. document. A relational structure (D-002 Level 1) is
  naturally a graph; but trees are simpler to diff and address. Hierarchical graphs (DAGs) are
  a middle ground. The right choice depends on whether the spec is expected to represent
  genuinely non-hierarchical relations at the spec level or push that to the semantic graph
  within the spec.
- *Schema strictness:* Strict schema (easier validation, harder to extend) vs. flexible schema
  with required core (easier extension, harder to validate). The closed v1 vocabulary (D-011)
  argues for strict; the deferred extensibility argues for a core/extension split now.
- *Math and long content:* Inline vs. referenced. Inline is simpler; referenced supports
  reuse and independent versioning of mathematical content.
- *Versioning and history:* How is revision history stored — event log (diffs), snapshot
  chain, or both? Tradeoffs: event log supports auditability (D-016) and is compact; snapshot
  chain supports random access to prior states.
**Constraints from D-016:** Must exhibit all eight spec properties. Functional role typing and
optional rhetorical purpose must be first-class, not bolted on.  
**Relevant precedents to evaluate:**
- JSON-LD / RDF for graph-structured semantic content with stable identifiers.
- AST-style tree with cross-references for semi-hierarchical structures.
- Custom DSL with schema (Vega-Lite model) for constrained declarative expression.
- Property graphs (Neo4j model) for labeled edges with attributes.

---

## Q-004 — Negotiation Protocol

**Status:** Blocked  
**Priority:** P1  
**Blocked by:** Q-002, Q-003  
**Blocks:** Q-006, Q-012, Q-014  
**Question:** How does the negotiation loop actually work? When does the system request
clarification vs. proceed with assumptions? How does the LLM receive and respond to rejected
diffs? How are multi-turn refinements sequenced?  
**Why it matters:** This is the primary user-facing behavior of the system. A protocol that
requests clarification too often is unusable; one that proceeds silently too often violates
D-005.  
**Open sub-questions:**
- What is the threshold for requesting clarification vs. applying a default? (Relates to Q-014.)
- Is the protocol synchronous (one diff at a time, wait for confirmation) or does it support
  batched diffs with partial approval?
- How does the system handle the case where it successfully renders but the audit report
  contains Tier 1 findings? Does rendering block on Tier 1 resolution?
- How is the negotiation state represented — as part of the spec, separately, or in the
  conversation history?
- What does a complete negotiation turn look like end-to-end? (A concrete worked example
  would be more valuable than a protocol description.)

---

## Q-005 — Genre Taxonomy (Final)

**Status:** Unblocked  
**Priority:** P2  
**Blocked by:** Nothing (can proceed in parallel with Q-001 work).  
**Blocks:** Q-008, Q-011  
**Question:** What is the final, principled genre vocabulary? The current 17-genre working set
(D-008) needs: a principled basis for the genre count and coverage, resolution of overlap cases,
and a formal definition of what distinguishes one genre from another.  
**Why it matters:** Genre drives everything downstream — layout algorithm, encoding conventions,
audit checks, accessibility narrative structure (D-007). The taxonomy needs to be stable before
per-genre work can proceed.  
**Known problems with the current set:**
- *Overlap:* causal-graph spans "showing relation" and "showing structure." Timeline spans
  "showing process" and "showing quantity." The design doc doesn't address how overlap is
  resolved (pick one? allow multi-label? define overlap as a separate genre class?).
- *No stated basis for 17:* Is this the right level of granularity? Too coarse loses layout
  precision (hierarchy and composition have very different layout needs); too fine creates
  maintenance burden and classification ambiguity.
- *"Composition" in "showing structure":* Ambiguous — it could mean part-whole composition
  (Venn-adjacent) or compositional structure (UML component diagram). Needs disambiguation.
- *Comparison-matrix:* Occupies an odd position — it's often a table-like structure whose
  primary content is categorical relationships, not a true "relation" in the graph sense.
**Approach:** Review each genre for: (a) distinct layout algorithm, (b) distinct encoding
conventions, (c) distinct accessibility narrative, (d) cases in practice that clearly require
it. Genres that don't need all four may be modifiers or sub-genres rather than top-level genres.
**Relevant literature:** Larkin & Simon on diagrammatic vs. sentential representations; Bertin's
semiology of graphics for encoding vocabulary; Engelhardt's taxonomy of graphic information.

---

## Q-006 — Audit Report Format

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-002 (findings reference spec elements by stable ID), Q-003 (same),
Q-004 (report structure relates to negotiation protocol)  
**Blocks:** Q-013  
**Question:** What is the concrete serialization format of the audit report? How is it structured
for LLM consumption? How does it accumulate across turns?  
**Why it matters:** The audit report is the "visual back to language" bridge — the most novel
block in the architecture. Its format determines whether the negotiation loop can actually run
in language without the LLM seeing the visual (the quality target from the design doc).  
**Design constraints from D-015, D-006:**
- Three-tier structure.
- Cumulative: unresolved findings persist across turns until addressed or dismissed.
- Every finding traces to a specific decision or measurement.
- Findings reference spec elements by stable identifier.
- Report is an account, not a justification; does not prescribe LLM behavior.
**Open sub-questions:**
- Is the report a structured data format (JSON) or a structured natural language format optimized
  for LLM readability? Or a template that renders to both?
- How are cross-cutting emergent findings (intent drift, disproportionate elision) represented
  when they span multiple blocks?
- What is the "turn" boundary for accumulation? Does each render produce a new report, or does
  the report grow incrementally?

---

## Q-007 — Small Multiples Spec Representation

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-003  
**Blocks:** Q-008 (partial — layout algorithms need to handle small-multiple instantiation)  
**Question:** How are small multiples represented in the spec? How is coordinated scale and
shared encoding enforced across instances?  
**Design constraint from D-009:** Small multiples is a modifier, not a genre. The spec must
represent: the repeated genre, the set of instances, the shared encoding contract (same axes,
aligned scales, common template), and the varying content per instance.  
**Open sub-questions:**
- Is the shared encoding contract part of the spec or derived by the Layout Engine from the
  instances?
- How are instances addressed — do they have stable IDs independent of their content?
- How does a diff that adds or removes an instance work? Does it invalidate the shared encoding
  contract?

---

## Q-008 — Layout Algorithms per Genre

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-005  
**Blocks:** Q-009  
**Question:** For each genre in the final taxonomy, which specific layout algorithm is used?
What are the algorithm's parameters, and how do they map to spec properties?  
**Design constraint from D-012:** Modular per genre; no universal solver. The genre-to-algorithm
binding lives in the Genre Resolver's policy output.  
**Approach:** Per genre: identify the algorithm family (hierarchical, force-directed, constraint-
based, geometric, etc.), select a specific algorithm or algorithm class, identify the parameters
that derive from the spec (node ordering from emphasis, grouping from role typing, etc.), and
identify the quality measurements the Layout Engine exposes for that algorithm.  
**Notes:** This is where the user's Java/IntelliJ implementation intuition is most directly
relevant. Algorithm selection has major implementation cost implications.

---

## Q-009 — Layout Stability Mechanism

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-008 (mechanism likely differs per algorithm family)  
**Blocks:** Nothing (terminal in the dependency graph for v1)  
**Question:** How specifically does the Layout Engine implement the stability principle (D-013)?
What does "minimize disruption to prior layout" mean formally, and how is it computed?  
**Why this is hard:** Mental-map preservation is an open research problem in graph drawing.
The design doc acknowledges this is a principle without specifying a mechanism.  
**Known approaches to evaluate:**
- *Anchored layout:* Some nodes are pinned to prior positions; layout solves around them.
  Simple but can produce poor layouts when anchor positions conflict with optimal layout.
- *Incremental layout:* Layout algorithm initialized from prior solution rather than random.
  Works well for force-directed; less applicable to hierarchical algorithms.
- *Correspondence-then-minimize:* Compute optimal layout independently; then find a rigid
  transformation (or permutation) that minimizes displacement from prior layout. Clean but
  may not preserve relative structure.
- *Constrained re-layout:* Add soft constraints that penalize deviation from prior positions.
  Flexible but adds parameters that need tuning.
**Likely answer:** No single mechanism; per-algorithm-family solutions. The design should
commit to the mechanism class per genre rather than a universal approach.

---

## Q-010 — Content Measurement Service Interface

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-001  
**Blocks:** Q-008 (layout algorithms need to know what they can query)  
**Question:** What is the interface of the Content Measurement Service? What content types does
it handle? What does it return? How are iterative measurement cycles managed internally?  
**Design constraints from D-014, D-017, D-019:**
- Must handle plain text, math (MathML/LaTeX-bearing nodes), and icons at minimum.
- Returns unit-invariant measurements (D-019).
- Iterative measurement cycles (for complex math layout) must be managed internally without
  exposing iteration to callers.
**Open sub-questions:**
- Does the service return bounding boxes only, or also baseline positions, ascenders/descenders,
  and other typographic metrics needed for alignment?
- For math expressions, does it return the full laid-out structure (for use in rendering) or
  just the bounding box (for use in layout)? The former creates a tighter coupling to the
  Renderer's typesetting system.
- Is measurement synchronous or does it support batching? (Layout algorithms typically need
  many measurements; round-trip latency matters.)
- What is the caching model? Content measurement is deterministic for given content and style
  context; caching is important for performance.

---

## Q-011 — Variation Axis Vocabulary

**Status:** Blocked  
**Priority:** P3  
**Blocked by:** Q-005 (genre variations need final genre list), Q-003 (variations are spec diffs)  
**Blocks:** Nothing (terminal)  
**Question:** What is the complete vocabulary of variation axes? For each axis, what values are
possible, and how is breadth parameterized?  
**Candidate axes from design doc:** Layout (algorithm, aspect ratio, grouping, edge routing);
Encoding (palette, line weight, typography, emphasis scheme); Genre (when genre confidence is
low, render under multiple plausible genres); Structural (cycle-breaking choices, elision choices,
decomposition choices).  
**Open sub-questions:**
- Are axes independent (any combination is valid) or do some axes interact (e.g., genre variation
  implies layout variation)?
- How is breadth parameterized — as a continuous parameter per axis, a discrete scale, or a
  named level (minor/moderate/major)?
- Some axes (genre, structural) produce qualitatively different visuals; others (encoding)
  produce stylistic variations. Should the Variation Generator treat these differently?

---

## Q-012 — Genre Revision Handling in the Negotiation Loop

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-002, Q-004  
**Blocks:** Nothing (terminal)  
**Question:** When a diff would shift the spec out of its current genre, how exactly does the
system respond? The design doc identifies three strategies (reject, auto-promote, surface
implication) and selects the third — but the mechanism is unspecified.  
**Design constraint from D-024:** The diff vocabulary must distinguish local changes from
frame-shifting changes. Genre commitment is recorded in the spec; downstream blocks read it
from the spec.  
**Open sub-questions:**
- How does the system detect that a diff is frame-shifting vs. local? Is this determined by
  the Spec Manager (structural), the Genre Resolver (semantic), or both?
- What exactly does "surface the implication" look like in practice? A Tier 1 audit finding?
  A blocking clarification request? A rendered comparison of the visual under both genres?
- Can a genre revision be proposed explicitly (as a first-class diff operation) separate from
  the content changes that motivated it?

---

## Q-013 — Finding Emission API

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-003 (findings reference spec elements by stable ID), Q-006  
**Blocks:** Nothing (terminal)  
**Question:** What is the shared API used by all blocks to emit findings to the Audit block?
What structure does a finding have? How is tier assignment enforced?  
**Design constraint from D-015, D-006:**
- Findings must reference spec elements by stable identifier, not by paraphrase or position.
- Tier assignment by emitting block, with re-tiering possible by Audit block.
- Every finding must trace to a specific decision or measurement.
- The API should enforce finding structure and discourage misuse (design doc notes this as
  partly an architectural issue).
**Note:** This is also the cross-cutting finding detection point. The Audit block receives
full block input/output, not just findings — the API needs to support this richer data flow
in addition to individual findings.

---

## Q-014 — Default vs. Inferred Tension in Intent Capture

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-004  
**Blocks:** Nothing (terminal)  
**Question:** How is the tension between D-005 (no silent inference) and usability resolved?
Every underspecified element requiring a negotiation round is impractical. A "reasonable
default" mode for low-stakes decisions is suggested in the design doc but not defined.  
**The tension:** D-005 prohibits silent inference. But a spec with 20 underspecified elements
that generates 20 clarification requests before rendering is unusable. Some defaults are
genuinely low-stakes (edge routing style); others matter a lot (genre choice, emphasis
assignments).  
**Possible resolution approaches:**
- *Stakes-based threshold:* Clarify when the decision affects meaning (would generate a Tier 1
  finding if wrong); apply defaults when it affects only presentation (Tier 2 or 3).
- *Explicit default declaration:* Blocks may apply defaults from a declared default set without
  clarification, but defaults applied are always reported as Tier 3 findings.
- *Confidence-gated:* Apply defaults when confidence is above a threshold; request clarification
  when below. The threshold is configurable per deployment context.
**Note:** The resolution here directly affects the feel of the negotiation loop more than any
other single decision.

---

## Q-015 — Spec Serialization Format

**Status:** Blocked  
**Priority:** P2  
**Blocked by:** Q-003  
**Blocks:** Nothing directly; affects GitHub repo structure and tooling choices  
**Question:** What is the concrete serialization format for the spec — the on-disk/in-transit
representation?  
**Candidates:**
- *JSON:* Universal tooling, good library support, poor human readability for complex structures,
  no comments, diff noise from whitespace/ordering.
- *YAML:* More human-readable, comments supported, but parsing is fragile and the format has
  well-known edge cases.
- *Custom DSL:* Maximum expressiveness, best human readability, maximum tooling cost. Appropriate
  if the spec has a natural textual structure.
- *JSON with schema (JSON Schema / Zod):* Good validation tooling, reasonable human readability,
  works well with Claude Code generation.
**Constraints:** Must support stable addressable identifiers (D-016); must be diff-friendly in
a version control context (git diff); must be parseable by Java (user's implementation language).
**Note:** JSON with strict schema is the pragmatic choice for Claude Code integration and Java
implementation. Custom DSL is architecturally ideal but high cost. Decision should be deferred
until Q-003 is resolved, since the logical shape of the spec constrains the serialization options.

---

## Deferred to v2+

These are tracked here so they don't get lost, but are explicitly not in scope for v1 design work.

- **Confections** — genuine multi-genre composition where no single genre dominates (D-010).
- **Animation** — temporal dimension in the spec, state evolution model (D-020).
- **Interactivity** — UI model layered on visual model (D-020).
- **Pedagogical sequences** — coordinated sequences of visuals (D-020).
- **Real-time/streaming data** — assumes stable spec; live data breaks that assumption (D-020).
- **Genre vocabulary extensibility mechanism** — closed for v1 (D-011).
- **Cartographic maps** — out of scope entirely, not deferred (D-020).

---

*Registry updated after Q-001 + Spec/State structure sessions. Q-001 closed.
Q-002 and Q-003 now unblocked — next session target.*
