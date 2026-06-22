# Spec and State Structure

**Status:** Current — updated after Q-001 review session  
**Depends on:** D-004 (spec as source of truth), D-005 (no silent inference),
D-015 (three-tier finding classification), D-016 (eight spec properties),
D-018 (theme separated from structure), D-019 (unit invariance),
D-026 through D-037 (decisions from Q-001 session — see Decision Log)

---

## The Core Distinction

**Spec** — semantic, versioned, authored. The record of intent and the decisions made
to realize it. An LLM or human reading the spec should understand what the visual means,
why those choices were made, and what the system concluded about the result. The spec
is the save file. It is branchable, rollback-capable, and exportable alongside or
embedded in the rendered output.

**State** — runtime envelope. Contains the current spec plus everything computed from
it: resolved layouts, the detailed audit record, and pipeline execution data. State is
per-session. Most of it is recomputable from the spec and the render context set.
The full audit record is the exception — it accumulates genuine new information.

The spec lives *inside* state. Blocks write to state; only confirmed DIFFs write to
the spec. This is the membrane between authored content and derived content.

---

## The Spec

### Internal Structure: Six Slots

Each slot has distinct write permissions, re-evaluation triggers, and semantic scope.
Slots 1–5 are authored (directly or via confirmation). Slot 6 is system-written.

---

#### Slot 1 — Semantic Intent

The meaning the LLM or human is trying to express. Pure Level 1 content (D-002):
no geometry, no algorithm hints, no style. Slot 1 is divided into two sections —
Data and Intent — because structured data (time series, tables, hierarchical trees)
is raw material for a visual, not semantic intent itself. "Show the growth rate trends
in the attached data with regression curves" is intent; the 47-row time series is data.
Conflating them would make the entity/relation set unwieldy and obscure the actual
analytical request.

```
Slot 1
  ├── Data Section
  └── Intent Section
```

---

**Data Section**

Raw structured data that the visual will draw on. Not the same as the entities and
relations — it is the material from which the visual's semantic content is derived.

- **Datasets** — named structured data objects. Each dataset has:
  - Stable ID (referenced by entities in the Intent Section)
  - Schema description (column names, types, units)
  - Data content: inline (small datasets) or referenced by attachment ID (large datasets)
  - For v1: data is static. Live/streaming data is deferred to v2.
- **Derived series** — pre-computed analytical results (regression curves, projections,
  moving averages) that the visual should display. For v1, these are provided by the
  user/LLM, not computed by the pipeline. Analytical computation by the pipeline is
  a v2 extension.

**Write permission:** LLM/human DIFF only. Data changes do not automatically trigger
genre re-evaluation unless the structural properties of the data change significantly
(e.g., adding a new time series vs. adding data points to an existing one).

---

**Intent Section**

The semantic structure of what the visual should express, referencing datasets by ID.

- **Entity set** — each entity has: stable ID, label (plain text or math expression),
  functional role (primary / reference / annotation / grouping per D-016), optional
  emphasis weight, optional domain type hint, optional data reference (dataset ID +
  column or key path)
- **Relation set** — each relation has: stable ID, type label, source entity ID,
  target entity ID, directionality, optional cardinality or weight, optional label
- **Analytical directives** — what the visual should communicate about the data,
  beyond structure: e.g., "compare growth rates across these series," "show the
  regression curve for each trend," "highlight where series cross." These are semantic
  intent, not layout instructions. They inform genre classification and audit checks.
- **Rhetorical purpose** — `descriptive` (default) or `suasive` (requires an explicit
  claim string; the claim is what the visual argues, not just describes)
- **Compositional structure** — how sub-visuals are organized:
  - Primary visual (one genre per D-010)
  - Inset sub-visuals (other genres embedded in the primary)
  - Small multiples modifier if applicable (per D-009): instance count, shared
    encoding contract (shared axes, aligned scales — these are semantic constraints
    because they affect data interpretation, not layout preferences)

**Diagnostic for compositional vs. layout boundary:** If changing a property would
change what the visual *means* (shared axis scale, number of panels, which entities
are grouped), it belongs in Slot 1. If it would only change how the visual *looks*
(2×2 vs. 1×4 grid arrangement, edge routing style), it belongs in Slot 4.

**Write permission:** LLM/human DIFF only. No system block proposes changes to
semantic intent. A block that discovers a potential intent problem surfaces it as
a finding; the LLM/human decides whether to change the intent.

**Re-evaluation trigger:** Any change to Slot 1 triggers Genre Resolver if the
change could affect genre classification (entity type changes, significant structural
changes, analytical directive changes). Always triggers Layout Engine and Renderer
downstream.

---

#### Slot 2 — Genre Commitment

The genre classification and the reasoning behind it. Written by the Genre Resolver
but committed only on LLM/human confirmation (per SM-2 decision).

**Contains:**
- **Primary genre** — token from the closed vocabulary (D-011)
- **Small multiples modifier** — present/absent; if present: the repeated genre,
  instance count (if known), coordinated encoding contract (shared axes, aligned scales)
- **Confidence level** — numeric or tiered (high / medium / low)
- **Alternatives considered** — list of other plausible genres with brief reasoning
  for each rejection
- **Feasibility probe summary** — if the Genre Resolver ran a layout feasibility
  probe via the Layout Engine: which genres were probed, what the LE reported for each,
  how this influenced the final selection
- **Genre revision history** — prior genre commitments with the reasoning for each
  change; preserves the trail if genre evolved during negotiation

**Write permission:** Genre Resolver proposes → LLM/human confirms → committed as
provenance-tagged system DIFF. The confirming party is recorded.

**Re-evaluation trigger:** Any change to Slot 2 triggers full downstream re-run:
Policy derivation (Slot 3 update), Layout Engine, Renderer, Audit.

**Note on explicit re-evaluation:** Genre Resolver does not auto-fire on every DIFF.
A DIFF must explicitly request genre re-evaluation, or the Layout Engine must raise
a genre-challenge finding, to trigger the Genre Resolver to re-run (GR-2 decision).

---

#### Slot 3 — Policy

The operational parameters for this genre. Tells downstream blocks how to behave.
Isolated from semantic intent; tracked under the same version management with DIFFs
and provenance. Policy is what the Genre Resolver proposes — this is the operational
definition that separates Slot 3 from Slot 4.

**Contains:**

- **Layout algorithm binding** — which algorithm family and specific variant to use
  for the primary genre
- **Encoding conventions** — the visual encoding rules appropriate for this genre
  (e.g., for hierarchy: spatial level = importance; for function-plot: axes carry
  quantitative scale)
- **Quality measurement targets** — what the Layout Engine should optimize and report
  (e.g., for network: minimize edge crossings; for timeline: preserve temporal
  proportionality)
- **Audit check list** — which genre-specific audit checks to run (e.g., for causal
  graphs: check that directionality is unambiguous; for comparison matrices: check
  that axes are labeled)
- **Feasibility probe parameters** — parameters for running the Layout Engine in
  probe mode during genre selection
- **Autonomy Charter** — a genre-defined table specifying what the Layout Engine
  and Renderer may do autonomously vs. what requires approval. Four categories:

  | Category | Definition | Example |
  |----------|-----------|---------|
  | **Autonomous** | May do without reporting | Minor edge re-routing, whitespace redistribution, font size within ±15% |
  | **Auto-DIFF** | System commits without human confirmation, but records in audit (Tier 3) | Label abbreviation when space is tight |
  | **Proposed-DIFF** | System proposes, LLM/human confirms before committing; Tier 2 finding | Collapsing nodes for density, grouping entities, adjusting axis scale |
  | **Never-autonomous** | Requires explicit LLM/human request; Tier 1 finding if situation arises | Removing entities, merging semantically distinct nodes, changing a suasive visual's claim |

  The genre definition provides the default Autonomy Charter. LLM/human can tighten
  (move items toward more-restricted categories) but cannot loosen the Never-autonomous
  category. The charter is the primary mechanism for controlling how much the pipeline
  changes without asking — and it connects directly to the Tier system: items in
  Proposed-DIFF generate Tier 2 findings; Never-autonomous situations generate Tier 1.

**Write permission:** Three write paths, all via the Spec Manager:
1. Genre Resolver proposes (derived from genre) → system-confirmed DIFF; no human
   confirmation required since policy is determined by genre. Proposer recorded.
2. Layout Engine proposes a policy change when it encounters a layout problem the
   current policy cannot resolve → LLM/human confirms. This is how LE escalates
   issues that can't be fixed within current policy (e.g., "tight density preference
   is producing unreadable labels — propose switching to open density").
3. LLM/human overrides a policy parameter directly → committed as human DIFF.

**Note on genre-locked policy:** Some policy parameters are genuinely part of the
genre definition and cannot be overridden without effectively changing the genre.
If an author needs a meaningfully different choice, the right response may be a
different genre (or a new genre if the vocabulary doesn't cover it). Genre
proliferation is acceptable if well-organized — the genre vocabulary can grow as
real usage reveals needed distinctions.

**Re-evaluation trigger:** Any change to Slot 3 triggers Layout Engine re-run.
A Slot 3 change without a corresponding Slot 2 change is anomalous if it contradicts
genre-derived defaults; the Spec Manager flags this as a Tier 2 finding.

---

#### Slot 4 — Presentation Preferences & Policy Overrides

User-expressed preferences and overrides that affect layout and rendering but are
not genre-derived policy (Slot 3) and not pure visual style (theme, which is outside
the spec entirely per D-018).

**Operational boundary:** Policy (Slot 3) = what the Genre Resolver proposes.
Style (theme, outside spec) = what the Genre Resolver never proposes.
Slot 4 = everything in between: preferences the LLM/human holds that neither the
GR would propose automatically nor belong to the theme layer. In practice, this
boundary will sharpen as actual genres are defined.

**Contains:**
- **Layout orientation preference** — wide-favoring / tall-favoring / unconstrained
- **Density preference** — tight / open / unconstrained (may override Slot 3 default;
  if LE finds this causes problems, it proposes a Slot 3 policy DIFF, not a Slot 4 change)
- **Legend treatment** — use-legend / label-in-place / none / auto
- **Aspect ratio guidance** — explicit target ratio; otherwise derived from orientation
  preference or render context dimensions
- **Explicit policy overrides** — named Slot 3 parameters the LLM/human has explicitly
  overridden (stored here with reference to the Slot 3 parameter being overridden, so
  the override relationship is traceable)
- **Other negotiated preferences** — preferences explicitly agreed in negotiation that
  don't fit above categories. Bounded scope: cannot contain anything about colors,
  fonts, or decorative elements (those are theme); cannot contain anything the Genre
  Resolver would normally propose (that belongs in Slot 3); cannot contain semantic
  intent (that belongs in Slot 1). If a preference doesn't fit a named field and
  doesn't satisfy these constraints, it triggers a clarification rather than going
  into the catch-all.

**Write permission:** LLM/human DIFF. May also be proposed by Intent Capture when
parsing layout-flavored language in the original request ("make it wide," "use a
legend," "I want it compact").

**Re-evaluation trigger:** Any change to Slot 4 triggers Layout Engine and Renderer
re-run.

---

#### Slot 5 — Render Context Set

The named rendering environments this spec is expected to produce output for. Defining
contexts in the spec makes output reproducible and gives the Layout Engine the
information it needs to optimize for multiple targets.

**Contains:** One or more named render contexts, each with:
- **Context ID** — stable name: e.g., `screen`, `print-a4`, `low-vision`, `thumbnail`
- **Target format** — SVG / PDF / raster / alt-text
- **Target dimensions** — explicit (px, pt) or responsive parameters
- **Font environment** — which font families are available; fallback behavior
- **Background and contrast context** — background color, contrast requirements
  (relevant for low-vision context and for CMS style context)
- **Prior layout reference** — pointer into State to the most recent resolved layout
  for this context + spec version (enables stability for re-runs)

**Write permission:** LLM/human DIFF. Typically set early in a session and rarely
changed. Adding a new context triggers a Layout Engine run for that context only.

**Re-evaluation trigger:** Adding a render context triggers LE + Renderer for that
context. Modifying an existing context triggers re-run for that context. Removing
a context removes the corresponding resolved layout from State (does not trigger re-run).

**Note:** This slot resolves LE-1 and CC-4 from the block interface document. The spec
defines *which* contexts exist and their parameters; the resolved layouts and prior
layout state live in State, keyed to (spec-version-id, context-id).

---

#### Slot 6 — Audit Summary

The system's account of the current visual: what decisions were made, what was found,
and what an LLM or human needs to know to understand the current state. This is the
"result of the full loop" — it belongs in the spec because it is meaningful content,
not ephemeral computation.

**Contains:**
- **Summary of salient genre/layout decisions** that could affect visual interpretation:
  e.g., "three nodes were merged into a group because the layout was too dense at the
  requested size," "directionality was inferred from verb tense in relation labels"
- **Key Tier 1 findings** from the most recent pipeline run — findings that affect
  meaning, condensed to what matters for the LLM's next action
- **Semantic completeness assessment** — result of the accessibility completeness
  test (D-022): did the accessibility text adequately capture the meaning? If suasive:
  does it articulate the claim?
- **Unresolved findings** — Tier 1 findings that have not been addressed, carried
  forward from prior runs
- **Run reference** — pointer to the corresponding entry in the Full Audit Record
  in State, for LLM or human who wants the complete account

**Write permission:** Audit block only, system DIFF, after each pipeline run.
Does not require LLM/human confirmation. Does not trigger any further re-evaluation
(it is the terminal output of a pipeline run, not an input to one).

**Versioning note:** The audit summary DIFF is recorded with the spec version that
produced it as parent. It creates a new spec version but does not cause a pipeline
re-run — the system recognizes Slot 6 DIFFs as terminal. The effect: each pipeline
run produces a new spec version consisting of the prior semantic content + the new
audit summary.

---

### Version Model

The spec is a linear chain of committed DIFFs with a stable current-version pointer.
v1 supports rollback only. Branching is deferred to v2 — the Variation Generator
already handles the primary "explore alternatives" use case, and full branching adds
significant State complexity for limited v1 benefit.

**Each DIFF record contains:**
- DIFF ID (stable, addressable)
- Parent version ID
- Timestamp
- Proposer type: `llm` | `human` | `system`
- Proposer block: `intent-capture` | `genre-resolver` | `layout-engine` |
  `audit` | `human-direct` | `variation-generator`
- Target slot(s) affected
- Change content (structured; format TBD pending Q-003)
- Confirmation record: who confirmed, when (for DIFFs requiring confirmation)
- Rejection record: if rejected, who rejected and why (preserved for provenance)

**Rollback:** Move the current-version pointer to any prior version in the chain.
No work is lost — all DIFFs remain in the chain. The rolled-back state reactivates
resolved layouts and audit records from State if they are still cached for that
version. A rollback followed by a new DIFF creates a new chain from the rollback
point, orphaning the previously-current versions (they remain in the DIFF chain
but are no longer on the path from root to current).

**Branching:** Deferred to v2.

---

### What the Spec Is Not

To prevent scope creep:

- **Not the theme.** Colors, fonts, line weights, iconography — all outside the spec.
  Theme is configuration passed to the Renderer.
- **Not the resolved layout.** Geometric positions, edge curves, absolute sizes — 
  all in State.
- **Not the full audit record.** The detailed three-tier finding log is in State.
  The spec carries the summary and unresolved Tier 1 findings only.
- **Not conversation history.** The negotiation transcript is not part of the spec.
  The spec records what was agreed, not how it was negotiated.
- **Not render-target artifacts.** SVG files, PDFs, raster images are outputs, not spec content.

---

## State

State is the runtime envelope that contains the spec and everything computed from it.
State is per-session but designed so that a session can be resumed from State.

```
State {
    spec:                    Spec                   // versioned; canonical; see above
    resolved_layouts:        Map<LayoutKey, Layout> // keyed to (spec_version_id, context_id)
    full_audit_record:       AuditRecord            // cumulative; three-tier; detailed
    pipeline_exec_record:    List<PipelineRun>      // interceptor data; ephemeral
    session_metadata:        SessionMetadata        // IDs, timestamps, participants
}

LayoutKey {
    spec_version_id:  string
    context_id:       string   // matches a context ID in Spec Slot 5
}
```

### Resolved Layouts

Each resolved layout corresponds to a specific spec version rendered in a specific
render context. Contains:

- Abstract geometry: entity positions, sizes, edge geometry, spatial annotations
  (all unit-invariant per D-019)
- Layout quality measurements: edge crossings, whitespace balance, label collision
  count, density metric — whatever the genre's quality targets specify (Slot 3)
- Stability reference: this layout *is* the prior layout for the next run in the same
  context. When the spec version advances, the prior run's layout is retrieved from
  `resolved_layouts[(prior_version_id, context_id)]`

Resolved layouts are recomputable from the spec + render context. They are cached in
State for performance and for stability (prior layout needed as LE input). Cache
eviction policy is a runtime concern, not an architectural one.

### Full Audit Record

The detailed three-tier finding log. Cumulative across pipeline runs within a session.

**Structure:**
```
AuditRecord {
    runs: List<RunRecord>
}

RunRecord {
    run_id:           string
    spec_version_id:  string
    context_id:       string
    timestamp:        datetime
    findings: {
        tier1: List<Finding>   // meaning-affecting; always surfaced to LLM
        tier2: List<Finding>   // revisable decisions; surfaced during negotiation
        tier3: List<Finding>   // routine processing record; on-request only
    }
    cross_cutting:    List<Finding>  // emergent findings that span blocks
    block_io_refs:    Map<BlockId, IORecordRef>  // references to interceptor data
}

Finding {
    finding_id:       string   // stable; referenced from Spec Slot 6
    tier:             1 | 2 | 3
    emitting_block:   BlockId
    spec_element_ref: string   // stable ID of the spec element this finding concerns
    description:      string   // what was found
    decision_taken:   string   // what the system did (not prescriptive; factual)
    status:           open | resolved | dismissed | stale
    resolution:       string?  // if resolved: how; if dismissed: by whom and why
}
```

**Finding lifecycle:** Findings begin as `open`. They become `resolved` when the
spec changes in a way that addresses them (auto-resolution requires explicit
resolution criteria per finding type — this is part of the finding taxonomy work,
AR-3). They become `dismissed` on explicit LLM/human action. They become `stale`
when the spec has changed in a related area but the finding's status is unclear —
the Audit block re-evaluates stale findings on the next run.

**Relationship to Spec Slot 6:** After each run, the Audit block derives the Slot 6
content (audit summary) from the most recent RunRecord. Slot 6 is a curated view;
the RunRecord is the complete record.

### Pipeline Execution Record

Raw interceptor data: for each pipeline run, each block's full input and output,
timestamped and sequenced. This is the data source for the Audit block's cross-cutting
analysis (AR-1 decision: interceptor model).

This data is not semantically meaningful on its own — it is operational telemetry.
Retention policy is a deployment concern. It need not survive session boundaries
(the Audit block derives its findings during the run; the Full Audit Record persists
those findings; the raw I/O can be evicted).

### Session Metadata

```
SessionMetadata {
    session_id:       string
    started_at:       datetime
    participants:     List<{role: "llm" | "human", identifier: string}>
    active_version:   string   // current-version pointer into the DIFF chain
}
```

---

## Read/Write Permissions Matrix

| Component | Spec Slot 1 | Slot 2 | Slot 3 | Slot 4 | Slot 5 | Slot 6 | Resolved Layouts | Full Audit Record |
|-----------|-------------|--------|--------|--------|--------|--------|-----------------|-------------------|
| Intent Capture | Propose | — | — | Propose | — | Read | — | — |
| Spec Manager | Commit/Reject | Commit/Reject | Commit/Reject | Commit/Reject | Commit/Reject | Commit | — | — |
| Genre Resolver | Read | Propose | Propose | Read | — | — | — | — |
| Layout Engine | Read | Read | Read | Read | Read | — | Write | — |
| Renderer | Read | Read | Read | Read | Read | — | Read | — |
| Audit Block | Read all | Read all | Read all | Read all | Read all | **Write** | Read | **Write** |
| Variation Generator | Read | Read | Read | Read | Read | Read | Read | — |
| Content Measurement | — | — | Read (style context) | — | Read (font env) | — | — | — |
| Human (direct) | Propose | Confirm/Propose | — | Propose | Propose | Read | — | — |

**Key:** Propose = submit a DIFF for consideration. Commit = accept and record a DIFF.
Write = write directly to State (not via DIFF). Read = read-only access.

Note that no block writes directly to Spec Slots 1–5. All spec writes go through
Spec Manager as committed DIFFs. Slot 6 (Audit Summary) is the one exception:
the Audit block writes it via a system DIFF that the Spec Manager commits automatically
(no human confirmation required, but provenance is recorded).

---

## Re-Evaluation Triggers

When a DIFF is committed to a spec slot, which blocks re-run?

| DIFF committed to | Triggers |
|-------------------|---------|
| Slot 1 (Semantic Intent) | Genre Resolver (if change is genre-relevant or explicitly requested); always Layout Engine, Renderer, Audit |
| Slot 2 (Genre Commitment) | Slot 3 policy derivation (Genre Resolver); Layout Engine; Renderer; Audit |
| Slot 3 (Policy) | Layout Engine; Renderer; Audit |
| Slot 4 (Presentation Preferences & Policy Overrides) | Layout Engine; Renderer; Audit |
| Slot 5 (Render Context Set) — add context | Layout Engine (new context only); Renderer (new context only); Audit |
| Slot 5 — modify context | Layout Engine (modified context only); Renderer; Audit |
| Slot 5 — remove context | No re-run; remove corresponding layout from State |
| Slot 6 (Audit Summary) | Nothing — terminal; does not trigger re-run |

Genre Resolver does **not** auto-fire on every Slot 1 DIFF. It fires when: the DIFF
explicitly requests genre re-evaluation; or the Layout Engine raises a
genre-challenge finding.

---

## Export / Embed Model

When the Renderer produces a visual artifact (SVG or other format supporting metadata):

**Default embed (always):**
- Accessibility text (full, from Accessibility Renderer)
- Genre token (from Slot 2)
- Spec version hash (stable fingerprint of the spec version that produced this output)
- Rhetorical purpose flag + claim string if suasive (from Slot 1)
- Session ID (for traceability without exposing content)

**Optional embed (user-controlled, off by default):**
- Full spec (all six slots, current version serialized)

The default embed is sufficient for an LLM consuming the artifact to understand what
the visual means, what genre it is, and what claim it makes (if any), without exposing
the full semantic graph. The full embed enables re-editing: loading a visual re-hydrates
the spec into a new session.

This is a property of the render context — each context can specify its embed policy.
A `thumbnail` context might embed nothing; a `print-a4` context might embed the full spec.

---

## Open Questions Surfaced by This Document

**SS-1 — Finding taxonomy for auto-resolution (AR-3).**
The `Finding` structure includes auto-resolution criteria, but the criteria themselves
require a finding taxonomy to be defined. This is the "challenging problem" flagged in
the AR-3 note. It is a prerequisite for the full audit lifecycle to work correctly.
Deferred to a dedicated session after Q-002/Q-003.

**SS-2 — Slot 3 derivation: fully deterministic or parameterized? [RESOLVED]**
Resolved: Policy has genre-defined defaults (deterministic derivation from genre) but
is overridable by LLM/human (Slot 4 policy overrides) or by LE proposals when layout
problems arise. Slot 3 stores the active policy explicitly for provenance and
inspectability. Consistency between Slot 2 and Slot 3 is enforced by the Spec Manager
flagging divergence from genre defaults as a Tier 2 finding (not an error — deliberate
overrides are valid).

**SS-3 — Branch management in State. [RESOLVED — deferred]**
Resolved: v1 uses linear history with rollback only. Branching is deferred to v2.
The Variation Generator covers the primary "explore alternatives" use case. A rollback
followed by a new DIFF orphans the previously-current versions (they remain in the
chain but are no longer on the path to current). No eviction policy needed for v1
since there is only one active chain.

**SS-4 — CMS style context source.**
The Content Measurement Service needs font metrics from the render context (Slot 5),
not the theme. The Layout Engine must pass a measurement context (font families and
sizes from the active render context) separately from the full theme when querying
CMS. Needs resolution in Q-010.

**SS-5 — Slot 4 catch-all scope.**
Partially resolved: Slot 4 ("Presentation Preferences & Policy Overrides") now has
explicit exclusion rules for what cannot go in the catch-all (colors/fonts → theme;
GR-proposed parameters → Slot 3; semantic intent → Slot 1). A preference that doesn't
fit a named field and violates these constraints triggers clarification rather than
catch-all storage. Full resolution requires actual genre definitions to make the
boundary concrete.

**SS-6 — Analytical computation scope for v1.**
The Data Section in Slot 1 accommodates derived series (regression curves, projections)
as pre-computed values provided by the user/LLM. For v1, the pipeline does not compute
analytics — it visualizes what it is given. Analytical computation (fitting regression
models, generating forecasts, computing moving averages) is a v2 extension. This
boundary needs to be made explicit in the Intent Capture block's "must not do" list
and in the Audit block's check for analytical directives that imply computation the
pipeline cannot perform.

---

*Document updated after Q-001 review session. Ready for use as structure reference
in Q-002 (Diff Vocabulary) and Q-003 (Spec Representation) sessions.*
