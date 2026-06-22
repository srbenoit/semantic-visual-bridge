# Open Questions Registry — Semantic-Visual Bridge

**Purpose.** Tracks unresolved design questions, their dependencies on each other, and their
priority. This is the session planning tool. At any point it should be clear which questions
are unblocked, which are blocked, and which are load-bearing.

**Statuses.**
- `Unblocked` — can be decided now; no open prerequisites.
- `Blocked` — one or more prerequisites are still open.
- `In progress` — currently being worked.
- `Closed` — decision made; see Decision Log for entry.

**Priority.** `P1` = load-bearing (many things wait on it). `P2` = significant but not on
critical path. `P3` = important detail, can be deferred safely.

---

## Dependency Map

```
Q-001 Block Interfaces [CLOSED]
  └──→ Q-002 Diff Vocabulary ──┐
  └──→ Q-003 Spec Representation ──┤──→ Q-004 Negotiation Protocol ──→ Q-006 Audit Report Format
       (Q-002 and Q-003 are            └──→ Q-012 Genre Revision Handling    └──→ Q-013 Finding Emission API
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

**Status:** Closed
**Closed by:** Q-001 session + Spec/State structure session
**Key decisions produced:** D-026 through D-037
**Artifacts:** `q001-block-interfaces.md`, `spec-and-state-structure.md`

---

## Q-002 — Diff Vocabulary

**Status:** Unblocked (Q-001 closed)
**Priority:** P1
**Blocks:** Q-004, Q-012
**Question:** What is the complete vocabulary of operations for proposing changes to the spec?
At what granularity? How are local changes distinguished from frame-shifting changes?
**Why it matters:** The diff vocabulary is the language of the negotiation loop. Getting this
wrong means either negotiations that can't express what's needed, or a vocabulary so fine-grained
that the LLM is back to doing geometry.
**Design constraints:**
- Must support semantic diff, not text diff.
- Must support stable element identifiers (D-016).
- Must distinguish local changes from frame-shifting changes (D-024).
- Must include source-type tagging for provenance (D-028 — GR proposes, LLM proposes, system commits).
- Must address all six spec slots distinctly (D-027).
**Open sub-questions:**
- Declarative (describe desired state) vs. operational (describe the change)? Declarative is
  LLM-friendlier; operational makes provenance clearer.
- What is the unit of addressability? Nodes and edges? Semantic role groupings? Both?
- What triggers a frame-shift determination — the diff itself, or the Genre Resolver evaluating
  the post-diff spec?
- How are rejected diffs communicated? Structured error vs. explanation vs. alternative proposal?

---

## Q-003 — Spec Representation

**Status:** Unblocked (Q-001 closed)
**Priority:** P1
**Blocks:** Q-004, Q-006, Q-007, Q-015
**Question:** What is the logical shape and schema of the working spec?
**Why it matters:** The spec is the source of truth for the entire system (D-004). Its shape
determines what can be expressed, what can be diffed, and what downstream blocks can consume.
**Axes of decision:**
- *Logical shape:* Graph vs. tree vs. DAG. Genuinely relational content (D-002 Level 1) is
  naturally a graph; trees are simpler to diff and address; hierarchical graphs (DAGs) are a
  middle ground.
- *Schema strictness:* Strict schema vs. flexible schema with required core. Closed v1
  vocabulary (D-011) argues for strict; deferred extensibility argues for a core/extension
  split now.
- *Math and content references:* Inline vs. referenced. Inline is simpler; referenced supports
  reuse and independent versioning.
- *Versioning and history:* Event log (diffs), snapshot chain, or both?
**Constraints from D-016, D-027, D-032, D-013:** Must exhibit all eight spec properties. Must accommodate six slots with distinct write permissions. Must support a Data Section with typed bulk content. Math must be a first-class content type.
**Note:** Q-002 and Q-003 are co-dependent and should be worked in the same session.

---

## Q-004 — Negotiation Protocol

**Status:** Blocked
**Priority:** P1
**Blocked by:** Q-002, Q-003
**Blocks:** Q-006, Q-012, Q-014
**Question:** How does the negotiation loop actually work? When does the system request
clarification vs. proceed with assumptions? How does the LLM receive and respond to rejected
diffs?

---

## Q-005 — Genre Taxonomy (Final)

**Status:** Unblocked
**Priority:** P2
**Blocks:** Q-008, Q-011
**Question:** What is the complete, finalized genre vocabulary for v1? How are genres named,
described, and distinguished from adjacent genres?
**Note:** Can be worked in parallel with Q-002/Q-003. Does not depend on spec representation.

---

## Q-006 — Audit Report Format

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-004 (partially), Q-003
**Blocks:** Q-013
**Question:** What is the structure of the LLM-readable audit report? What does Slot 6 look
like in concrete form?

---

## Q-007 — Small Multiples Spec Representation

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-003
**Blocks:** Q-008 (partial)
**Question:** How does the spec represent a small multiples composition — multiple instances
of the same genre applied to different data slices, with shared axes and scales?

---

## Q-008 — Layout Algorithms per Genre

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-005, Q-007 (partial)
**Blocks:** Q-009
**Question:** For each genre in the v1 taxonomy, what is the layout algorithm family?
What parameters does it expose? What does it declare in the Autonomy Charter?

---

## Q-009 — Layout Stability Mechanism

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-008
**Question:** When a spec change causes re-layout, how is layout stability preserved to
prevent visually jarring changes? What is the stability contract?

---

## Q-010 — Content Measurement Interface

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-001 [now closed; formally unblocked but depends on Q-003 for content types]
**Question:** What is the caller interface for the Content Measurement Service? What content
types does it measure? How does it receive render context?

---

## Q-011 — Variation Axis Vocabulary

**Status:** Blocked
**Priority:** P3
**Blocked by:** Q-005
**Question:** What are the named axes along which the Variation Generator can produce
alternatives? How are these axes defined per genre?

---

## Q-012 — Genre Revision Handling

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-002, Q-004
**Question:** When a genre revision is proposed (frame-shifting change), what is the exact
protocol? What state is preserved? What must be re-derived?

---

## Q-013 — Finding Emission API

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-006
**Question:** What is the programmatic interface by which pipeline blocks emit findings to
the Audit block? How are finding tier, source, and payload specified?

---

## Q-014 — Default vs. Inferred Tension

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-004
**Question:** When should the pipeline apply a default vs. request clarification from the
LLM? What is the threshold rule?
**Note:** Likely resolved by a stakes-based threshold using the finding tier structure —
clarify when a wrong choice would generate a Tier 1 finding, apply defaults otherwise.
Strong candidate for this conclusion; treat as near-decided pending Q-004.

---

## Q-015 — Spec Serialization Format

**Status:** Blocked
**Priority:** P2
**Blocked by:** Q-003
**Question:** What is the concrete on-disk/in-transit serialization format for the spec?
**Candidates:** JSON with strict schema, YAML, custom DSL, JSON-LD.
**Constraints:** Must be diff-friendly in git; must be parseable by Java; must support stable
addressable identifiers (D-016).
**Note:** JSON with strict schema is the pragmatic front-runner for Claude Code integration
and Java implementation, but deferred until Q-003 settles the logical shape.

---

## Deferred to v2+

- Confections — genuine multi-genre composition (D-010)
- Animation — temporal dimension (D-020)
- Interactivity — UI model (D-020)
- Pedagogical sequences (D-020)
- Real-time/streaming data (D-020)
- Genre vocabulary extensibility (D-011)
- Cartographic maps — out of scope entirely (D-020)
- Branching version history (D-031)

---

*Registry updated after Q-001 session. Q-001 closed. Q-002 and Q-003 now unblocked.*
*Next update: after Q-002/Q-003 session.*
