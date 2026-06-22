# Use Cases Considered — Summary

## Tier 1 — Single Entity

How I understand this tier:  An "atomic" object of some sort; more general than a "typed value". An equation is a good
example of structured data that is still "atomic". A "biohazard" symbol requires a collection of drawing primitives,
but forms one semantic element.

- Key metric / scalar display (includes gauge or meter visualizations: progress bar, dial, meter)
- Labeled glyph, icon, or symbol (includes color swatch, Boolean TRUE/FALSE indicator)
- Standalone equation

## Tier 2 — Homogeneous Collection

How I understand this tier: This would correspond to a single "sequence" in a spreadsheet, with an optional parallel
labels sequence, or more generally to a set of objects of the same fundamental class with attributes. This tier does
not envision relationships between elements in the set, although common values of attributes could imply relationships
(for example, entities could be businesses, some of which are car dealerships - a visualization might want to
represent these commonalities by proximity or similar colors/textures/styles).

Question: does this tier include multiple homogeneous collections, like multi-series charts? Series 1 vs. series 2?
Donut plots with several rings?

- Bar chart (categorical comparison)
- Pie chart (part-to-whole)
- Ranked list
- Histogram

## Tier 3 — Heterogeneous Elements, Shared Frame

How I understand this tier: multi-dimensional data (with dimension 2 being most common). Time series seems an odd fit
under that interpretation - that seems more like "series with attributes" that I imagine in the prior tier.

There is an interesting space here to consider: "point sets". A scatter plot is a 2D point set, as is a function plot
(explicit, implicit, etc.). There are 1-D point sets (intervals on a number line), 2-D point sets (graphs or regions
in the plane, described semantically, 3-D point sets) like spheres or surfaces that could be represented in a
perspective/orthogonal view. Point sets could define plot windows for each axis, linear vs nonlinear scaling, etc.
These could all mix isolated points with connected regions. Would point-sets be worth a distinct tier?

Vector fields are related concepts as well. They could be layered onto point sets of any displayable dimension.
Once you admit "point sets with attributes", a lot of visualization types open up - the question is whether this is the
right semantic form for an LLM to specify such visualizations, or just an elegant mathematical structure.

- Scatter plot with regression line and confidence interval overlay
- Dual-axis chart (primary + secondary y-axis)
- Time series with event annotations

## Tier 4 — Explicit Relational Structure

How I understand this tier: non-geometric relationships (so not point sets). Things that can be laid out many different
way but still convey relationships through links between nodes, tree/containment structures. Relations could be
directed or not. Would a Gannt chart fall in this category? I would put commutative diagrams here.

- Graph / network diagram
- Org chart, family tree (labeled hierarchy)
- Dependency graph (DAG)
- Concept/mind map (heterogeneous relation types)
- Venn diagrams
- Flowchart
- Database ERD

## Tier 5 — Spatial Structure

How I understand this tier: objects with meaningful spatial or geographic relationships, where the spatial piece is key
to communicating meaning. I would put geometric diagrams (circles, tangent lines, arcs, trigonometric figures, etc.) in
this category. A network topology is interesting - physical locations might be less important than logical
connections - some representations should honor actual physical locations of nodes/phones/devices, but others might
view the same data in a layout more organized by connectivity (subnet, switch, etc.).

- Floor plan with labeled regions
- Network topology diagram
- Circuit diagram
- Cartographic maps (or data overlaid on maps, like circles sized by population on each state/country)

## Tier 6 — Composite / Multi-Panel

How I understand this tier: multiple child visualizations that are somehow related and are to be presented as a unit,
perhaps with some meaningful visual relationships, like axes that line up for easier comparison.

- 2×2 grid of aligned charts with shared axes
- Dashboard (summary stat + detail chart + annotation)
- Visual proof (sequential panels with logical dependency)
- Before/after comparison pair

## Tier 7 — Cross-Panel Relations

How I understand this tier: An extension of the prior tier that allows relations (arrows, flows, etc.) between the
child visualizations.

- Overview + detail pair (selection in one drives the other)
- Same entity appearing in multiple panels
- Shared legend governing multiple panels

Other visualization types to consider:

- Heat maps
- Are tables out-of-scope?

- Where would things like "visual metaphors" end up"  For example, "machine" that represents a function, with an input
  funnel and output; meshed gears, pulleys, rope dragging a box, rolling carts, and things like that.

