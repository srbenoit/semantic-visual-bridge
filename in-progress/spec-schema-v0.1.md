# SVB Spec Schema — v0.1 Skeleton

**Status:** Round 1 draft — structural decisions only. Content types, validation rules,
and per-genre policy details are deliberately omitted. The goal is to make the shape
decisions visible and stress-testable.

**Shape decision recorded here:** The spec is a tree at the view-composition level.
Cross-element and cross-view semantic relations are represented as a named flat list
(`relations`) separate from the tree. This is the AST-with-cross-references pattern.

---

## Top-Level Envelope

```json
{
  "svb": "1.0",
  "id": "spec-uuid-stable-across-versions",
  "version": 7,
  "created": "ISO-8601",
  "modified": "ISO-8601",

  "slot1": { ... },
  "slot2": { ... },
  "slot3": { ... },
  "slot4": { ... },
  "slot5": { ... },
  "slot6": { ... }
}
```

**Notes:**
- `id` is stable across all versions of this spec (the spec's identity).
- `version` is a monotonically increasing integer. Each confirmed DIFF increments it.
- The six slots are present at the top level, not nested.

---

## Slot 1 — Semantic Intent

```json
"slot1": {
  "data": {
    "datasets": [
      {
        "id": "ds-sales-2020",
        "label": "Annual Sales 2020–2024",
        "type": "table",
        "fields": [
          { "id": "f-year",  "label": "Year",  "type": "ordinal" },
          { "id": "f-sales", "label": "Sales", "type": "quantitative", "unit": "USD" }
        ],
        "inline": [
          { "f-year": 2020, "f-sales": 142000 },
          { "f-year": 2021, "f-sales": 158000 }
        ]
      }
    ]
  },

  "intent": {
    "entities": [
      {
        "id": "e-sales-series",
        "role": "primary",
        "label": "Annual Sales",
        "dataRef": { "dataset": "ds-sales-2020", "field": "f-sales" }
      }
    ],
    "relations": [
      {
        "id": "r-001",
        "type": "temporal-sequence",
        "from": "e-sales-series",
        "to": "f-year",
        "label": "sales over time"
      }
    ],
    "analyticalDirectives": [
      { "id": "ad-001", "type": "show-trend", "target": "e-sales-series" }
    ],
    "rhetoricalPurpose": {
      "present": true,
      "claim": "Sales have grown consistently since 2020.",
      "claimType": "trend-assertion"
    },
    "compositionalStructure": null
  }
}
```

**Notes:**
- `data.datasets` holds bulk structured data. Types include `table`, `series`,
  `tree`, `graph`, `scalar`. Data stays here; it is referenced from `intent` by ID.
- `intent.entities` are the semantic objects being visualized. Each has a stable `id`,
  a functional `role` (primary, reference, annotation, grouping), a human label, and
  an optional data reference.
- `intent.relations` is the cross-element relation list — the "cross-references" in
  the AST-with-cross-references pattern. Relations between entities live here, not
  embedded in the tree. Types will be genre-informed (temporal-sequence,
  part-of, causes, compared-to, same-as, etc.).
- `intent.analyticalDirectives` express what the visual should do: show-trend,
  show-distribution, show-comparison, highlight-outliers, etc.
- `intent.rhetoricalPurpose` is optional. When present, it records an explicit claim
  the visual is making. Absence means the visual is descriptive, not suasive.
- `intent.compositionalStructure` is null for single-panel visuals. For multi-panel,
  it describes the panel tree (see Slot 1 — Composite Extension below).

---

## Slot 1 — Composite Extension (Multi-Panel)

When `compositionalStructure` is non-null, it describes the panel tree:

```json
"compositionalStructure": {
  "id": "comp-root",
  "layout": "grid-2x2",
  "alignAxes": ["x"],
  "panels": [
    {
      "id": "panel-a",
      "label": "Q1 Sales",
      "specRef": null,
      "dataFilter": { "dataset": "ds-sales-2020", "field": "f-quarter", "value": "Q1" }
    },
    {
      "id": "panel-b",
      "label": "Q2 Sales",
      "specRef": null,
      "dataFilter": { "dataset": "ds-sales-2020", "field": "f-quarter", "value": "Q2" }
    }
  ]
}
```

**Notes:**
- Each panel has a stable `id` used for cross-panel relations (Tier 7).
- `alignAxes` is a semantic claim ("these panels share a scale") not a layout hint.
- `specRef` is null when the panel inherits the parent's genre and intent with only
  a data filter applied (the small-multiples case). Non-null when the panel has its
  own full sub-spec (the dashboard case — not in v1 scope, reserved).
- Cross-panel relations (e.g., "panel-a element e-001 is-same-as panel-b element e-002")
  live in `intent.relations`, referencing panel-scoped element IDs.

---

## Slot 2 — Genre Commitment

```json
"slot2": {
  "genre": "categorical-chart",
  "subtype": "bar",
  "confidence": 0.92,
  "proposedBy": "genre-resolver",
  "confirmedBy": "llm",
  "confirmedAt": "ISO-8601",
  "alternatives": [
    { "genre": "function-plot", "confidence": 0.31, "reason": "data is ordinal not continuous" }
  ]
}
```

**Notes:**
- `genre` is a value from the closed v1 genre vocabulary (to be finalized in Q-005).
- `subtype` is genre-specific optional refinement.
- `confidence` is the Genre Resolver's confidence at time of proposal.
- `proposedBy` / `confirmedBy` implement the provenance tagging required by D-028.
- `alternatives` records what was considered and rejected, with reasoning. This is
  part of the audit record at the slot level.

---

## Slot 3 — Policy

```json
"slot3": {
  "layoutAlgorithm": {
    "family": "cartesian",
    "algorithm": "ordinal-bar",
    "parameters": {
      "orientation": "vertical",
      "grouping": "none",
      "sortOrder": "data-order"
    }
  },
  "encodings": {
    "x": { "field": "f-year", "scale": "ordinal" },
    "y": { "field": "f-sales", "scale": "linear", "includeZero": true },
    "color": null
  },
  "autonomyCharter": {
    "autonomous": ["label-truncation", "tick-density"],
    "autoDiff":   ["aspect-ratio", "bar-width"],
    "proposedDiff": ["axis-range", "sort-order"],
    "neverAutonomous": ["which-field-encodes-x", "which-field-encodes-y"]
  },
  "derivedBy": "genre-resolver",
  "derivedAt": "ISO-8601"
}
```

**Notes:**
- `layoutAlgorithm` is the binding from Genre Resolver policy to Layout Engine
  (D-009). The Layout Engine reads this, not the full spec.
- `encodings` maps semantic fields (by field ID from Slot 1 data) to visual channels.
  This is where the Vega-Lite encoding model lives in our architecture.
- `autonomyCharter` implements D-030. The four categories correspond to the four
  autonomy levels. Values are named capabilities, not parameters — the Layout Engine
  knows what each capability means.
- `derivedBy` / `derivedAt` provenance. Slot 3 is system-confirmed (no human
  confirmation required) but provenance is recorded.

---

## Slot 4 — Presentation Preferences & Policy Overrides

```json
"slot4": {
  "overrides": [
    {
      "id": "ov-001",
      "target": "slot3.layoutAlgorithm.parameters.sortOrder",
      "value": "descending",
      "reason": "highlight highest performers first",
      "setBy": "llm"
    }
  ],
  "preferences": [
    {
      "id": "pref-001",
      "type": "emphasis",
      "target": "e-sales-series",
      "level": "high",
      "setBy": "human"
    }
  ]
}
```

**Notes:**
- `overrides` are explicit reversals of Slot 3 policy values. They reference the
  target by JSON path within Slot 3. The Genre Resolver would not produce these —
  they are human/LLM preference (D-033 boundary).
- `preferences` express things the Genre Resolver has no opinion on: emphasis,
  annotation requests, focus regions. These become inputs to the Layout Engine
  and Renderer alongside Slot 3 policy.
- `setBy` tracks whether the preference came from the LLM or a human direct edit.

---

## Slot 5 — Render Context Set

```json
"slot5": {
  "contexts": [
    {
      "id": "ctx-screen",
      "label": "Screen HD",
      "format": "svg",
      "width": 1200,
      "height": 800,
      "units": "px",
      "fontEnvironment": {
        "mathRenderer": "mathjax",
        "bodyFont": "system-ui",
        "monoFont": "monospace"
      },
      "accessibility": {
        "generateAltText": true,
        "contrastMode": "standard"
      }
    },
    {
      "id": "ctx-print",
      "label": "Print A4",
      "format": "pdf",
      "width": 170,
      "height": 257,
      "units": "mm",
      "fontEnvironment": {
        "mathRenderer": "mathjax",
        "bodyFont": "Latin Modern",
        "monoFont": "Latin Modern Mono"
      },
      "accessibility": {
        "generateAltText": false,
        "contrastMode": "print"
      }
    }
  ]
}
```

**Notes:**
- Each render context is a named profile (D-019). Resolved layouts in State are
  keyed to (spec-version-id, context-id).
- `fontEnvironment` is what the Content Measurement Service reads (D-037) — not
  from a theme.
- Dimensions are authored intent. The Layout Engine produces unit-invariant geometry
  (D-012); the Renderer resolves to the context's units at render time.

---

## Slot 6 — Audit Summary

```json
"slot6": {
  "pipelineVersion": "1.0.0",
  "renderedAt": "ISO-8601",
  "specVersion": 7,
  "findings": [
    {
      "id": "f-001",
      "tier": 3,
      "source": "layout-engine",
      "message": "Bar width auto-adjusted to 18px to avoid overlap at current aspect ratio.",
      "specRef": "slot3.layoutAlgorithm.parameters.bar-width",
      "autonomous": true
    }
  ],
  "unresolvedCount": { "tier1": 0, "tier2": 0, "tier3": 1 },
  "status": "clean"
}
```

**Notes:**
- Slot 6 is written by the Audit block only (D-029). It is a curated summary;
  the full three-tier finding record lives in State.
- `findings` here are only Tier 1 and Tier 2 items (which the LLM needs to act on)
  plus a count of Tier 3. Full Tier 3 detail is in State.
- `specRef` references the affected slot/field by JSON path, enabling the LLM to
  address findings precisely.
- `status` is a summary signal: `clean` (no unresolved Tier 1/2), `needs-review`
  (unresolved Tier 2), `blocked` (unresolved Tier 1).

---

## Element Addressing

Every addressable element in the spec has a stable `id`. The addressing scheme is:

```
{slot}.{section}.{element-id}
```

Examples:
- `slot1.intent.entities.e-sales-series` — a specific entity
- `slot1.intent.relations.r-001` — a specific relation
- `slot3.autonomyCharter` — the charter as a whole
- `slot3.encodings.x` — the x encoding channel
- `slot6.findings.f-001` — a specific finding

DIFFs reference elements by these paths. The `id` fields provide stable identity
even when the surrounding tree structure changes (e.g., an entity reordered in the
array is still `e-sales-series`, not `entities[0]`).

---

## What Is Deliberately Omitted from v0.1

- Complete genre vocabulary (Q-005)
- Full data type taxonomy (table, series, tree, graph, math, scalar, image)
- Math content type schema
- Full relation type vocabulary
- Per-genre policy detail
- DIFF operation schema (Q-002)
- Validation rules and required vs. optional fields

---

*v0.1 — structural skeleton only. Round 2: stress test against use cases.*
