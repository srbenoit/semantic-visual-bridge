# Decision Log — Semantic-Visual Bridge

**Purpose.** Tracks what has been decided, what alternatives were considered, why, and what each decision
blocks or depends on. Read before each session to avoid re-litigating closed questions. Update at the
end of every session.

**Statuses.**
- `Closed` — committed; requires strong architectural reason to reopen.
- `Revisable` — committed for now; expected to be refined as downstream decisions are made.
- `Superseded` — replaced by a later decision; kept for provenance.

---

## D-001 — Central Architectural Commitment

**Status:** Closed  
**Source:** Preamble  
**Decided:** The LLM expresses semantic intent; a deterministic system resolves intent into geometry.
Neither side does the other's job. The two sides communicate through a structured, diff-friendly working
document and an LLM-readable audit report.  
**Rationale:** LLMs are unreliable at geometric reasoning. Existing tools (Mermaid, Graphviz, TikZ)
require the LLM to write at the constrained-layout level (Level 3) while it reasons at the relational
level (Level 1). The gap produces unreliable, lossy output that is difficult to refine.  
**Alternatives considered:** Not stated; treated as the foundational commitment from which all other
decisions derive.  
**Depends on:** Nothing.  
**Blocks:** Everything. All subsequent decisions must be consistent with this split.

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
**Rationale:** Provides a precise vocabulary for locating where existing tools operate and diagnosing
why they fail when driven by LLM output.  
**Depends on:** D-001.  
**Blocks:** D-003 (block decomposition), genre taxonomy work.

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
**Rationale:** Clean separation of concerns matching the levels in D-002. Each block has a single
well-defined responsibility.  
**Depends on:** D-001, D-002.  
**Blocks:** All block-specific decisions; interface specifications (Q-001).

---

## D-004 — Spec as Source of Truth

**Status:** Closed  
**Source:** Cross-Cutting Principles  
**Decided:** The working spec — not conversation history, not rendered output — is the canonical state.
Every block reads from the spec or from upstream output derived from it.  
**Rationale:** Enables reproducibility, diff-and-refine workflow, and LLM-readable audit without
depending on stateful conversation context.  
**Depends on:** D-003.  
**Blocks:** Spec representation (Q-003), diff vocabulary (Q-002), negotiation protocol (Q-004).

---

## D-005 — No Silent Inference

**Status:** Closed  
**Source:** Cross-Cutting Principles  
**Decided:** When ambiguity arises, blocks surface it explicitly rather than picking an interpretation
invisibly. Ambiguity is information, not a problem to hide.  
**Rationale:** Silent inference produces outputs that look correct but aren't. Errors surface late,
are hard to diagnose, and erode trust in the system.  
**Depends on:** D-001.  
**Blocks:** Negotiation protocol (Q-004); Intent Capture behavior; audit report format (Q-006).
**Note:** Creates tension with usability — not every ambiguity warrants a clarification round.
Resolution deferred to Q-004 and Q-014.

---

## D-006 — Failure Modes are First-Class Outputs

**Status:** Closed  
**Source:** Cross-Cutting Principles  
**Decided:** A system that produces a bad visual silently is worse than one that produces no visual
and a clear diagnosis. The pipeline can halt and request clarification at any block.  
**Rationale:** Silent failures compound. Explicit failures are actionable. Reporting a failure with
a precise diagnosis is more useful than a degraded output.  
**Depends on:** D-005.  
**Blocks:** Audit report format (Q-006); per-block error handling; finding emission API (Q-013).

---

## D-007 — Genre Organized by Cognitive Function, Not Form

**Status:** Closed  
**Source:** Block 3: Genre Resolver  
**Decided:** Genre taxonomy is organized by what cognitive work a visual does, not what it looks like.
Two visuals can look similar but be different genres because they do different cognitive work.  
**Rationale:** LLMs reason about purposes more reliably than about geometric appearances. Function-based
organization makes the vocabulary more reliable as an LLM input.  
**Depends on:** D-001, D-002.  
**Blocks:** Genre taxonomy finalization (Q-005); genre resolution algorithm.

---

## D-008 — Working Genre Taxonomy (17 genres, revisable)

**Status:** Revisable  
**Source:** Block 3: Genre Resolver  
**Decided:** 17 genres organized under five cognitive-function headings:
- *Showing structure:* hierarchy, set-relation, composition
- *Showing process:* flow, state-machine, timeline, transformation
- *Showing relation:* causal-graph, network, comparison-matrix
- *Showing quantity:* function-plot, distribution, categorical-chart
- *Showing geometry:* geometric-construction, schematic, annotated-image

Acknowledged in the design doc as a working set requiring refinement.  
**Rationale:** Derived from common explanatory/analytical visual types. 17 is a working number,
not a principled count. The five-heading structure follows from D-007.  
**Depends on:** D-007.  
**Blocks:** Layout algorithm selection per genre (Q-008); audit check specification per genre (Q-009);
genre taxonomy finalization (Q-005).  
**Known gap:** Genre overlap not addressed. E.g., causal-graph spans "showing structure" and
"showing relation"; timeline spans "showing process" and "showing quantity." Resolution approach
is open.

---

## D-009 — Small Multiples as Modifier, Not Genre

**Status:** Closed  
**Source:** Block 3: Genre Resolver  
**Decided:** Small multiples is a compositional modifier applicable to several genres, not its own genre.
A small-multiple-of-function-plots and a small-multiple-of-state-machines share the repetition structure
but have different layout and encoding conventions. The genre is the repeated unit; the repetition
pattern is the modifier.  
**Rationale:** Treating small multiples as a genre would require duplicating layout/encoding logic
across every combination. Modifier treatment is compositionally cleaner.  
**Depends on:** D-008.  
**Blocks:** Spec representation of small multiples (Q-007); coordinated scale/encoding enforcement
across instances.

---

## D-010 — v1 Composition Model: Primary with Insets

**Status:** Closed  
**Source:** Block 3: Genre Resolver  
**Decided:** v1 supports one primary genre with other genres appearing as embedded sub-visuals (insets).
Full confections — genuine multi-genre composition where no single genre dominates — are deferred to
v2+.  
**Rationale:** Confections require a richer compositional model the v1 architecture cannot adequately
support. The primary-with-insets model covers most practical explanatory visuals while remaining
implementable.  
**Depends on:** D-008.  
**Blocks:** Nothing in v1. Deferred list item for v2.  
**Note:** This is an acknowledged limitation. Confections are where v1 will be most visibly
inadequate relative to a skilled human illustrator.

---

## D-011 — Closed Genre Vocabulary in v1

**Status:** Closed  
**Source:** Block 3: Genre Resolver  
**Decided:** v1 has a closed genre vocabulary. Unclassified or low-confidence requests are tracked
as a corpus to inform later vocabulary additions. Unrecognized genre names are mapped to the nearest
known genre with the mapping reported, or classification refuses and requests clarification.  
**Rationale:** An extensibility mechanism adds complexity. Better to close the vocabulary for v1
and learn what's missing from real usage.  
**Depends on:** D-008.  
**Blocks:** v2 extensibility mechanism (deferred).

---

## D-012 — Layout Engine: Modular per Genre, Not Universal Solver

**Status:** Closed  
**Source:** Block 4: Layout Engine  
**Decided:** Each genre has dedicated layout machinery. No attempt at a universal layout solver.
Genre-to-algorithm binding lives in the Genre Resolver's policy output and is replaceable.  
**Cited candidates:** Sugiyama for layered flows; Reingold-Tilford for trees; force-directed for
networks. Others unspecified.  
**Rationale:** A universal solver would need to accommodate radically different layout semantics
across genres. Dedicated algorithms produce better results and are more maintainable.  
**Depends on:** D-008, D-003.  
**Blocks:** Layout algorithm selection per genre (Q-008).

---

## D-013 — Layout Stability Principle

**Status:** Closed (principle); open (mechanism)  
**Source:** Block 4: Layout Engine  
**Decided:** After a spec diff, the Layout Engine produces a layout that minimizes disruption to
the prior layout while still respecting all current constraints. The engine remembers the prior
solution as an input to the next solution.  
**Rationale:** Global layout algorithms can produce wildly different layouts from small spec changes,
breaking the diff-and-refine model and disorienting users.  
**Depends on:** D-012.  
**Blocks:** Layout stability mechanism (Q-009).  
**Note:** The mental-map preservation problem has no satisfying general solution. Tractable for
constrained-layout genres (geometric construction, schematic); hard for force-directed and
hierarchical layouts. The mechanism will likely be per-algorithm rather than universal.

---

## D-014 — Content Measurement Service as Cycle-Breaker

**Status:** Closed  
**Source:** Block 4: Layout Engine; Auxiliary  
**Decided:** A shared Content Measurement Service (not on the main pipeline) is consulted by
both Layout Engine and Renderer to size labels and math expressions. This breaks the
layout→typesetting→layout dependency cycle.  
**Rationale:** Layout needs label sizes; label sizes (especially math) require typesetting;
typesetting lives in the Renderer. Extracting measurement as a shared service breaks the cycle
without violating block responsibilities.  
**Depends on:** D-003, D-017.  
**Blocks:** Content Measurement Service interface (Q-010).  
**Note:** For complex math expressions, iterative measure cycles may still be required internally.
The service interface must support this without exposing the iteration to callers.

---

## D-015 — Audit Report: Three-Tier Finding Classification

**Status:** Closed  
**Source:** Block 6: Audit & Report  
**Decided:**
- Tier 1: Findings that affect meaning. LLM reads always.
- Tier 2: Decisions that could reasonably be revised. LLM reads during negotiation.
- Tier 3: Routine processing record. LLM reads only for deeper investigation.

Each block tags findings with a tier; the Audit block may re-tier based on cross-cutting concerns.  
**Rationale:** Manages LLM attention. The LLM shouldn't process a dense log to find what matters.
Three tiers match the three reasons to consult the report: "did something go wrong?", "should I
refine this?", "what exactly happened?"  
**Depends on:** D-003, D-006.  
**Blocks:** Audit report format (Q-006); finding emission API (Q-013).

---

## D-016 — Required Spec Properties (Eight)

**Status:** Closed  
**Source:** Block 2: Spec Manager  
**Decided:** The spec must exhibit:
1. *Diffability* — semantic diff, not text diff.
2. *Addressability* — stable identifiers for every spec element, across revisions.
3. *Locality of change* — changes to one part do not require restating unrelated parts.
4. *Self-description* — carries enough metadata to be understandable on its own.
5. *Auditability* — current state reconstructible from revision history.
6. *Renderability gradient* — distinguishes at least: syntactically valid, semantically complete,
   finalized.
7. *Functional role typing* — spec elements carry functional roles: primary content, reference
   structure, annotation, grouping. Not a one-dimensional emphasis scale.
8. *Optional rhetorical purpose* — spec can record an explicit claim when one is present,
   distinguishing descriptive from suasive visuals.

**Rationale:** Each property derived from a specific architectural requirement. Functional role
typing from Tufte's layering analysis. Rhetorical purpose from Tufte's treatment of visuals as
arguments.  
**Depends on:** D-004, D-005.  
**Blocks:** Spec representation (Q-003) must satisfy all eight; diff vocabulary (Q-002) must
support diffability and addressability.

---

## D-017 — Math Content as First-Class

**Status:** Closed  
**Source:** Block 5: Renderer; Principles Worth Harvesting  
**Decided:** Math expressions are structured labeled nodes (MathML or LaTeX-bearing), not strings
or images.  
**Rationale:** Strings lose structure needed for layout and accessibility. Images lose accessibility,
localization, and the ability to participate in layout measurement.  
**Depends on:** D-016 (self-description property implies structured content).  
**Blocks:** Content Measurement Service interface (Q-010); spec schema for labeled content.

---

## D-018 — Theme Separated from Structure

**Status:** Closed  
**Source:** Block 5: Renderer; Principles Worth Harvesting  
**Decided:** Same resolved layout can be rendered at different targets and in different visual styles
without re-layout. The LLM never touches the theme layer.  
**Rationale:** Prevents style changes from triggering re-layout. Allows rendering for multiple
targets (SVG, accessibility text, print) from a single resolved spec.  
**Depends on:** D-003.  
**Blocks:** Renderer architecture; theme specification format.

---

## D-019 — Unit Invariance

**Status:** Closed  
**Source:** Cross-Cutting Principles; Principles Worth Harvesting  
**Decided:** Internal representations are unit-invariant. Concrete units (pixels, points, em)
applied only at presentation time.  
**Rationale:** Decouples layout from output medium. Allows re-rendering at different sizes, on
different devices, without re-layout.  
**Depends on:** D-018.  
**Blocks:** Layout Engine output representation; Content Measurement Service interface.

---

## D-020 — Scope Boundary

**Status:** Closed  
**Source:** Scope section  
**Decided:**
- *In scope:* static, single-visual, explanatory and analytical graphics with high-determinacy
  meaning. Every element has specifiable meaning. Interactive negotiation over the spec is supported;
  the output is static.
- *Definitively out of scope:* generative imagery, aesthetic/expressive visuals, full page/document
  design, photorealistic/mimetic representation, UI/interaction design, cartographic maps.
- *Deferred (adjacent, plausible extension, not categorically excluded):* animation, interactivity,
  pedagogical sequences, real-time/streaming data.

**Rationale:** Scope derived from the central commitment (D-001). Each exclusion corresponds to a
place where the semantics-to-geometry architecture would fail. Naming the scope by derivation rather
than declaration makes the exclusions principled and the extension criteria clear.  
**Depends on:** D-001.  
**Blocks:** Nothing — scope decisions close off design space.

---

## D-021 — Human Involvement Gradient

**Status:** Closed  
**Source:** Human Involvement Modes  
**Decided:** Three modes supported simultaneously via the same working spec:
1. Watching the LLM negotiate (passive, possibly with veto).
2. Selecting among variations via Variation Generator.
3. Editing the spec directly.

This is a structural property of the architecture (single working spec as source of truth), not a
separate feature.  
**Depends on:** D-004, D-003.  
**Blocks:** Spec editing interface; Variation Generator spec.

---

## D-022 — Accessibility as Semantic Completeness Test

**Status:** Closed  
**Source:** Accessibility as Semantic Completeness Test  
**Decided:** If the spec doesn't carry enough information to generate a meaningful accessibility
description, it doesn't carry enough information. For suasive visuals (per D-016), accessibility
text must articulate the claim, not just describe the structure. Failure to articulate a recorded
claim in a suasive visual is a Tier 1 audit finding.  
**Test:** Generate accessibility text from the spec; show it to a different LLM or human who
hasn't seen the visual; compare their predicted visual to the actual visual. Match = spec is
semantically adequate; divergence = information is embedded in layout/rendering decisions that
should have been in the spec.  
**Depends on:** D-016, D-015.  
**Blocks:** Accessibility Renderer specification.

---

## D-023 — Variation Generator: Structured Variations, Not Random

**Status:** Closed (principle); open (specification)  
**Source:** Auxiliary: Variation Generator  
**Decided:** Variations are diffs over the spec drawn from a defined vocabulary of axes (layout,
encoding, genre, structural). Breadth is controllable by parameter. Each variation produces its
own rendering plus a structured account of how it differs from the center spec. Selection commits
a variation's diff cleanly — no partial selection or mixing.  
**Rationale:** Solves aesthetic and "I'll know it when I see it" judgment calls that verbal
negotiation cannot handle. Keeps the LLM in linguistic territory by providing structured difference
descriptions, not just pixel comparisons.  
**Precedent:** Kai Krause Power Tools (mid-1990s) for breadth-controlled variant exploration.
Interactive evolutionary computation in HCI for the selection-as-convergence model.  
**Depends on:** D-004 (variations are spec diffs), D-003.  
**Blocks:** Variation axis vocabulary (Q-011); breadth control parameter specification.

---

## D-024 — Genre is a Hypothesis, Not a Fact

**Status:** Closed  
**Source:** Block 3: Genre Resolver  
**Decided:** Genre classification is revisable by LLM, human, or downstream evidence at any point.
Confidence is reportable. High and low confidence are distinguishable downstream. Low confidence
may trigger clarification. Genre commitment is recorded in the spec; downstream blocks read it
from the spec.  
**Rationale:** Genre drives the entire downstream pipeline. Committing to a wrong genre silently
is worse than surfacing uncertainty.  
**Depends on:** D-007, D-005.  
**Blocks:** Negotiation protocol (Q-004) — specifically how genre revision is handled; diff
vocabulary (Q-002) — specifically the frame-shifting vs. local change distinction (Q-012).

---

## D-025 — Greenfield Implementation; Principles Harvested from Prior Work

**Status:** Closed  
**Source:** Principles Worth Harvesting  
**Decided:** Principles imported from MathOps and chart-theme prior work: unit invariance, math as
first-class, style/structure separation, explicit geometry only where semantically meaningful.
Implementation is greenfield — not constrained by prior schemas.  
**Rationale:** Prior work solved related problems but not this one. Carrying forward implementation
would add constraints without adding value.  
**Depends on:** D-001.  
**Blocks:** Nothing directly; informs all implementation choices.

---

---

## D-026 — Spec / State Architectural Separation

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The working spec and the runtime state are formally distinct objects.
The spec is semantic, versioned, authored, and exportable. State is the runtime
envelope containing the spec plus all derived artifacts (resolved layouts, full audit
record, pipeline execution data). Blocks write to state; only confirmed DIFFs write
to the spec. This is the membrane between authored content and derived content.  
**Rationale:** Conflating spec and state was identified as a load-bearing architectural
error during block interface analysis. The spec/state split makes the versioning model,
rollback semantics, and export/embed model coherent.  
**Depends on:** D-004.  
**Blocks:** All slot-level decisions (D-027 through D-032); Q-002, Q-003.

---

## D-027 — Six-Slot Spec Structure

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The spec has six slots with distinct write permissions, re-evaluation
triggers, and semantic scope:
1. Semantic Intent (Data Section + Intent Section) — LLM/human only
2. Genre Commitment — GR proposes, LLM/human confirms
3. Policy (including Autonomy Charter) — GR proposes, LE may propose, LLM/human may override
4. Presentation Preferences & Policy Overrides — LLM/human only
5. Render Context Set — LLM/human only
6. Audit Summary — Audit block only, system DIFF, terminal

**Rationale:** Each slot has a distinct author and a distinct downstream role.
Separating them prevents scope creep, makes write permissions enforceable, and
gives the version history semantic meaning (which slot changed → what re-runs).  
**Depends on:** D-026.  
**Blocks:** Q-002 (diff vocabulary must address all six slots), Q-003 (spec
representation must accommodate all six slots with their distinct properties).

---

## D-028 — Genre Resolver Writes via Confirmed DIFF (SM-2 Resolution)

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The Genre Resolver does not write directly to the spec. When it wants
to commit a genre classification or a policy derivation, it proposes a DIFF through
the normal diff pathway. Genre commitment (Slot 2) requires LLM/human confirmation.
Policy derivation (Slot 3) is system-confirmed (no human confirmation required) but
is recorded with Genre Resolver as proposer for provenance.  
**Rationale:** Maintaining a uniform DIFF model with provenance tagging is cleaner
than creating a separate write pathway for system blocks. All spec history is
consistent and auditable.  
**Depends on:** D-026, D-027.  
**Blocks:** Q-002 (diff vocabulary must include source-type tagging).

---

## D-029 — Audit Summary Belongs in the Spec (Slot 6)

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The audit summary is Slot 6 of the spec, not a separate state artifact.
It represents the result of the full pipeline loop — what decisions were made, what
was found, what the LLM needs to know. This is meaningful content, not ephemeral
computation. The full three-tier finding record remains in State; Slot 6 is a curated
summary derived from it. Slot 6 DIFFs are system-written and terminal — they do not
trigger further pipeline re-runs.  
**Rationale:** Placing the audit summary in the spec ensures it is versioned alongside
the semantic content that produced it, exportable with the visual, and readable by a
future LLM session without requiring access to runtime state.  
**Depends on:** D-026, D-027, D-015.  
**Blocks:** Audit block interface; Q-006.

---

## D-030 — Autonomy Charter in Slot 3 Policy

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** Each genre's policy (Slot 3) includes an Autonomy Charter — a table
specifying what the Layout Engine and Renderer may do autonomously vs. what requires
approval. Four categories: Autonomous (no reporting), Auto-DIFF (system commits,
Tier 3 finding), Proposed-DIFF (LLM/human confirms, Tier 2 finding),
Never-autonomous (Tier 1 finding if situation arises, explicit request required).
The genre definition provides the default charter. LLM/human can tighten but not
loosen the Never-autonomous category.  
**Rationale:** Without explicit autonomy governance, the pipeline either asks too
many questions (breaking usability) or makes changes silently (violating D-005).
The Autonomy Charter makes the boundary explicit, genre-appropriate, and auditable.
It also connects the autonomy level directly to the finding tier structure (D-015).  
**Depends on:** D-005, D-015, D-027.  
**Blocks:** Per-genre definition work; Q-008 (layout algorithms must declare their
autonomy behavior).

---

## D-031 — v1 Versioning: Linear History with Rollback; Branching Deferred

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** v1 uses a linear DIFF chain with a current-version pointer. Rollback
moves the pointer to any prior version. A rollback followed by a new DIFF orphans
the previously-current versions (they remain in the chain but are no longer on the
path to current). Branching — maintaining multiple live chains simultaneously — is
deferred to v2. The Variation Generator covers the primary "explore alternatives"
use case without requiring true branching.  
**Rationale:** Branching adds significant State complexity (which branch is active?
when are branches evicted?) for limited v1 benefit. Linear rollback is sufficient
for the "things went wrong, go back to a known-good state" use case.  
**Depends on:** D-026.  
**Blocks:** State structure (no branch management needed in v1 State).

---

## D-032 — Slot 1 Data Section: Structured Data Separate from Semantic Intent

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** Slot 1 is divided into a Data Section (raw structured data: tables,
time series, hierarchical trees) and an Intent Section (entity/relation set, analytical
directives, rhetorical purpose, compositional structure). Data is raw material;
intent is what the visual should express about that data. Entities in the Intent
Section reference datasets by ID. For v1, data is static (uploaded/provided by
user or LLM); analytical computation by the pipeline (fitting regressions, generating
forecasts) is deferred to v2.  
**Rationale:** Conflating bulk structured data with semantic intent makes the
entity/relation set unwieldy and obscures the actual analytical request. Separating
them also makes versioning cleaner: updating a dataset is a different operation from
revising the analytical intent.  
**Depends on:** D-027.  
**Blocks:** Spec schema design (Q-003 must accommodate two-section Slot 1).

---

## D-033 — Slot 3/4 Operational Boundary: Policy vs. Preferences

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The operational definition:
- **Policy (Slot 3)** = what the Genre Resolver proposes. Genre-derived parameters
  with defaults; overridable by LLM/human or LE proposals.
- **Presentation Preferences & Policy Overrides (Slot 4)** = what the LLM/human
  expresses that the Genre Resolver would not propose.
- **Style (theme, outside spec)** = what the Genre Resolver never proposes.
  Colors, fonts, line weights, decorative elements.

This boundary will sharpen as actual genres are defined. The catch-all in Slot 4 has
explicit exclusion rules: cannot contain theme content; cannot contain GR-proposed
parameters (those go in Slot 3); cannot contain semantic intent (that goes in Slot 1).  
**Rationale:** A functional definition of the boundary is more durable than a
content-based definition, since the content varies by genre.  
**Depends on:** D-027, D-018.  
**Blocks:** Per-genre definition (genre definitions will make the boundary concrete).

---

## D-034 — Audit Block Uses Interceptor Model (AR-1 Resolution)

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The Audit block receives full block I/O via a shared observability
interceptor layer, not via explicit emission from each block. Blocks do not need
to know they are being audited. The interceptor captures all block inputs and outputs
and makes them available to the Audit block. This is infrastructure separate from
the finding emission API.  
**Rationale:** Explicit I/O emission from each block would couple every block to
the audit infrastructure and add overhead to all block implementations. An interceptor
keeps blocks clean and allows the Audit block to decide what to retain.  
**Depends on:** D-003, D-015.  
**Blocks:** Q-013 (finding emission API is separate from the interceptor).

---

## D-035 — Variation Generator: Two Tap Points; Genre Variations Out of Scope

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The Variation Generator has two tap points:
1. Pre-Layout Engine (after Genre Resolver): structural variations within the
   committed genre. Variants run through the full remaining pipeline.
2. Post-Layout Engine (before Renderer): aesthetic/rendering variations. Variants
   run through the Renderer only — cheaper and appropriate for fine-tuning appearance.

Genre variations (generating the same content under different genres) are out of scope
for the Variation Generator. Genre exploration belongs in the Genre Resolver /
Intent Capture negotiation, not in post-genre variation.  
**Rationale:** Genre variations require running the full pipeline including Genre
Resolver and produce qualitatively different visuals, not "variations" of a chosen
visual. Keeping VG within the committed genre makes the variation vocabulary coherent
and the selection model tractable.  
**Depends on:** D-023, D-024.  
**Blocks:** Q-011 (variation axis vocabulary must respect these tap points).

---

## D-036 — Content Measurement Service: Async Parallel Calls

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** The Content Measurement Service supports async parallel measurement
requests, analogous to browser layout threads. The Layout Engine can batch measurement
requests and receive results asynchronously rather than blocking on each. For complex
math expressions requiring iterative internal layout, the CMS handles iteration
internally and presents a single async response to callers.  
**Rationale:** Layout algorithms need many measurements; sequential blocking calls
would compound latency unacceptably. Async batching matches established browser
engine practice for the same problem.  
**Depends on:** D-014.  
**Blocks:** Q-010 (CMS interface must specify the async/batch model).

---

## D-037 — Render Context Set Lives in Spec Slot 5 (LE-1/CC-4 Resolution)

**Status:** Closed  
**Source:** Q-001 session  
**Decided:** Named render contexts (screen, print-a4, low-vision, thumbnail, etc.)
with their parameters (target format, dimensions, font environment, background/contrast)
are defined in Spec Slot 5. The spec defines *which* contexts exist and their
parameters. Resolved layouts in State are keyed to (spec-version-id, context-id).
Prior layouts for stability are also keyed this way. Environmental parameters are
not floating loose — they are owned by the spec's render context definitions.  
**Rationale:** Storing render contexts in the spec makes output reproducible
(same spec + same context = same output), versionable (adding a render context
is a Slot 5 DIFF), and exportable (an imported spec includes its render targets).  
**Depends on:** D-026, D-027, D-019.  
**Blocks:** Q-010 (CMS needs font env from render context, not from theme).

---

*Log updated after Q-001 review session. 37 total decisions recorded.*
