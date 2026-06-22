# Decision Log — Semantic-Visual Bridge

**Purpose.** Permanent record of closed architectural decisions. Each entry records what was
decided, why, what it depends on, and what it blocks. Entries are never deleted; superseded
decisions are marked and cross-referenced to their replacement.

**Statuses.**
- `Closed` — committed; requires strong architectural reason to reopen.
- `Revisable` — committed for now; expected to be refined as downstream decisions are made.
- `Superseded` — replaced by a later decision; kept for provenance.

---

## D-001 — Central Architectural Commitment

**Status:** Closed
**Source:** Preamble
**Decided:** The LLM expresses semantic intent; a deterministic system resolves intent into
geometry. Neither side does the other's job. The two sides communicate through a structured,
diff-friendly working document and an LLM-readable audit report.
**Rationale:** LLMs are unreliable at geometric reasoning. Existing tools (Mermaid, Graphviz,
TikZ) require the LLM to write at the constrained-layout level (Level 3) while it reasons at
the relational level (Level 1). The gap produces unreliable, lossy output that is difficult to
refine.
**Depends on:** Nothing.
**Blocks:** Everything.

---

## D-002 — Four-Level Hierarchy of Visual Meaning

**Status:** Closed
**Source:** Preamble
**Decided:** Visual meaning is stratified into four levels:
1. Relational structure — entities and labeled relations; no geometry.
2. Spatial metaphor — a layout genre is selected.
3. Constrained layout — geometric constraints a solver can satisfy.
4. Rendered output — actual pixels.

The LLM operates at Levels 1–2; the deterministic pipeline handles 3–4.
**Rationale:** Provides a precise vocabulary for locating where existing tools operate and
diagnosing why they fail when driven by LLM output.
**Depends on:** D-001.
**Blocks:** D-003, genre taxonomy work.

---

## D-003 — Six-Block Pipeline Architecture

**Status:** Closed
**Source:** Architecture Overview
**Decided:** Six functional blocks in pipeline order:
1. Intent Capture
2. Spec Manager
3. Genre Resolver
4. Layout Engine
5. Renderer (plus parallel: Accessibility Renderer, other output modalities)
6. Audit & Report

Two auxiliaries not on the main pipeline: Variation Generator, Content Measurement Service.
A working spec flows downward; a decision log flows upward; the LLM negotiates through both.
**Rationale:** Clean separation of concerns matching the levels in D-002.
**Depends on:** D-001, D-002.
**Blocks:** All block-specific decisions; Q-001.

---

## D-004 — Spec as Source of Truth

**Status:** Closed
**Source:** Cross-Cutting Principles
**Decided:** The working spec — not conversation history, not rendered output — is the
canonical state. Every block reads from the spec or from upstream output derived from it.
**Rationale:** Enables reproducibility, diff-and-refine workflow, and LLM-readable audit
without depending on stateful conversation context.
**Depends on:** D-003.
**Blocks:** Q-003, Q-002, Q-004.

---

## D-005 — No Silent Inference

**Status:** Closed
**Source:** Cross-Cutting Principles
**Decided:** When ambiguity arises, blocks surface it explicitly rather than picking an
interpretation invisibly. Ambiguity is information, not a problem to hide.
**Rationale:** Silent inference produces outputs that look correct but aren't.
**Depends on:** D-001.
**Blocks:** Q-004, Intent Capture behavior, Q-006.

---

## D-006 — Failure Modes are First-Class Outputs

**Status:** Closed
**Source:** Cross-Cutting Principles
**Decided:** The pipeline can halt and request clarification at any block. A system that
produces a bad visual silently is worse than one that produces no visual and a clear diagnosis.
**Rationale:** Silent failures compound. Explicit failures are actionable.
**Depends on:** D-005.
**Blocks:** Q-006, Q-013.

---

## D-007 — Genre Organized by Cognitive Function, Not Form

**Status:** Closed
**Source:** Block 3: Genre Resolver
**Decided:** Genre taxonomy is organized by what cognitive work a visual does, not what it
looks like. Two visuals can look similar but be different genres because they do different
cognitive work.
**Rationale:** LLMs reason about purposes more reliably than geometric appearances.
**Depends on:** D-001, D-002.
**Blocks:** Q-005, genre resolution algorithm.

---

## D-008 — Genre Resolver as Active Interrogator

**Status:** Closed
**Source:** Block 3: Genre Resolver
**Decided:** The Genre Resolver does not passively accept a genre label. It probes feasibility,
checks for genre–content mismatches, and surfaces issues before committing. It may suggest
alternatives.
**Rationale:** Passive acceptance allows mismatches to propagate silently to the Layout Engine.
**Depends on:** D-005, D-006, D-007.
**Blocks:** Q-004, Q-012.

---

## D-009 — Layout Engine Reads Policy, Not Spec Directly

**Status:** Closed
**Source:** Block 4: Layout Engine
**Decided:** The Layout Engine receives a policy object derived from the genre commitment, not
direct access to the full spec. The policy object contains algorithm binding, parameter
constraints, and encoding conventions.
**Rationale:** Decouples the Layout Engine from spec representation details. Provides a clean
interface boundary.
**Depends on:** D-003.
**Blocks:** Q-008.

---

## D-010 — Confections Deferred to v2

**Status:** Closed
**Source:** Architecture discussion
**Decided:** Genuine multi-genre compositions where no single genre dominates are out of scope
for v1. v1 supports a primary genre with insets.
**Rationale:** The spec representation for confections requires solving cross-genre identity
and shared-frame semantics simultaneously. Too high cost for v1.
**Depends on:** D-007.
**Blocks:** Nothing directly; affects v2 scope.

---

## D-011 — Closed Genre Vocabulary for v1

**Status:** Closed
**Source:** Architecture discussion
**Decided:** The genre vocabulary is closed for v1. No extensibility mechanism in v1.
**Rationale:** An open vocabulary requires a plugin/registry architecture that is out of scope
for v1. The closed vocabulary can be designed to be correct; an open one requires correctness
of the extension mechanism.
**Depends on:** D-007.
**Blocks:** Q-005.

---

## D-012 — Unit-Invariant Layout

**Status:** Closed
**Source:** Principles Worth Harvesting
**Decided:** The Layout Engine produces unit-invariant geometry. Concrete units are resolved
at render time against the Render Context.
**Rationale:** Decouples layout computation from output format. A layout computed in abstract
units renders correctly at any scale or resolution.
**Depends on:** D-009.
**Blocks:** Q-008, Q-009.

---

## D-013 — Math as First-Class Content

**Status:** Closed
**Source:** Principles Worth Harvesting
**Decided:** Mathematical content (equations, expressions, symbolic structures) is a first-class
content type in the spec. It is not reduced to images or strings.
**Rationale:** Math is a primary use case. Reducing it to images loses semantics and prevents
accessibility rendering.
**Depends on:** D-001.
**Blocks:** Q-003 (spec must have a math content type), Q-010.

---

## D-014 — Style / Structure Separation

**Status:** Closed
**Source:** Principles Worth Harvesting
**Decided:** Visual structure (what entities exist, how they relate, what layout genre applies)
is strictly separated from style (colors, fonts, decorative choices). The spec contains
structure; style is applied at render time.
**Rationale:** Structure carries meaning; style is presentation. Conflating them prevents
semantic reasoning about the visual.
**Depends on:** D-001.
**Blocks:** Slot 3/4 boundary (Q-003); D-027.

---

## D-015 — Three-Tier Finding Classification

**Status:** Closed
**Source:** Audit & Report block
**Decided:** Audit findings are classified in three tiers:
- Tier 1: Blocking — pipeline cannot proceed without resolution.
- Tier 2: Significant — pipeline proceeds with a proposed resolution; LLM/human confirmation
  required before commit.
- Tier 3: Advisory — pipeline proceeds; finding logged; no confirmation required.
**Rationale:** Flat finding lists conflate urgency. Tiered classification makes pipeline
behavior predictable and auditable.
**Depends on:** D-006.
**Blocks:** Q-006, Q-013, Autonomy Charter (D-030).

---

## D-016 — Eight Spec Properties (Addressability, etc.)

**Status:** Closed
**Source:** Design doc
**Decided:** The spec must exhibit eight properties: addressability, semantic typing,
composability, diff-friendliness, LLM-readability, human-readability, round-trip fidelity,
and stable identity across versions.
**Rationale:** These properties constrain the representation choice and are collectively
necessary for the system to function correctly.
**Depends on:** D-004.
**Blocks:** Q-003.

---

## D-017 — Negotiation Loop Architecture

**Status:** Closed
**Source:** Design doc
**Decided:** The LLM does not receive the full pipeline output directly. It receives the
Audit Report (a curated, LLM-readable summary) and proposes DIFFs against the spec.
This is the negotiation loop.
**Rationale:** Giving the LLM full pipeline output would include low-level geometry and
render artifacts that are not useful for semantic reasoning.
**Depends on:** D-004, D-006.
**Blocks:** Q-004.

---

## D-018 — Variation Generator as Auxiliary

**Status:** Closed
**Source:** Design doc
**Decided:** The Variation Generator is not on the main pipeline. It taps the pipeline at
defined points to produce alternatives. It does not modify the working spec.
**Rationale:** Variations are explorations, not commits. Keeping the VG off the main pipeline
prevents accidental spec mutation.
**Depends on:** D-003.
**Blocks:** VG tap point decisions (D-034).

---

## D-019 — Render Context as Named Profiles

**Status:** Closed
**Source:** Design doc
**Decided:** Output targets are named profiles (e.g., "screen-HD", "print-A4", "low-vision")
rather than ad-hoc parameter sets. The spec carries the set of target profiles; the Renderer
resolves each.
**Rationale:** Named profiles are composable, reusable, and auditable. Ad-hoc parameters
are neither.
**Depends on:** D-012.
**Blocks:** Slot 5 definition; Q-010.

---

## D-020 — Out-of-Scope for v1

**Status:** Closed
**Source:** Design doc
**Decided:** Explicitly out of scope for v1: animation, interactivity, pedagogical sequences,
real-time/streaming data, cartographic maps, genre vocabulary extensibility.
**Rationale:** Each of these requires spec-level features that would significantly complicate
the core model. They are deferred, not rejected.
**Depends on:** D-001.
**Blocks:** Nothing; constrains scope.

---

## D-021 — Accessibility as Parallel Output, Not Afterthought

**Status:** Closed
**Source:** Design doc
**Decided:** Accessibility rendering (alt text, structured description, screen-reader markup)
is a parallel output produced by the Accessibility Renderer, not post-processing of the visual
output. It reads from the same spec.
**Rationale:** Post-processing accessibility is lossy. Reading from the spec allows accurate
semantic description.
**Depends on:** D-004.
**Blocks:** Accessibility renderer interface.

---

## D-022 — Content Measurement as Shared Utility

**Status:** Closed
**Source:** Design doc
**Decided:** Content measurement (text metrics, math bounding boxes, image dimensions) is a
shared utility service, not a per-block responsibility. Any block that needs measurements calls
the Content Measurement Service.
**Rationale:** Measurement logic is complex, render-context-dependent, and should not be
duplicated across blocks.
**Depends on:** D-003.
**Blocks:** Q-010.

---

## D-023 — Explicit Geometry Only Where Semantically Meaningful

**Status:** Closed
**Source:** Principles Worth Harvesting
**Decided:** The spec contains explicit geometric constraints only when position or dimension
carries semantic meaning (e.g., a floor plan where room adjacency is a semantic claim). In
all other cases, geometry is fully derived by the Layout Engine.
**Rationale:** Putting arbitrary geometry in the spec conflates structure and style and makes
the spec brittle to render context changes.
**Depends on:** D-014.
**Blocks:** Q-003 (spatial constraint representation).

---

## D-024 — Frame-Shifting Changes Require Explicit Flagging

**Status:** Closed
**Source:** Design doc
**Decided:** A change that requires switching the genre (e.g., from hierarchy to network)
is a frame-shifting change and requires explicit confirmation, not just a DIFF apply.
Non-frame-shifting changes can be applied with the normal DIFF confirmation flow.
**Rationale:** Genre changes invalidate all downstream policy and layout work. Treating them
as ordinary DIFFs would silently discard significant computation.
**Depends on:** D-008, D-017.
**Blocks:** Q-002, Q-012.

---

## D-025 — Greenfield Implementation; Principles Harvested from Prior Work

**Status:** Closed
**Source:** Principles Worth Harvesting
**Decided:** Implementation is greenfield. Principles imported from prior work: unit invariance,
math as first-class, style/structure separation, explicit geometry only where semantically
meaningful. Prior schemas are not carried forward.
**Rationale:** Prior work solved related problems but not this one.
**Depends on:** D-001.
**Blocks:** Nothing directly; informs all implementation choices.

---

## D-026 — Spec / State Architectural Separation

**Status:** Closed
**Source:** Q-001 session
**Decided:** The working spec and the runtime state are formally distinct objects. The spec
is semantic, versioned, authored, and exportable. State is the runtime envelope containing
the spec plus all derived artifacts (resolved layouts, full audit record, pipeline execution
data). Blocks write to state; only confirmed DIFFs write to the spec.
**Rationale:** Conflating spec and state was identified as a load-bearing architectural error
during block interface analysis.
**Depends on:** D-004.
**Blocks:** D-027 through D-032; Q-002, Q-003.

---

## D-027 — Six-Slot Spec Structure

**Status:** Closed
**Source:** Q-001 session
**Decided:** The spec has six slots with distinct write permissions, re-evaluation triggers,
and semantic scope:
1. Semantic Intent (Data Section + Intent Section) — LLM/human only
2. Genre Commitment — GR proposes, LLM/human confirms
3. Policy (including Autonomy Charter) — GR proposes, LE may propose, LLM/human may override
4. Presentation Preferences & Policy Overrides — LLM/human only
5. Render Context Set — LLM/human only
6. Audit Summary — Audit block only, system DIFF, terminal
**Rationale:** Each slot has a distinct author and a distinct downstream role. Separating them
prevents scope creep and makes write permissions enforceable.
**Depends on:** D-026.
**Blocks:** Q-002, Q-003.

---

## D-028 — Genre Resolver Writes via Confirmed DIFF

**Status:** Closed
**Source:** Q-001 session
**Decided:** The Genre Resolver does not write directly to the spec. It proposes a DIFF
through the normal diff pathway. Genre commitment (Slot 2) requires LLM/human confirmation.
Policy derivation (Slot 3) is system-confirmed but recorded with Genre Resolver as proposer.
**Rationale:** Maintaining a uniform DIFF model with provenance tagging is cleaner than a
separate write pathway.
**Depends on:** D-026, D-027.
**Blocks:** Q-002 (diff vocabulary must include source-type tagging).

---

## D-029 — Audit Summary Belongs in the Spec (Slot 6)

**Status:** Closed
**Source:** Q-001 session
**Decided:** The audit summary is Slot 6 of the spec. The full three-tier finding record
remains in State; Slot 6 is a curated summary derived from it. Slot 6 DIFFs are
system-written and terminal — they do not trigger further pipeline re-runs.
**Rationale:** Placing the audit summary in the spec ensures it is versioned alongside the
semantic content that produced it and readable by a future LLM session without runtime state.
**Depends on:** D-026, D-027, D-015.
**Blocks:** Audit block interface; Q-006.

---

## D-030 — Autonomy Charter in Slot 3 Policy

**Status:** Closed
**Source:** Q-001 session
**Decided:** Each genre's policy (Slot 3) includes an Autonomy Charter — a table specifying
what the Layout Engine and Renderer may do autonomously vs. what requires approval. Four
categories: Autonomous (no reporting), Auto-DIFF (system commits, Tier 3 finding),
Proposed-DIFF (LLM/human confirms, Tier 2 finding), Never-autonomous (Tier 1 finding if
situation arises). The genre definition provides the default charter. LLM/human can tighten
but not loosen the Never-autonomous category.
**Rationale:** Without explicit autonomy governance, the pipeline either asks too many questions
or makes changes silently. The Autonomy Charter connects autonomy level directly to the finding
tier structure.
**Depends on:** D-005, D-015, D-027.
**Blocks:** Per-genre definition work; Q-008.

---

## D-031 — v1 Versioning: Linear History with Rollback

**Status:** Closed
**Source:** Q-001 session
**Decided:** v1 uses a linear version history. Each confirmed DIFF creates a new version.
Rollback to any prior version is supported. Branching is deferred to v2.
**Rationale:** Branching adds significant State complexity. The Variation Generator handles
the "show alternatives" use case without requiring spec-level branching.
**Depends on:** D-026.
**Blocks:** Nothing directly; constrains State structure.

---

## D-032 — Data Section in Slot 1

**Status:** Closed
**Source:** Q-001 session (feedback)
**Decided:** Slot 1 (Semantic Intent) is subdivided into a Data Section and an Intent Section.
The Data Section holds bulk structured data (tables, series, hierarchical trees) inline or
by reference. The Intent Section holds the semantic description (entity set, relation set,
analytical directives, rhetorical purpose, compositional structure). Data is referenced from
intent by dataset/column identifiers.
**Rationale:** Bulk data is not semantic intent. Separating them keeps the intent readable and
diffable without embedding large data payloads in semantic fields.
**Depends on:** D-027.
**Blocks:** Q-003 (data content types must be specified).

---

## D-033 — Slot 3 / Slot 4 Boundary: Policy vs. Style

**Status:** Closed
**Source:** Q-001 session (feedback)
**Decided:** Slot 3 (Policy) contains what the Genre Resolver proposes. Slot 4 (Presentation
Preferences & Policy Overrides) contains what the Genre Resolver never proposes but LLM/human
may set. This is an operational definition: if the GR would produce it from genre reasoning,
it belongs in Slot 3; if it is purely a human/LLM preference that GR has no opinion on,
it belongs in Slot 4.
**Rationale:** A clean operational definition avoids case-by-case debates about which slot
owns a given parameter.
**Depends on:** D-027.
**Blocks:** Genre definition work; Slot 3 schema.

---

## D-034 — Variation Generator Tap Points

**Status:** Closed
**Source:** Q-001 session
**Decided:** The Variation Generator has two tap points: pre-Layout Engine (structural
variations — different spec/genre combinations) and post-Layout Engine (aesthetic variations —
same structure, different layout/style realizations).
**Rationale:** The two tap points serve different use cases. Pre-LE variations explore
structural alternatives; post-LE variations explore aesthetic alternatives within a fixed
structure.
**Depends on:** D-018.
**Blocks:** VG interface specification.

---

## D-035 — Audit Block Uses Interceptor / Observability Model

**Status:** Closed
**Source:** Q-001 session
**Decided:** The Audit block receives the full input/output of each pipeline block via an
interceptor model, not via explicit pass-through in each block's interface. Blocks are not
responsible for routing their I/O to the Audit block.
**Rationale:** An interceptor model provides full observability without coupling the Audit
block into every block's interface design. Blocks remain unaware of the audit mechanism.
**Depends on:** D-003.
**Blocks:** Audit block implementation.

---

## D-036 — Content Measurement Service is Async with Caching

**Status:** Closed
**Source:** Q-001 session
**Decided:** The Content Measurement Service operates asynchronously. Results are cached
keyed to (content, render-context). Callers do not block the pipeline waiting for
measurements that are likely already cached.
**Rationale:** Measurement (especially math typesetting) is expensive. Caching is essential
for performance. Async operation prevents measurement from becoming a pipeline bottleneck.
**Depends on:** D-022.
**Blocks:** Q-010.

---

## D-037 — Render Context Set Belongs in Slot 5 of Spec

**Status:** Closed
**Source:** Q-001 session
**Decided:** The set of named render contexts (output targets) is Slot 5 of the spec, authored
by LLM/human. The CMS receives font environment from the render context, not from a theme.
**Rationale:** Render contexts are authored intent about where the visual will be used.
They belong in the spec alongside the other authored slots.
**Depends on:** D-026, D-027, D-019.
**Blocks:** Q-010.

---

*Log updated after Q-001 session. 37 total decisions recorded (D-001 through D-037).*
*Next update: after Q-002/Q-003 session.*
