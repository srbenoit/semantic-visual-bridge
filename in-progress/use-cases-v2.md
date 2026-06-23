# Use Cases — Expanded Taxonomy
# SVB In-Progress / Thought Experiment

**Status:** Working document. Expanded from initial tier-based taxonomy to cover all five
content types. Representative cases selected to stress the spec schema.

**Relationship to tiers:** The original Tier 1–7 taxonomy organized cases by *structural
complexity of the visual composition*. This document reorganizes by *content type* —
the nature of what is being represented, not how it is composed. Both dimensions matter.
A Type 2 (geometric) case can appear as a Tier 1 (single entity) or Tier 6 (composite).

---

## Content Type 1 — Tabular / Statistical Data

*The dataset is the anchor. Intent describes what analytical claim to make about it.*

### Simple Cases
- Bar chart: categorical comparison of a single quantitative field
- Line chart: a quantitative field over a continuous or ordered axis
- Pie / donut: part-to-whole decomposition of a single field
- Scatter plot: relationship between two quantitative fields
- Histogram: distribution of a single quantitative field
- Box plot: distributional summary (median, quartiles, outliers) of a field by category

### More Demanding Cases
- Multi-series line chart: same analytical frame, multiple series, comparison claim
- Scatter with regression overlay: point data + derived statistical object (regression line
  is not in the raw data — it is computed from it)
- Dual-axis chart: two fields with incommensurable scales, shared x-axis, comparison claim
- Heatmap: two categorical dimensions, one quantitative field encoded as color
- Bubble chart: scatter plus a third quantitative field encoded as mark size

### Structurally Interesting Cases
- Survival curve (Kaplan-Meier): time-to-event data with censoring — unusual data type
- Error bar chart: each entity has a central value and an uncertainty — two fields per entity,
  one rendered as position, one as extent
- Waterfall chart: running total with incremental steps — each bar's position depends on
  prior bars (non-independent encoding)
- Parallel coordinates: high-dimensional data, each dimension an axis — challenges the
  standard x/y encoding model

### Structural Challenges Surfaced
- **Derived objects:** Regression lines, confidence intervals, trend lines are not in the
  raw data. They are computed from it. Does the spec represent them as content, or as
  analytical directives that the pipeline executes? This is a Slot 1 / Slot 3 boundary
  question with no clean answer yet.
- **Data-dependent geometry:** Some chart types have marks whose positions depend on
  accumulated data values (waterfall, stacked bar). The Layout Engine needs access to
  data values, not just field-to-channel mappings.
- **Statistical objects as first-class content:** A confidence interval is not just a
  visual decoration — it is a statistical claim. It may need to be an entity in
  Slot 1 intent, not merely a rendering parameter in Slot 3.

---

## Content Type 2 — Mathematical / Geometric Content

*The content is a mathematical structure — geometric objects, symbolic expressions,
function definitions. Intent describes what the diagram should show about that structure.*

### Simple Cases
- Labeled geometric figure: a triangle with vertices A, B, C, angle markers, side labels
- Number line: an interval with marked points, arrows indicating direction or range
- Function plot: y = f(x) over a specified domain, with labeled intercepts or asymptotes
- Unit circle: canonical labeled diagram showing angle, coordinates, trig values
- Venn diagram (set-theoretic): two or three sets with labeled regions, no quantitative data

### More Demanding Cases
- Geometric construction: compass-and-straightedge steps shown in sequence, each step
  adding new objects to the figure
- Proof diagram: figure annotated to support a specific geometric argument (congruence
  proof with tick marks and angle marks, similarity proof with proportion labels)
- Implicit curve plot: f(x,y) = 0 in the plane — not a function, may have multiple
  branches, may not be expressible as y = f(x)
- Parametric curve: (x(t), y(t)) with direction arrows indicating parameter direction
- Phase portrait: vector field overlaid with solution curves — multiple semantic layers
- 3D surface: z = f(x,y) in perspective or orthographic view with optional contour lines

### Structurally Interesting Cases
- Commutative diagram: mathematical objects as nodes, morphisms as labeled arrows,
  arranged so path-independence is visually apparent — Tier 4 (relational) applied to
  mathematical objects, with very specific layout constraints
- Euler diagram: like Venn but sets can be non-overlapping or properly contained —
  the containment relationships are the content, not data frequencies
- Cobordism / manifold diagram: topological surfaces with labeled boundaries — the
  "content" is a mathematical object with no natural coordinate representation
- Truth table: table structure, but content type is logical rather than statistical —
  rows are logical cases, not data observations

### Structural Challenges Surfaced
- **Geometric objects as content:** The "data" for a geometric figure is a set of
  mathematical objects (points, lines, angles, curves) with properties and constraints.
  A point has coordinates; a line has a defining equation or two-point specification;
  an angle has a vertex and two rays. This is fundamentally different from a data table
  and requires a distinct content schema.
- **Semantic annotations:** Tick marks (equal lengths), arc marks (equal angles),
  right-angle markers, and dimension labels are semantic — they assert mathematical
  claims. They are part of the content, not presentation preferences. They belong in
  Slot 1, not Slot 4.
- **Sequence and construction order:** Geometric constructions and proof diagrams have
  an inherent ordering that carries meaning. The spec currently has no representation
  for sequence within a single-panel visual.
- **Mathematical expressions as labels:** Axis labels and entity labels may be
  mathematical expressions, not plain strings. Math must be first-class content
  (D-013), including in annotation and label positions.

---

## Content Type 3 — Relational / Structural Content

*Named entities with typed relations. No dataset, no geometry — just nodes and edges
with semantic labels. The structure itself is the content.*

### Simple Cases
- Flowchart: process steps with decision branches and flow arrows
- Org chart: hierarchy with labeled roles
- Family tree / genealogy: hierarchical with marriage and sibling relations
- Mind map: central concept with radiating associations
- Simple directed graph: nodes with labeled directed edges

### More Demanding Cases
- UML class diagram: classes with attributes, methods, and typed relationships
  (inheritance, composition, aggregation, association)
- Entity-Relationship diagram: entities, attributes, and relationship types
  (one-to-many, many-to-many, optional, required)
- State machine: states with transitions labeled by triggers and guards
- Dependency graph: directed acyclic graph with typed dependency edges
- Concept map: heterogeneous relation types between concepts — not all edges are
  the same kind of relationship

### Structurally Interesting Cases
- Petri net: places, transitions, and tokens — a computational model with specific
  visual conventions and formal semantics
- Causal diagram (DAG): nodes are variables, edges are causal claims — resembles
  a graph but carries specific inferential and statistical semantics
- Argument map: claims, evidence, objections, rebuttals — typed rhetorical relations
  between propositions
- BPMN process diagram: business process with swim lanes, events, gateways —
  multiple simultaneous layout constraints

### Structural Challenges Surfaced
- **Relation types drive layout algorithm selection:** An org chart (strict tree,
  top-down) requires a completely different layout algorithm than a concept map
  (unconstrained graph, force-directed). The genre selection depends heavily on
  the relation type vocabulary in Slot 1.
- **Standardized visual grammars:** UML, BPMN, and Petri nets have formal visual
  grammars with standardized symbols. The content includes symbol type, not just
  entity type. This is a genre-level concern — the symbol vocabulary is part of
  the genre definition, not part of the authored content.
- **Semantic layout constraints:** In a causal DAG, temporal/causal order typically
  flows left-to-right. This is not a stylistic preference — it is a semantic
  convention that carries inferential meaning. Belongs in Slot 3 policy.

---

## Content Type 4 — Physical / Schematic Content

*Components with connection topology. Intent describes what the schematic communicates
about the physical or engineered system.*

### Simple Cases
- Basic circuit diagram: resistors, capacitors, voltage sources, wires
- Free body diagram: an object with labeled force vectors
- Simple plumbing schematic: pipes, valves, pumps with flow direction arrows
- Block diagram: functional blocks with signal flow arrows between them

### More Demanding Cases
- Op-amp circuit: standard symbols, feedback paths, labeled node voltages and currents
- Digital logic circuit: gates, flip-flops, clock signals, bus notation
- Control system diagram: plant, controller, sensor with feedback loop — specific
  layout conventions (typically left-to-right with feedback returning bottom)
- Network topology: physical or logical, with device symbols and connection types
- Hydraulic / pneumatic circuit: domain-specific component symbols, flow direction

### Structurally Interesting Cases
- Signal flow graph: nodes are signals, edges are transfer functions — a mathematical
  structure (Type 2) with physical interpretation
- Ladder diagram (PLC): industrial control notation with horizontal rails, contacts
  and coils — a very specific domain visual grammar
- Feynman diagram: particles as lines, interactions as vertices — highly stylized
  physical schematic with non-obvious visual grammar conventions
- Molecular structure diagram: atoms as nodes, bonds as edges, with 2D structural
  conventions (Lewis structure vs. skeletal formula vs. space-filling)

### Structural Challenges Surfaced
- **Domain-specific symbol vocabularies:** IEEE electrical symbols, ISO hydraulic
  symbols, chemical structural conventions are large and standardized. The spec
  needs a way to reference a component by type without describing its geometry —
  the symbol library is a genre-level resource.
- **Topology as content:** In a circuit, the connection topology *is* the content.
  Two circuits that are topologically equivalent but drawn differently are the same
  circuit. The spec must distinguish topological identity from spatial layout.
- **Implicit connectivity conventions:** In a circuit, ground symbols can appear
  visually disconnected but are connected by convention. Voltage supply rails have
  similar implied connectivity. These conventions belong in the genre's symbol
  vocabulary, not in explicitly authored relations.
- **Mixed content type boundary:** Signal flow graphs (Type 4) have formal
  mathematical semantics (transfer functions). Molecular diagrams border on Type 2
  (geometric constraints on bond angles). The content type boundary is fuzzy at
  the edges.

---

## Content Type 5 — Conceptual / Rhetorical Content

*An idea structure — metaphor, process, argument, visual analogy. The communicative
intent is the primary content; the visual realization is relatively unconstrained.*

### Simple Cases
- Process arrow diagram: a linear sequence of conceptual steps (A → B → C → outcome)
- Before / after pair: two states of a system with a transformation indicated
- Analogy diagram: "X is to Y as A is to B" shown visually
- Simple visual metaphor: a concept represented by a familiar physical object
  (pipeline for data flow, funnel for filtering, bridge for transition)

### More Demanding Cases
- Iceberg diagram: visible part / hidden part metaphor for a concept with surface
  and depth (surface behavior vs. underlying mechanism)
- Virtuous / vicious cycle: reinforcing feedback loop shown as a circular process
- Pyramid / hierarchy metaphor: importance, quantity, or precedence shown as
  layered pyramid (Maslow's hierarchy, etc.)
- Journey map: narrative sequence with stages, emotional valence, and touchpoints —
  more structured than a simple timeline
- Comparison matrix: not quantitative data, but conceptual attributes compared
  across options — a qualitative table with semantic cell content

### Cases Specific to Math / Teaching Domain
- **Function machine:** input / funnel / output metaphor for a mathematical function —
  the "machine" is the genre, the function definition is the content. Common in
  middle-school and introductory college mathematics.
- **Proof strategy diagram:** "we want to show P; we'll show P ↔ Q, then show Q" —
  a metacognitive map of an argument structure, not the proof itself
- **Conceptual bridge:** two mathematical domains shown as islands with labeled
  connections — used to explain analogies between structures (e.g., derivatives
  and differences, integrals and sums)
- **Visual definition:** a concept shown through a carefully labeled example and
  non-example pair — the pair structure and the contrast are the content
- **Transformation diagram:** an object before and after a mathematical operation
  (rotation, reflection, translation) with the operation labeled and illustrated

### Structural Challenges Surfaced
- **Intent-heavy, data-light:** For most Type 5 cases, the "content" lives almost
  entirely in the intent section. The rhetorical purpose *is* the content. The data
  section may be nearly empty or contain only light anchoring information.
- **Loose genre vocabulary:** "Function machine" is a recognizable genre in math
  pedagogy but has no standard definition outside that context. Type 5 genres will
  require more negotiation and more Tier 2 findings than Types 1–4.
- **Acceptability criterion is loose (feature, not bug):** Unlike data visualizations
  where there is a ground truth to be wrong about, Type 5 diagrams have many valid
  realizations. The pipeline has more freedom and less ability to produce definitively
  wrong output. This makes Type 5 more tractable to ship something useful for, even
  if formal specification is harder.
- **Mixed content type at boundaries:** A "function machine" diagram combines a
  conceptual metaphor (Type 5) with mathematical content — the function definition
  and example input/output (Type 2). Type 5 diagrams frequently incorporate content
  from other types. This is the confection boundary case, handled in v1 as a primary
  genre with insets (D-010).

---

## Cross-Cutting Observations

These challenges appear across multiple content types and will need explicit treatment
in the spec schema.

### The Annotation Problem
Every content type has annotations — labels, tick marks, dimension lines, callout
boxes, emphasis markers. Annotations are semantic (they assert something about the
content) but sit awkwardly in the spec: too specific for Slot 1 intent, too
content-dependent for Slot 3 policy.

Candidate resolution: Annotations are entities in Slot 1 intent with role = "annotation",
a content reference pointing to what they annotate, and a semantic type (label,
measurement, emphasis, explanation). The Layout Engine places them; the genre policy
governs their style.

### Symbol Vocabularies
Types 3, 4, and parts of 2 rely on standardized domain symbol sets. The spec needs
a way to reference a symbol by semantic type without encoding its geometry. This is a
genre-level resource — the symbol vocabulary is part of the genre definition, not
authored content. Genre policy (Slot 3) should include a symbol vocabulary reference.

### The Derived Object Problem
Appears across types:
- Type 1: regression line, confidence interval (derived from data)
- Type 2: angle bisector, perpendicular bisector (derived from geometric objects)
- Type 4: Thevenin equivalent (derived from a circuit)

Derived objects are computed content, not authored content. Candidate resolution:
a `derived` flag in the content section, with a `derivedFrom` reference and a
`derivationMethod` field. Analytical directives (Slot 1 intent) trigger derivation;
the pipeline executes it; the result is added to content as a derived item.

### Sequence and Narrative Structure
Appears in:
- Type 2: geometric constructions, proof diagrams (construction order carries meaning)
- Type 5: process diagrams, journey maps, proof strategy diagrams
- Tier 6 composite: visual proofs, sequential panels

The spec currently has no representation for ordered sequence within a single panel.
Candidate resolution: an optional `sequence` field on an entity set in Slot 1, which
imposes a total or partial order on the entities. The Layout Engine interprets sequence
as a layout constraint (left-to-right, top-to-bottom) unless the genre overrides it.

### The Confection Boundary
Several use cases mix content types within a single visual:
- Function machine: Type 5 metaphor + Type 2 mathematical content
- Annotated circuit with equations: Type 4 schematic + Type 2 math labels
- Data chart with conceptual callout: Type 1 chart + Type 5 annotation

In v1, these are handled as "primary genre with insets" (D-010). The primary content
type determines the genre; inset content follows the primary genre's policy for
embedded content. Full multi-genre confection support is deferred to v2.

---

## Tentative Scope Assessment for v1

| Content Type | Core Cases | Structural Challenge Level | v1 Assessment |
|---|---|---|---|
| Type 1: Tabular/Statistical | Charts, plots, distributions | Medium — derived objects, data-dependent geometry | In scope; well-supported by existing genre work |
| Type 2: Mathematical/Geometric | Function plots, geometric figures, proof diagrams | Medium-High — geometric content schema, sequence | In scope; requires geometry content type in Slot 1 |
| Type 3: Relational/Structural | Flowcharts, graphs, UML | Medium — relation type vocabulary, symbol sets | In scope; relations list in spec handles this |
| Type 4: Physical/Schematic | Circuit, force, block diagrams | High — symbol vocabularies, topology vs. layout | Partial scope; simple schematics in v1, complex domain grammars deferred |
| Type 5: Conceptual/Rhetorical | Process diagrams, metaphors, teaching diagrams | Low-Medium — loose genre, intent-heavy | In scope; acceptability criterion is loose, pipeline has appropriate freedom |

---

*v2 — expanded from initial tier taxonomy to five content types.*
*Updated after lit review and content type analysis session.*
*Next: Round 2 schema stress-test against one representative case from each type.*
