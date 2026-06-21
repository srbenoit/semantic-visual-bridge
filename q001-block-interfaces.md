# Block Interface Specification — Q-001

**Status:** Draft for review  
**Addresses:** Q-001 in Open Questions Registry  
**Depends on:** D-003 (six-block architecture), D-004 (spec as source of truth),
D-005 (no silent inference), D-006 (failure modes as first-class outputs),
D-015 (three-tier finding classification), D-016 (eight spec properties)

---

## How to Read This Document

Each component entry covers:
- **Receives** — what arrives at this component, from where, in what form
- **Emits** — what leaves this component, to where, in what form
- **Shared state** — any persistent state this component reads or writes outside of message passing
- **Failure modes** — what the component emits when it cannot proceed normally
- **Design issues surfaced** — assumptions or conflicts the interface analysis reveals

"Form" is currently expressed as a description, not a schema — that waits on Q-003
(Spec Representation). The goal here is to establish what crosses each boundary and
in what direction, not yet how it is encoded.

---

## Notation

```
[TYPE: description]
  TYPE = SPEC | DIFF | FINDING | LAYOUT | RENDER | MEASURE | POLICY | SIGNAL | CLARIFICATION
```

- `SPEC` — the working spec, or a view of it
- `DIFF` — a proposed change to the spec
- `FINDING` — a structured audit finding (tier-tagged)
- `LAYOUT` — resolved abstract geometry (unit-invariant)
- `RENDER` — concrete visual output (SVG or other)
- `MEASURE` — content dimensions and typographic metrics
- `POLICY` — genre-derived parameters for downstream blocks
- `SIGNAL` — a control signal (halt, proceed, request clarification)
- `CLARIFICATION` — a structured request for more information, directed at LLM or human

---

## Block 1: Intent Capture

**Responsibility recap:** Translate the LLM's natural-language or semi-structured visual
request into a normalized semantic spec or spec diff. Absorbs vocabulary flexibility.
Active interrogator, not a passive executor.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| LLM / Human | Visual request or refinement utterance | Natural language or semi-structured | Initial request or a turn in the negotiation loop |
| Spec Manager | Current spec snapshot | `SPEC` (read-only view) | Intent Capture needs to know current state to interpret "add a node" or "change the title" correctly |
| Spec Manager | Diff rejection notice | `SIGNAL` + structured reason | When a prior Intent Capture output was rejected; Intent Capture may need to re-interpret or escalate to clarification |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| Spec Manager | Proposed spec change | `DIFF` | Either an initial spec (first turn) or an incremental diff (subsequent turns) |
| LLM / Human | Clarification request | `CLARIFICATION` | When multiple interpretations are plausible or confidence is below threshold; see Design Issue IC-1 |
| LLM / Human | Normalized interpretation echo | Structured natural language | Always emitted for confirmation before diff is submitted; "I understand you want X — confirming before I proceed" |
| Audit Block | Intent capture findings | `FINDING` (tier-tagged) | Vocabulary mappings applied, ambiguities resolved, interpretations chosen |

### Shared State

- Read: Current spec (via Spec Manager read interface)
- Write: None — Intent Capture proposes; it does not commit

### Failure Modes

| Condition | Response |
|-----------|----------|
| Request maps to out-of-scope category (D-020) | `SIGNAL` halt + structured explanation to LLM/human; does not submit a diff |
| Multiple plausible interpretations, none clearly dominant | `CLARIFICATION` to LLM/human; does not guess |
| Confidence below threshold on genre or entity extraction | `CLARIFICATION` or tentative diff with explicit uncertainty flag |
| Diff rejected by Spec Manager with reason | Re-interpret or escalate; if re-interpretation also fails, `CLARIFICATION` to LLM/human |
| Request is syntactically parseable but semantically empty ("make it better") | `CLARIFICATION` requesting more specific intent |

### Design Issues Surfaced

**IC-1 — Clarification threshold is unspecified.**
Intent Capture must decide when to clarify vs. proceed with a tentative diff. This is Q-014
in the Open Questions Registry. The interface is clean; the policy is not yet defined.

**IC-2 — Intent Capture needs spec read access, creating a dependency.**
Intent Capture is described as the "front end" but it needs to read current spec state to
interpret refinement utterances correctly. This means the Spec Manager must expose a read
interface that Intent Capture can call before submitting a diff. Not a problem architecturally,
but it must be made explicit — Intent Capture is not stateless.

**IC-3 — Who owns the clarification conversation?**
When Intent Capture issues a clarification request, the LLM responds. Does that response come
back to Intent Capture directly, or does it go through the full pipeline again? If the former,
Intent Capture needs a conversational sub-loop. If the latter, the negotiation protocol must
distinguish "clarification response" from "new request." This needs resolution in Q-004.

---

## Block 2: Spec Manager

**Responsibility recap:** Maintain the working spec as canonical state. Accept proposed
revisions; commit or reject with structured reasons. Passive custodian — does not interpret,
infer, or generate.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| Intent Capture | Proposed diff | `DIFF` | Initial spec creation or incremental change |
| Genre Resolver | Genre commitment diff | `DIFF` | Special case: Genre Resolver proposes a write to a specific spec field (genre, confidence, policy) |
| Human (direct edit mode) | Direct spec edit | `DIFF` or full spec replacement | D-021: human can edit spec directly; edits have different provenance than LLM-proposed diffs |
| Variation Generator | Variant spec selection | `DIFF` | User selects a variation; its diff is committed via the Spec Manager |
| Any block | Read request | `SIGNAL` | Any downstream block can request the current spec or a partial view |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| Intent Capture | Diff acceptance | `SIGNAL` | Confirmation that a diff was committed |
| Intent Capture | Diff rejection | `SIGNAL` + structured reason | Schema violation, conflict, invalid transition |
| Genre Resolver | Current spec | `SPEC` (full, read-only) | Triggered after each committed diff |
| Layout Engine | Current spec | `SPEC` (full, read-only) | Triggered after genre is committed |
| Accessibility Renderer | Current spec | `SPEC` (partial view) | May not need full geometric spec |
| Variation Generator | Current spec | `SPEC` (full, read-only) | On request, for variation generation |
| Audit Block | Diff history entry | `FINDING` (tier-tagged) + provenance | Applied diffs, rejected diffs, validation findings; includes who proposed (LLM vs. human) |
| Any requesting block | Spec view | `SPEC` (partial or full) | Read interface; does not trigger pipeline |

### Shared State

- Owns: Current spec (canonical)
- Owns: Revision history with provenance
- Owns: Stable element identifier registry

### Failure Modes

| Condition | Response |
|-----------|----------|
| Diff violates schema | Rejection with specific schema violation |
| Diff conflicts with existing spec state | Rejection with conflict description (which element, which property) |
| Diff would create an invalid intermediate state | Rejection with reason; or atomic multi-step diff support (see Design Issue SM-1) |
| Diff attempts to change a locked element (e.g., human-authored content) | Rejection with provenance explanation |
| Spec reaches an invalid state despite validation (shouldn't happen; defensive) | Halt + rollback to prior valid state + Tier 1 finding |

### Design Issues Surfaced

**SM-1 — Atomic multi-step diffs.**
Some changes that are logically single operations require multiple spec edits. Example:
renaming an entity that appears in multiple relations. Should the diff vocabulary support
atomic compound diffs, or should the Spec Manager handle cascading updates internally?
If the Spec Manager handles cascades, it's no longer a purely passive custodian. This
needs resolution in Q-002 (Diff Vocabulary).

**SM-2 — Genre Resolver needs write access.**
The design doc states "Genre commitment is recorded in the spec; downstream blocks read it
from the spec." This means the Genre Resolver must write to the spec via the Spec Manager.
But the Spec Manager is described as accepting diffs only from Intent Capture and humans.
Resolving this requires either: (a) Genre Resolver submits a diff through the normal diff
pathway, clearly tagged as system-generated; or (b) the Spec Manager has a separate
internal-write pathway for system blocks. Option (a) is cleaner because it maintains
provenance uniformly. Option (b) is more efficient but creates a two-tier write model.
**This is a load-bearing design question that must be resolved before Q-003.**

**SM-3 — Prior layout is not in the spec, but needs to survive revision cycles.**
The Layout Engine needs the prior layout for stability (D-013). The prior layout is not
spec content (it is a geometric artifact, not semantic content). The Spec Manager is the
natural owner of persistent state, but this is not semantic state. Options: (a) Spec Manager
stores the prior layout as an opaque attachment; (b) Layout Engine owns its own prior-layout
cache keyed on spec version; (c) a separate artifact store. See Design Issue LE-2.

---

## Block 3: Genre Resolver

**Responsibility recap:** Classify the spec into a genre; bind the downstream contract
(layout algorithm, encoding conventions, audit checks). The most consequential semantic
choice in the system.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| Spec Manager | Current spec | `SPEC` (full, read-only) | Triggered after each committed diff that could affect genre classification |
| (Implicit) | Prior genre classification | Read from spec | Genre is recorded in the spec (SM-2 above); Genre Resolver reads its own prior output |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| Spec Manager | Genre commitment | `DIFF` (genre field + confidence + alternatives) | Writes genre classification back to spec; see SM-2 |
| Layout Engine | Downstream policy | `POLICY` | Layout algorithm binding, encoding conventions, quality measurement targets, audit check list. See Design Issue GR-1. |
| Audit Block | Genre resolution findings | `FINDING` (tier-tagged) | Chosen genre and rationale, alternatives considered, confidence level, any mismatches detected between hint and structural analysis |
| LLM / Human (via Audit) | Clarification request (low confidence) | `CLARIFICATION` | When genre confidence is below threshold; may block pipeline |

### Shared State

- Read: Current spec (via Spec Manager)
- Write: Genre commitment field in spec (via Spec Manager diff — SM-2)

### Failure Modes

| Condition | Response |
|-----------|----------|
| Spec is too sparse to classify | `CLARIFICATION` request; pipeline halts at this block |
| Genre confidence below threshold | Tentative classification with explicit flag + Tier 1 finding; may trigger clarification |
| Unrecognized genre name in hint | Map to nearest known genre with mapping reported as Tier 2 finding; or refuse and request clarification if mapping confidence is low |
| Genre mismatch: structural features inconsistent with proposed genre | Tier 1 finding; surface specifically enough to make the right correction obvious |
| Diff would shift genre (frame-shifting change) | Surface implication to LLM; do not silently auto-promote (D-024) |

### Design Issues Surfaced

**GR-1 — Does the Policy object go in the spec or in a separate channel?**
The `POLICY` emitted by Genre Resolver (layout algorithm binding, encoding conventions,
audit checks) needs to reach the Layout Engine. Two options:
- (a) Policy is written into the spec by Genre Resolver (alongside the genre commitment).
  Clean, consistent, traceable; the spec remains the single source of truth. But it means
  the spec contains implementation-level parameters (algorithm selection), which muddies
  the semantic/geometric separation.
- (b) Policy is emitted as a separate out-of-band object that flows from Genre Resolver
  to Layout Engine directly, not via the spec. Cleaner semantically — the spec stays at
  Levels 1-2; policy is a Level 3 concern. But the spec is no longer the complete record
  of the pipeline's state.

**This is a meaningful architectural choice.** Option (a) preserves D-004 (spec as source
of truth) most completely. Option (b) preserves the semantic level separation most cleanly.
A middle path: policy is derived deterministically from the genre commitment and is therefore
redundant to store — but this only holds if the derivation is stateless and reproducible.

**GR-2 — When does Genre Resolver run?**
Currently implied: after every committed diff. But genre classification is expensive (if
using a model) and genre rarely changes. A trigger model (run on: first diff; any diff that
touches entity types, relation types, or explicit genre hints; explicit genre revision request)
is more efficient but requires the Spec Manager to classify diffs by their likely effect on
genre. This is a performance concern, not an architectural one, but worth flagging early.

**GR-3 — Genre Resolver reads its own prior output from the spec.**
This creates a subtle state dependency: the Genre Resolver's input includes its own previous
decision. This is correct behavior (the prior genre is a strong prior for the next
classification), but it means Genre Resolver is not stateless. The spec carries this state,
which is fine — but it means re-running Genre Resolver on an older spec snapshot will produce
different results than the original run if the genre field was populated. Relevant for
audit replay and debugging.

---

## Block 4: Layout Engine

**Responsibility recap:** Translate semantic constraints into geometric facts. Produce a
resolved abstract layout (unit-invariant) complete enough to render but free of
presentational concerns. The membrane between meaning and pixels.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| Spec Manager | Current spec with genre committed | `SPEC` (full, read-only) | Triggered after genre is committed |
| Genre Resolver | Downstream policy | `POLICY` | Algorithm binding, encoding conventions, quality targets |
| Content Measurement Service | Content dimensions | `MEASURE` | Label sizes, math expression bounding boxes and metrics; queried on demand during layout |
| Layout Engine (self, prior run) | Prior layout | `LAYOUT` (prior) | For stability (D-013); see Design Issue LE-2 |
| (Implicit) | Environmental parameters | Configuration | Target dimensions, presentation context; not from spec |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| Renderer | Resolved layout | `LAYOUT` | Positions, sizes, edge geometry, layout-derived structure, spatial annotations — all unit-invariant |
| Accessibility Renderer | Resolved layout | `LAYOUT` (possibly partial) | May not need full geometric detail |
| Content Measurement Service | Measurement requests | `SIGNAL` + content node | Queries during layout; responses arrive synchronously or batched |
| Audit Block | Layout findings | `FINDING` (tier-tagged) | Algorithm used, constraints satisfied/violated, stability assessment, quality measurements (edge crossings, whitespace balance, label collisions) |

### Shared State

- Read: Current spec (via Spec Manager), prior layout (see LE-2)
- Write: Prior layout cache (for next run)
- Does not write to spec

### Failure Modes

| Condition | Response |
|-----------|----------|
| Conflicting constraints | Structured diagnosis as Tier 1 finding; halt; does not emit a best-guess layout |
| Content too dense for target dimensions | Elision decision required; surface as Tier 1 finding; may request clarification on what to elide |
| Layout algorithm fails (e.g., no valid topological sort in cyclic graph) | Tier 1 finding with specific diagnosis; pipeline halts |
| Constraint satisfaction degrades quality below threshold | Tier 2 finding with quality measurements; does emit layout but flags it |
| Stability cannot be preserved without violating a constraint | Tier 2 finding; constraint takes priority over stability |

### Design Issues Surfaced

**LE-1 — Environmental parameters are not in the spec.**
Target dimensions and presentation context are inputs to the Layout Engine but are not
semantic content — they don't belong in the spec. But they affect the layout, which means
the same spec can produce different layouts in different contexts. The audit report needs
to record these parameters to make layout results reproducible. Where do they live?
Options: (a) a separate environment config object passed at render time; (b) a rendering
context attached to the spec (distinct from semantic content). This needs a home.

**LE-2 — Prior layout storage is unresolved.**
The Layout Engine needs the prior layout for stability (D-013). This is not spec content.
Three options:
- (a) Spec Manager stores it as an opaque versioned attachment (prior layout for spec
  version N is stored alongside spec version N).
- (b) Layout Engine owns a cache, keyed on spec content hash. Simpler; doesn't burden
  the Spec Manager. But cache invalidation on spec edits must be managed carefully.
- (c) Prior layout is embedded in the `LAYOUT` output and passed back as input next turn
  (caller holds state). Clean but requires the orchestrating layer to manage this.

Option (b) is pragmatically cleanest. Option (a) is most consistent with D-004.

**LE-3 — Content Measurement is synchronous or batched?**
Layout algorithms typically need many measurements in sequence (measure a node, place it,
measure the next). If each measurement is a synchronous call, latency compounds. A batching
model (request all measurements upfront, then run layout) requires the algorithm to enumerate
all content nodes before beginning — possible for some algorithms, not for others (e.g.,
incremental force-directed that adds nodes one at a time). This affects the Content
Measurement Service interface (Q-010) and the Layout Engine's internal algorithm structure.

---

## Block 5: Renderer

**Responsibility recap:** Convert the resolved layout into visual output (SVG or other),
handling all presentational concerns — typography, color, line weights, math typesetting.
Largely conventional once upstream blocks do their jobs.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| Layout Engine | Resolved layout | `LAYOUT` | Abstract geometry; unit-invariant |
| (Configuration) | Theme specification | Theme object | Style parameters: palette, typography, line weights, etc. LLM never touches this (D-018) |
| (Configuration) | Target format | `SIGNAL` | SVG, print, etc. |
| Content Measurement Service | Precise glyph metrics | `MEASURE` | May re-query CMS for precise metrics not needed during layout but needed for final rendering (e.g., exact baseline positions) |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| Human / LLM (via delivery layer) | Rendered visual | `RENDER` | SVG or other target format |
| Audit Block | Renderer findings | `FINDING` (tier-tagged) | Late-stage typesetting issues (baseline misalignment, glyph metric surprises); not silently absorbed |

### Shared State

- Read: Theme specification (external configuration)
- Write: None

### Failure Modes

| Condition | Response |
|-----------|----------|
| Unsupported content type in layout | Tier 1 finding; halt or render with placeholder and flag |
| Typesetting issue (baseline misalignment, content closer than expected due to actual glyph metrics) | Tier 2 or 3 finding; does not silently absorb; continues rendering |
| Target format cannot represent a layout element | Tier 1 finding if meaning-affecting; Tier 2 if presentational |
| Math expression cannot be typeset (unknown symbol, parse error) | Tier 1 finding; halt |

### Design Issues Surfaced

**R-1 — Renderer may need to re-query Content Measurement.**
The design doc mentions "small post-typesetting issues (baseline misalignment, content
closer than intended due to actual glyph metrics)." This implies the Renderer may discover
that actual rendered dimensions differ from what CMS reported during layout. If the
discrepancy is large enough to affect layout (not just appearance), this creates a
feedback loop: Renderer → CMS → Layout Engine → Renderer. The design doc treats these
as "late-stage findings" rather than triggers for re-layout. This is a pragmatic choice
but needs to be an explicit policy decision, not an implicit one.

**R-2 — The Renderer is one of multiple parallel output paths.**
The architecture diagram shows Renderer, Accessibility Renderer, and "other output
modalities" as parallel consumers of the Layout Engine output. They presumably all receive
the same `LAYOUT` object. The Renderer emits findings to the Audit block; do the other
parallel paths also emit findings? The Accessibility Renderer almost certainly does
(spec semantic completeness test, D-022 suasive-visual claim requirement). This should
be made explicit rather than inherited by assumption.

---

## Block 6: Audit & Report

**Responsibility recap:** Produce a structured account of the pipeline's behavior for LLM
consumption. The "visual back to language" bridge. The most novel block. Does real
cross-cutting analytic work — not just finding aggregation.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| All blocks | Per-block findings | `FINDING` (tier-tagged) | Via shared finding emission API (Q-013) |
| All blocks | Full block input/output | Block I/O record | Required for cross-cutting emergent finding detection (intent drift, disproportionate elision, constraint conflict resolution interaction); this is a significantly richer input than just findings |
| Audit Block (self, prior run) | Prior audit report | Audit report | For accumulation; unresolved findings persist across turns |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| LLM / Human | Structured audit report | Three-tier structured format | Tier 1 always; Tier 2 during negotiation; Tier 3 on request (D-015) |

### Shared State

- Owns: Cumulative audit report (persists across turns until finding is resolved or dismissed)
- Read: Full pipeline I/O (ephemeral per run)

### Failure Modes

The Audit block does not halt the pipeline — it is a reporter, not a gatekeeper. However:

| Condition | Response |
|-----------|----------|
| Missing input from an upstream block | Flag as a meta-finding: "Block X did not report; audit incomplete" |
| Cross-cutting emergent finding detected | Synthesize as Tier 1 finding with cross-references to contributing block findings |
| Prior finding was not resolved but spec has changed in a way that makes it stale | Re-evaluate and update or dismiss |

### Design Issues Surfaced

**AR-1 — "Full block I/O" is a substantial interface requirement.**
The design doc states the Audit block receives "the full input/output of each block, not
just findings." This is not a finding stream — it is every block's complete input and
output for the current run. This means either:
- (a) Each block serializes and emits its full I/O to the Audit block as part of normal
  processing. Adds overhead to every block; Audit block becomes a data warehouse.
- (b) A shared observability layer captures block I/O without blocks explicitly emitting
  it (e.g., an interceptor on block calls). Cleaner for blocks; adds infrastructure.
- (c) The Audit block can query blocks post-hoc for their I/O. Requires blocks to retain
  their I/O until audited; async model.

Option (b) is architecturally cleanest (blocks don't need to know they're being audited)
but requires infrastructure the design hasn't specified. **This is a significant interface
design decision that affects every block.**

**AR-2 — The Audit block does "real analytic work," not aggregation.**
The design doc explicitly states the Audit block "must do real analytic work to detect
[cross-cutting findings], not just aggregate per-block findings." This means the Audit
block contains non-trivial logic: comparing Genre Resolver's interpretation against
Intent Capture's normalized spec against the rendered output. This is closer to a
reconciliation engine than a log aggregator. Its computational requirements should not
be underestimated.

**AR-3 — Finding accumulation across turns needs a dismissal model.**
Unresolved findings persist across turns (D-015: "Cumulative"). But some findings become
stale (the spec changed in a way that addresses the concern without explicitly resolving
it). The Audit block needs a model for: (a) which findings auto-resolve when specific
spec changes are made, (b) explicit dismissal by LLM or human, (c) findings that can
never auto-resolve (Tier 1 meaning-affecting findings probably shouldn't silently drop).

---

## Auxiliary: Variation Generator

**Responsibility recap:** Given the current spec and a variation scope, generate a small
set of alternative specs differing in controlled ways. Supports variation exploration mode.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| Spec Manager | Current spec | `SPEC` (full, read-only) | Starting point for variation generation |
| LLM / Human | Variation request | Scope + breadth parameter + axis selection | What to vary and how far |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| (Downstream pipeline, ×N) | Variant specs | N × `SPEC` | Each variant is a complete spec; variations run through the full pipeline independently |
| LLM / Human | Structured variation descriptions | Structured natural language | For each variant: how it differs from the center, on which axes, by how much |
| Spec Manager (on selection) | Selected variant as diff | `DIFF` | User selects a variation; its delta from the center spec is submitted as a diff via Spec Manager |

### Shared State

- Read: Current spec (via Spec Manager)
- Write: None directly; writes via Spec Manager on selection

### Failure Modes

| Condition | Response |
|-----------|----------|
| Variation space too small for requested breadth | Report available range; offer narrower variations |
| Variant spec is invalid (variation produces a schema violation) | Skip that variant; report which axis produced the invalid variant |
| Genre-level variation produces specs with genre conflicts | Surface genre conflict as part of the variation description; do not suppress |

### Design Issues Surfaced

**VG-1 — Tap point is ambiguous.**
The design doc says the Variation Generator sits "between Spec Manager and downstream."
This is underspecified. Three possible tap points:
- (a) After Spec Manager, before Genre Resolver — variations are on the spec's semantic
  content; each variant runs through the full pipeline including genre classification.
  Best for structural and genre variations.
- (b) After Genre Resolver, before Layout Engine — genre is fixed; variations are on
  layout and encoding. Best for aesthetic variations within a committed genre.
- (c) Configurable by variation axis — structural/genre variations tap at (a); aesthetic
  variations tap at (b).

Option (c) is architecturally correct but requires the pipeline to support multiple
entry points, which is a non-trivial infrastructure requirement. **This should be an
explicit design decision, not an inherited one.**

**VG-2 — N independent pipeline runs are expensive.**
Each variant spec runs through Genre Resolver, Layout Engine, and Renderer independently.
For N=5 variants, this is 5x the pipeline cost of a single render. Caching (Genre
Resolver result reuse when genre is fixed; Layout Engine reuse for encoding-only variations)
can reduce this, but requires the pipeline to know which blocks need to re-run for a
given variation axis. This is an optimization concern but affects the architecture if
the answer requires blocks to expose "re-run trigger" metadata.

---

## Auxiliary: Content Measurement Service

**Responsibility recap:** Know how to size labels, math expressions, and other content
with intrinsic dimensions. Shared utility consulted by Layout Engine and Renderer.
Not on the main pipeline.

### Receives

| From | What | Form | Notes |
|------|------|------|-------|
| Layout Engine | Measurement request | Content node + style context | Style context needed because font size, weight, etc. affect dimensions |
| Renderer | Measurement request | Content node + style context + precision level | Renderer may need more precise metrics than Layout Engine (exact baseline, not just bounding box) |

### Emits

| To | What | Form | Notes |
|----|------|------|-------|
| Layout Engine | Dimensions | `MEASURE` | Bounding box (unit-invariant); baseline position; ascent/descent if requested |
| Renderer | Full typographic metrics | `MEASURE` | Bounding box + baseline + any additional metrics needed for precise rendering |
| (Internal) | Laid-out math structure | Internal | For complex math, CMS may need to lay out the expression internally to measure it; this structure is available for Renderer reuse (see CMS-1) |

### Shared State

- Owns: Measurement cache (content hash + style context → measurement result)
- Accesses: Font/typesetting subsystem (external dependency)

### Failure Modes

| Condition | Response |
|-----------|----------|
| Unknown content type | Error with content type; caller decides how to handle |
| Font unavailable | Error + fallback dimensions with explicit "fallback" flag; caller decides |
| Math expression unparseable | Error with parse failure detail; propagates to Renderer as Tier 1 finding |
| Measurement requires iterative layout (complex math) | Handled internally; callers see a single synchronous response |

### Design Issues Surfaced

**CMS-1 — Math layout is computed twice if CMS and Renderer are decoupled.**
For complex math expressions, CMS internally lays out the expression to compute its
bounding box. The Renderer needs to lay out the same expression to render it. If these
are separate computations, work is duplicated. Options:
- (a) CMS caches and exposes the laid-out math structure; Renderer retrieves it from CMS
  rather than re-computing. CMS becomes a math layout service, not just a measurement
  service. Scope creep risk.
- (b) Accept the duplication; math layout is fast compared to the rest of the pipeline.
  Pragmatic but wasteful.
- (c) Renderer is responsible for math layout; CMS calls Renderer internally for
  measurements. Creates the exact cycle the CMS was designed to break.

Option (a) is architecturally clean if the scope is accepted explicitly. The CMS
description in the design doc ("know how to size labels, math expressions, and any other
content with intrinsic dimensions") is consistent with option (a) if "know how to size"
is interpreted as "lay out internally and report dimensions."

**CMS-2 — Style context scope.**
CMS needs style context (font, size, weight) to return correct measurements. But the
LLM never touches the theme layer (D-018). This means style context for CMS comes from
the theme configuration, not the spec. The Layout Engine must be given style context
when it calls CMS — but the Layout Engine is supposed to be theme-free (it produces
unit-invariant layout). This is a subtle coupling: layout correctness depends on style
parameters even though style is separated from layout. The resolution is probably to
pass a "measurement context" (font metrics only, not full style) to CMS separately from
the theme; but this needs an explicit design decision.

---

## Cross-Cutting Issues

Issues that don't belong to a single block:

**CC-1 — Two-tier write model in the Spec Manager (SM-2 restated as cross-cutting).**
The Spec Manager currently must accept writes from: Intent Capture (LLM-driven diffs),
Genre Resolver (system-generated genre commitment), Variation Generator (selected variant
diffs), and Human (direct edits). These have different provenance, different validation
requirements, and possibly different access controls. A uniform diff model with
provenance tagging is cleaner than separate write pathways. Recommendation: all writes
go through the same diff pathway, tagged with source type `{llm, system, human, variation}`.
The Spec Manager validates and records provenance for all equally.

**CC-2 — The finding emission API is shared infrastructure, not a block.**
The design doc notes: "a shared finding-emission API that enforces structure and discourages
misuse." This is a cross-cutting infrastructure component that every block uses but no
block owns. It needs a home. Options: owned by the Audit block (natural, since Audit
consumes findings); owned by a separate infrastructure layer (cleaner separation); injected
into each block at construction. This is an architectural decision that precedes Q-013.

**CC-3 — The LLM appears at both ends of the pipeline but has different roles.**
At the input end (Intent Capture), the LLM is a request source whose output is normalized.
At the output end (Audit & Report), the LLM is a consumer of structured findings. These
are the same LLM in the conversational use case, but the architecture should treat them
as two distinct roles with distinct interfaces. Conflating them leads to designs that
work for the conversational case but fail when the input LLM and output LLM are different
systems (e.g., an automated pipeline where a different model drives input than monitors output).

**CC-4 — Environmental parameters need a home.**
Target dimensions and presentation context (Layout Engine, LE-1) are not semantic content
and don't belong in the spec. But they affect reproducibility and audit. They need a
named home — a "render context" or "environment" object that flows alongside the spec,
is recorded in the audit, but is not part of the spec's semantic content.

---

## Summary: Design Questions Requiring Resolution Before Q-002/Q-003

The following issues surfaced by this interface analysis must be resolved before diff
vocabulary and spec representation work can proceed. Each is a genuine fork in the
architecture:

1. **SM-2 / CC-1 — How do system blocks write to the spec?** (Genre Resolver, Variation
   Generator selection.) Recommendation: uniform diff model with provenance tagging.

2. **GR-1 — Does the Policy object go in the spec or in a separate channel?**
   Recommendation: separate policy channel; policy is derived from genre commitment and
   is a Level 3 concern that shouldn't live in the semantic spec.

3. **AR-1 — How does the Audit block receive full block I/O?** Recommendation: shared
   observability layer (interceptor model), not explicit emission from each block.

4. **VG-1 — Where does the Variation Generator tap into the pipeline?**
   Recommendation: configurable by variation axis (structural/genre at post-Spec-Manager;
   aesthetic at post-Genre-Resolver).

5. **CC-4 — Where do environmental parameters (target dimensions, etc.) live?**
   Recommendation: a named "render context" object, separate from the spec, recorded in
   the audit report.

---

*Q-001 draft. For review and revision. Open questions discovered here are tracked above
as sub-items; the load-bearing ones (items 1–5 in the summary) should be resolved before
the Q-002/Q-003 sessions.*
